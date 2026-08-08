# Day 1 — Blink & Toolchain

First day of the ESP32 course. Goal: get the toolchain working end-to-end (write → upload → verify) using the simplest possible circuit, and learn the one concept that shapes every project after this: **blocking vs. non-blocking code**.

## Table of Contents

- [Objectives](#objectives)
- [Hardware](#hardware)
- [Wiring](#wiring)
- [Concepts](#concepts)
- [Code: Blocking Blink](#code-blocking-blink)
- [Code: Non-Blocking Blink](#code-non-blocking-blink)
- [How They Differ](#how-they-differ)
- [Build & Upload](#build--upload)
- [Checklist](#checklist)
- [Troubleshooting](#troubleshooting)
- [Stretch Goals](#stretch-goals)
- [Repo Structure](#repo-structure)

## Objectives

By the end of Day 1 you should be able to:

- [ ] Wire an external LED safely with a current-limiting resistor
- [ ] Explain what `pinMode()` actually configures at the hardware level
- [ ] Write a blocking blink using `digitalWrite()` / `delay()`
- [ ] Rewrite it as a non-blocking blink using `millis()`

## Hardware

| Qty | Part |
|---|---|
| 1 | ESP32 dev board (e.g., ESP32-WROOM-32 DevKit) |
| 1 | LED (any color, standard 5mm) |
| 1 | 220 Ω resistor |
| 2 | Jumper wires |
| 1 | Breadboard |

## Wiring

```
ESP32 GPIO4 ──── 220Ω resistor ──── LED anode (long leg)
                                     LED cathode (short leg) ──── GND
```

- The resistor can sit on either side of the LED — anywhere in series is fine.
- **Long leg = anode** → toward the resistor/signal pin.
- **Short leg = cathode** → toward GND.
- ESP32 GPIO pins are **3.3V logic**, not 5V — the board is not 5V-tolerant, so never feed a GPIO pin more than 3.3V.
- 220 Ω keeps current in the ~5–6 mA range at 3.3V: safe for both the LED (~20 mA max) and the pin (ESP32 recommends staying under ~12 mA per pin, with a 40 mA absolute max).
- GPIO4 is a safe general-purpose output pin with no special boot-strapping behavior, so it's a good default choice. Avoid strapping pins (GPIO0, 2, 12, 15) for your first project since their state at boot affects flashing mode.
- LED backwards? It just won't light — LEDs are diodes, they don't conduct in reverse. No damage, just flip it.

## Concepts

**`pinMode(pin, mode)`**
Configures a pin's electrical behavior. `OUTPUT` mode lets the pin actively drive HIGH (3.3V) or LOW (0V) with enough current to run small loads like an LED. Left in the default `INPUT` state, the pin is high-impedance and can't reliably drive anything.

**`digitalWrite(pin, value)`**
Sets an `OUTPUT` pin to `HIGH` or `LOW`. That's it — it's a single instantaneous voltage change, not a duration.

**Blocking vs. non-blocking**
A *blocking* call (like `delay()`) halts the entire program for its duration — nothing else runs, no inputs are read, no other logic executes. A *non-blocking* approach never sleeps; it repeatedly checks "has enough time passed?" using a running clock (`millis()`), so the rest of `loop()` stays free to do other things every single pass. This is the foundation for handling buttons, sensors, Wi-Fi, multiple timers, and animations at once later in the course.

## Code: Blocking Blink

```cpp
const int LED_PIN = 4;

void setup() {
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_PIN, HIGH);
  delay(500);
  digitalWrite(LED_PIN, LOW);
  delay(500);
}
```

## Code: Non-Blocking Blink

```cpp
const int LED_PIN = 4;
const unsigned long INTERVAL = 500; // ms

unsigned long previousMillis = 0;
bool ledState = LOW;

void setup() {
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  unsigned long currentMillis = millis();

  if (currentMillis - previousMillis >= INTERVAL) {
    previousMillis = currentMillis;
    ledState = !ledState;
    digitalWrite(LED_PIN, ledState);
  }

  // any other code placed here keeps running freely,
  // completely unaffected by the LED timing
}
```

## How They Differ

| | `delay()` version | `millis()` version |
|---|---|---|
| CPU during the "wait" | Frozen | Free to run other code |
| Can read a button mid-blink? | No | Yes |
| Can run two independent timers? | Not cleanly | Yes |
| Behavior at `millis()` overflow (~49 days) | N/A | Handled correctly by subtraction |
| Mental model | Sequential script | Repeating state check |

The subtraction pattern `currentMillis - previousMillis >= INTERVAL` is used instead of `currentMillis >= previousMillis + INTERVAL` specifically because unsigned integer subtraction still behaves correctly across the `millis()` rollover point; the addition form doesn't.

## Build & Upload

1. Install ESP32 board support in the Arduino IDE if you haven't already: **File → Preferences**, add the ESP32 boards manager URL, then **Tools → Board → Boards Manager**, search "esp32" (by Espressif Systems), and install it.
2. Open `blocking_blink/blocking_blink.ino` (or `nonblocking_blink/nonblocking_blink.ino`) in the Arduino IDE.
3. Select **Tools → Board** and pick your specific ESP32 board (e.g., "ESP32 Dev Module").
4. Select **Tools → Port** and pick the port your board enumerated on.
5. Click **Upload** (→). Some boards require holding the **BOOT** button while upload starts if it doesn't auto-reset into flashing mode.
6. Confirm the LED blinks at a steady ~1 Hz (500ms on, 500ms off).

## Checklist

- [ ] LED lights when pin is HIGH, off when LOW
- [ ] Blocking version compiles and blinks at ~500ms intervals
- [ ] Non-blocking version compiles and blinks at ~500ms intervals
- [ ] Can explain, out loud, why `delay()` would break a "blink LED + read button" program
- [ ] Comfortable changing the blink interval in both versions

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| LED never lights | Reversed polarity, wrong pin number in code, loose jumper |
| LED very dim | Resistor value too high, or wired to a pin that isn't actually toggling |
| Board not detected | Missing USB-to-serial driver (CP2102/CH340), wrong port selected, bad USB cable (some are power-only) |
| Upload fails / times out | Wrong board selected in Tools menu, or board needs manual BOOT button press to enter flashing mode |
| Non-blocking version never toggles | `INTERVAL` type mismatch or `previousMillis` never updated — check it's `unsigned long` |

## Stretch Goals

- Change the non-blocking interval to 200ms.
- Add a second LED on a different pin, blinking at a different rate, using a second `previousMillis` variable — this only works cleanly with the `millis()` approach.
- Make the blink rate speed up over time without using `delay()`.

---
**Course:** ESP32 Fundamentals · **Day:** 1 of N · **Topic:** Blink & Toolchain

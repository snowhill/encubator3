# encubator3
Here is the updated **Incubator Pro V3.2 Datasheet**. This version includes all the recent hardware additions (Egg Probe), software features (Periodic Ventilation), and UI changes.

You can copy this directly into your GitHub `README.md`.

---

# Incubator Pro V3.2 - Advanced ESP32 Controller

**Incubator Pro V3.2** is a professional-grade, PID-controlled incubator firmware built on [ESPHome](https://esphome.io). It transforms an ESP32 into a fully autonomous climate controller with medical-grade monitoring, simulated biological cooling, periodic fresh air purging, and "True Egg Temperature" tracking.

## 🌟 Key Features

* **Precision PID Heating:** Controls SSR/Heater with < 0.1°C variance.
* **Tri-Sensor Monitoring:**
* **Air (Internal):** SHT30 High-precision I2C sensor for rapid PID response.
* **Egg Core:** DS18B20 Waterproof Probe to track thermal mass (simulation of internal egg temp).
* **Room (External):** DHT22 sensor to monitor ambient conditions and thermal loss.


* **Smart Ventilation System:**
* **Biological Cool Down:** Daily scheduled cooling (simulates mother bird leaving nest).
* **Periodic Gas Exchange:** Configurable "Fresh Air Flush" (e.g., run fan for 30s every hour) to reduce CO2 buildup.


* **Advanced Safety:**
* Hardware-independent overheat cutoff.
* Sensor failure detection (Heater disable on NaN).
* Configurable absolute max temp limit.


* **Intuitive UI:** 2.4" IPS Display with **Edit Mode** visual cues (Yellow focus box, blinking indicators).
* **Connectivity:** Native Home Assistant API + Web Server + WiFi Fallback Hotspot.

---

## 🛠 Hardware Specifications

| Component | Model/Type | Interface |
| --- | --- | --- |
| **MCU** | ESP32 Dev Kit V1 | - |
| **Display** | 2.4" TFT IPS (240x320) | SPI (ST7789V) |
| **Air Sensor** | SHT30 | I2C (0x44) |
| **Egg Probe** | DS18B20 Waterproof | 1-Wire |
| **Room Sensor** | DHT22 | GPIO |
| **Heater** | Solid State Relay (SSR) | Slow PWM (GPIO) |
| **Humidifier** | 5V Relay Module | GPIO |
| **Vent Fan** | 5V Relay / MOSFET | GPIO |
| **Controls** | Rotary Encoder + Button | GPIO (Pullup) |

---

## 🔌 Wiring Diagram & Pinout

### **Actuators**

| Device | ESP32 Pin | Notes |
| --- | --- | --- |
| **Heater (SSR)** | **GPIO 13** | Controlled via PID / Slow PWM. |
| **Humidifier** | **GPIO 5** | *Caution: Boot strapping pin. If relay is Active High, ensure it is OFF during boot.* |
| **Ventilation Fan** | **GPIO 17** | Active during Cool-Down OR Periodic Flush. |

### **Sensors**

| Sensor | ESP32 Pin | Notes |
| --- | --- | --- |
| **SHT30 (SDA)** | **GPIO 21** | Main PID control sensor. |
| **SHT30 (SCL)** | **GPIO 22** | Main PID control sensor. |
| **DS18B20 Data** | **GPIO 15** | **Requires 4.7kΩ pull-up resistor** between Data and 3.3V. |
| **DHT22 Data** | **GPIO 19** | External room monitoring. |

### **Display (ST7789V SPI)**

| Display Pin | ESP32 Pin |
| --- | --- |
| **CS** | **GPIO 27** |
| **DC** | **GPIO 14** |
| **RST** | **GPIO 4** |
| **SDA (MOSI)** | **GPIO 23** |
| **SCL (CLK)** | **GPIO 18** |
| **VCC/GND** | 3.3V / GND |

### **Input Controls**

| Input | ESP32 Pin | Function |
| --- | --- | --- |
| **Encoder CLK** | **GPIO 32** | Value Adjustment |
| **Encoder DT** | **GPIO 33** | Value Adjustment |
| **Encoder SW** | **GPIO 25** | Select / Save / Toggle Edit Mode |
| **Nav Button** | **GPIO 26** | Cycle Pages |

---

## 🖥 User Interface Guide

Use the **Bottom Button (GPIO 26)** to navigate. Use the **Knob Button** to Edit/Save.

### **Page 0: Dashboard**

* **Visuals:** Big Temp/Hum numbers, Visual Target Bars.
* **Status:** "Incubator Active" / "Cooling Down" / "Fresh Air Flush".

### **Page 1: Detailed Stats**

* **Data:** Current Air Temp, Target Temp, **Egg Probe Temp (Yellow)**.
* **Analysis:** Temperature Deviation (+/-), Humidity Min/Max.

### **Page 2: Room Environment**

* **Data:** External Temp & Humidity (DHT22).
* **Delta:** Difference between Internal and Room temp (thermal load).

### **Page 3: Device Status**

* **Relays:** Real-time status (ON/OFF) of Heater, Humidifier, and Fan.
* **Counters:** Total Runtime (Hours/Minutes) for each device.

### **Page 4: Set Temperature**

* **Action:** Press Knob to enter **Edit Mode**.
* **Visual Cue:** Number turns **Yellow** inside a box, arrows `< >` blink.
* **Range:** 30.0°C - 40.0°C.

### **Page 5: Set Humidity**

* **Action:** Press Knob to Edit.
* **Range:** 30% - 80%.

### **Page 6: Safety Settings**

* **Max Temp:** Heater Hard-Cutoff limit (Default 40.0°C).
* **Cool Down Target:** Target temp during daily biological cooling phase.

### **Page 7: Ventilation Settings (New)**

* **Vent Interval:** Run fan every X minutes (0 - 240 min). Set to 0 to disable.
* **Vent Duration:** Run fan for X seconds (5 - 120 sec).
* **Countdown:** Shows time remaining until next air flush.

### **Page 8: System Info**

* **Network:** WiFi Signal (dBm/%), IP Address.
* **Diagnostics:** Sensor Status (SHT30/DHT22 OK/FAIL), Free RAM, Uptime.

---

## ⚙️ Logic & Automation

### **1. PID Heating Algorithm**

The system uses Proportional-Integral-Derivative control on the **SHT30 (Air Temp)** to maintain stability.

* **Kp:** 0.6 | **Ki:** 0.005 | **Kd:** 0.0
* *Note:* The DS18B20 is for monitoring only; it is too slow for active heater control.

### **2. Biological Cool Down**

Simulates the mother bird leaving the nest to eat.

* **Trigger:** Daily at 12:00 PM.
* **Duration:** 30 Minutes.
* **Action:** Target Temp lowers to `Cool Down Target` (32°C), Fan turns ON continuously.

### **3. Periodic Fresh Air Flush (CO2 Management)**

Prevents embryo suffocation in late stages.

* **Trigger:** Based on `Vent Interval` setting (e.g., every 60 mins).
* **Action:** Fan turns ON for `Vent Duration` (e.g., 30s).
* **Heater:** Remains active (PID) to recover temp quickly after flush.

---

## 🏠 Home Assistant Entities

| Entity ID | Type | Description |
| --- | --- | --- |
| `climate.incubator_heater` | Climate | Main control (Target/Current/Mode). |
| `sensor.internal_temp` | Sensor | Air Temperature (SHT30). |
| `sensor.egg_core_temp` | Sensor | Egg Probe Temperature (DS18B20). |
| `number.target_temp` | Number | Set Point (synced with screen). |
| `number.vent_interval` | Number | Ventilation frequency (minutes). |
| `number.vent_duration` | Number | Ventilation duration (seconds). |
| `binary_sensor.ventilation_fan` | Binary | Fan Status. |

---

## ⚠️ Safety Disclaimer

**High Voltage Warning:** This project controls heating elements (110V/220V). Ensure all AC wiring is insulated and enclosed.
**Redundancy:** Never rely solely on software. Always install a **Thermal Fuse** (e.g., 70°C) in series with your heater to prevent fire in case of SSR failure.



Here is a comprehensive **Technical Reference & Implementation Guide** for your Incubator Pro V3.2.

This document is structured specifically to be **fed into an AI model** at the start of a future project. It defines the hardware constraints, software drivers, and "lessons learned" to prevent regression errors.

---

# 📘 Project Reference: Incubator Pro V3.2 (ESPHome)

**Purpose:** This document defines the hardware architecture, software drivers, and critical implementation details for an ESP32-based PID Incubator.
**Target Audience:** AI Assistants / Developers for future iterations.

---

## 1. Hardware Architecture & Pinout Strategy

### **A. ESP32 Pin Selection Rules (Critical)**

* **Strapping Pins (Avoid if possible):** GPIO 0, 2, 5, 12, 15.
* *Lesson:* **GPIO 5** was used for the Humidifier. If the relay module is "Active High" or has a strong pull-up/down, it can prevent the ESP32 from booting.
* *Fix:* Ensure relays on strapping pins are "Active Low" or disconnected during flashing.


* **Input Only Pins:** GPIO 34, 35, 36, 39. (Cannot be used for Relays/SDA/SCL).
* **VSPI Bus:** Used for Display to ensure high-speed refresh (GPIO 18, 19, 23, 5).

### **B. Wiring Map**

| Component | Interface | ESP32 Pin | Logic / Driver | Notes |
| --- | --- | --- | --- | --- |
| **Display (ST7789V)** | SPI | **CS: 27** | Active Low | Chip Select |
|  |  | **DC: 14** | Data/Command |  |
|  |  | **RST: 4** | Active Low | Reset |
|  |  | **SDA: 23** | SPI MOSI | Hardware SPI (VSPI) |
|  |  | **SCL: 18** | SPI CLK | Hardware SPI (VSPI) |
| **Internal Sensor** | I2C | **SDA: 21** | I2C Data | SHT30 Address: `0x44` |
|  |  | **SCL: 22** | I2C Clock |  |
| **Egg Probe** | 1-Wire | **Data: 15** | Digital | **REQUIRES 4.7kΩ Pull-Up Resistor** |
| **Room Sensor** | 1-Wire | **Data: 19** | DHT22 |  |
| **Heater** | PWM | **Pin: 13** | Slow PWM | Solid State Relay (SSR) |
| **Humidifier** | Digital | **Pin: 5** | Active High/Low | *Warning: Strapping Pin* |
| **Vent Fan** | Digital | **Pin: 17** | Active High | MOSFET or Relay |
| **Encoder** | Quadrature | **A: 32** | Internal Pullup |  |
|  |  | **B: 33** | Internal Pullup |  |
|  |  | **Btn: 25** | Inverted Pullup | Integrated Button |
| **Nav Button** | Digital | **Pin: 26** | Inverted Pullup | Separate Page Button |

---

## 2. Component Implementation Details

### **1. Display: ST7789V 2.4" IPS**

* **Driver:** `platform: st7789v`
* **Model:** `model: CUSTOM` (Generic 240x320)
* **Methodology:**
* Use `eightbitcolor: true` to save RAM.
* **Optimization:** Avoid `id(my_display).update()` in every loop. Use a `display_dirty` flag logic to only redraw when values change or during navigation.
* **Fonts:** Loading large fonts (Size 44+) consumes significant Flash (ROM). Use `gfonts://` carefully.



### **2. Heater Control: PID + SSR**

* **Driver:** `platform: pid` (Climate) + `platform: slow_pwm` (Output).
* **Why Slow PWM?** Solid State Relays (SSRs) are Zero-Crossing devices. They cannot switch at 1000Hz like an LED.
* **Configuration:**
* `period: 2s` (The heater cycles ON/OFF within a 2-second window).
* **PID Constants:** `kp: 0.6`, `ki: 0.005`, `kd: 0.0`. (Tuned for insulated foam box).



### **3. Temperature Sensors (The "Tri-Sensor" Approach)**

* **SHT30 (I2C):**
* *Role:* **Primary Control.** Fast response time. Used for PID input.
* *Driver:* `platform: sht3xd`.


* **DS18B20 (1-Wire):**
* *Role:* **Verification.** Slow thermal response. Simulates egg core mass.
* *Driver:* `one_wire` (Bus) + `dallas_temp` (Sensor).
* *Constraint:* **Do not use for PID.** Its thermal lag will cause the heater to overshoot dangerous temps before the sensor registers it.


* **DHT22:**
* *Role:* **Context.** Measures room ambient to calculate thermal loss.



---

## 3. "Mistakes Learned" & Solutions (The Ledger)

**These are specific errors we encountered and solved. Do not repeat them.**

### **❌ Mistake 1: Using `dallas` platform in newer ESPHome**

* **Error:** `Failed config: dallas: [source /config/esphome/dis.yaml:133] The "dallas" component has been replaced...`
* **Solution:** As of 2024+, the component is split.
* Define the bus using `one_wire:`.
* Define the sensor using `platform: dallas_temp`.



### **❌ Mistake 2: Missing Pull-Up on DS18B20**

* **Symptom:** Sensor reads `NaN`, `0.0`, or cannot be found.
* **Physics:** The 1-Wire protocol requires the data line to be pulled HIGH (3.3V) when idle.
* **Fix:** Install a physical **4.7kΩ resistor** between the YELLOW (Data) and RED (3.3V) wires.

### **❌ Mistake 3: YAML Indentation in `interval` Blocks**

* **Error:** `expected <block end>, but found '<scalar>'`
* **Cause:** Writing a comment or code inside a lambda without proper indentation relative to the `then:` block.
* **Fix:** Ensure the `- lambda: |-` block is indented by 2 spaces, and the C++ code inside is indented further.

### **❌ Mistake 4: Using "PWM" for AC Relays**

* **Risk:** Using standard `ledc` (Fast PWM) on an SSR will cause it to overheat or fail to switch (because it waits for AC zero-crossing).
* **Fix:** Always use `platform: slow_pwm` with a period > 1 second for AC heating elements.

### **❌ Mistake 5: Flash Memory Overflow**

* **Symptom:** `Text segment doesn't fit in IRAM`.
* **Cause:** Too many large fonts or images.
* **Fix:**
* Add `min_version: 3.1` to `esp32:` config to strip legacy code.
* Reuse fonts where possible.
* Use `web_server` sparingly if space is tight.



### **❌ Mistake 6: CO2 Suffocation**

* **Observation:** High hatch failure rate in final days despite perfect temps.
* **Cause:** Lack of gas exchange.
* **Fix:** Added **Page 7 (Ventilation)** to force fan ON for `X` seconds every `Y` minutes regardless of temperature.

---

## 4. Software Design Patterns (Methods)

When generating code for this system, follow these patterns:

1. **State Management:** Use `globals` for all state (Target Temp, Menu Page, Timers). Do not rely on static C++ variables inside lambdas as they reset on boot.
2. **Non-Blocking UI:** Never use `delay()` inside the display lambda. It freezes the PID controller. Use `millis()` or ESPHome `interval` components.
3. **Safety Loop:**
```cpp
// Always include this in the main interval loop
if (isnan(id(main_sensor).state) || id(main_sensor).state > id(safety_cutoff).state) {
   id(heater).turn_off();
   id(pid_climate).mode = CLIMATE_MODE_OFF;
}

```


4. **Edit Mode Pattern:**
* Use a `bool edit_mode` global.
* If `true`: Encoder changes the *Value*.
* If `false`: Encoder changes the *Page*.
* Use a `timer` to auto-exit Edit Mode after 60s to prevent accidental changes.



---

## 5. Start-Up Prompt for AI

*Copy and paste this block when starting a new chat with an AI about this project:*

> **Context:** I am working on "Incubator Pro V3," an ESP32-based project using ESPHome.
> **Hardware:** ESP32 DevKit V1, ST7789V Display (SPI), SHT30 (I2C), DS18B20 (1-Wire with 4.7k Pullup), SSR Heater (Slow PWM), Relay Humidifier (GPIO 5).
> **Constraints:**
> 1. Use `one_wire` component for DS18B20, not the deprecated `dallas`.
> 2. Do not put critical relays on strapping pins (GPIO 0, 2, 12, 15) unless necessary; GPIO 5 is currently used for Humidifier (active high caution).
> 3. Use `slow_pwm` for the heater.
> 4. The Display uses `display_dirty` flag logic to prevent flickering.
> 5. Safety: Heater MUST shut off if SHT30 reads NaN.
> **Current Goal:** [Insert your new request here]
> 
>

# 🍽️ Digestive System Simulation (Arduino Mega)

## 📌 Overview

This project simulates a simplified digestive system using an **Arduino Mega 2560**. A color sensor detects macromolecules (carbohydrates, fats, proteins) and triggers:

* Audio playback (DFPlayer Mini)
* LED activation (organs)
* Servo movement (digestion stages)

---

## 🧩 Hardware Components

* Arduino Mega 2560
* TCS3200 / TCS230 Color Sensor
* DFPlayer Mini + Speaker
* 5 Servo Motors
* 9 LEDs
* Calibration LED (pin 37)

---

## ⚙️ How It Works

1. The color sensor reads RGB frequency values.
2. The system performs **10 consecutive readings**.
3. A **voting system** selects the most frequent result.
4. Based on the detected macromolecule:

   * Plays audio
   * Lights specific LEDs
   * Moves servos

---

## 📁 DFPlayer File Structure

```
001.mp3 → Carbohydrates (carb)
008.mp3 → Fats
015.mp3 → Proteins
```

Use:

```cpp
myDFPlayer.play(file);
```

---

## 🧠 Voting System

The code reduces noise by:

* Taking 10 samples
* Classifying each sample
* Selecting the majority result

Example:
6 blue, 4 yellow → final result = **blue**

---

## 🎯 Calibration (Critical)

### ⚠️ Why Calibration Is Required

The TCS3200 sensor does **not** output fixed values. Readings vary depending on:

* Ambient lighting
* Distance from object
* Sensor angle
* Surface reflectivity
* Whether the calibration LED is ON or OFF

Uncalibrated values will result in constant misclassification.

---

### 🔬 Calibration Procedure

1. Upload a raw-reading test code
2. Keep conditions **strictly identical**:

   * Same distance
   * Same lighting
   * Same LED state
3. Collect **10–15 readings per color**
4. Determine:

   * Value ranges (min/max)
   * Channel relationships (R, G, B ordering)
5. Update the `detectColor()` function

---

### 🧩 Detection Strategy

Reliable classification is based on:

* **Relative ordering**

  * Yellow → `B > G > R`
  * Blue → `G ≥ R ≥ B`
  * Red → `G ≈ B >> R`

* **Channel differences**

  * Example: `abs(G - B) < threshold`

Absolute values alone are not sufficient.

---

## 💡 Calibration LED (Pin 37)

Used to stabilize detection:

* ON → during color reading
* OFF → during actions (audio, servo movement)

This ensures consistent sensor input.

---

## ⚠️ Limitations

* Sensitive to environmental changes
* Requires recalibration if setup changes
* Audio timing relies on manual delays

---

## 🧪 Summary

This project combines:

* Sensor input (color detection)
* Signal processing (voting system)
* Actuation (servos, LEDs, audio)

System reliability depends entirely on **consistent calibration conditions**.

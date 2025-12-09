# 💧 ESPHome Water Tank Level Sensor

This project details the creation of a reliable water tank level sensor using an **ESPHome**-enabled **Wemos D1 Mini** (ESP8266) and a **JSN-SR04T waterproof ultrasonic sensor**.

---

## 🎯 Project Goals & Features

* **Reliable Measurement:** Use a waterproof ultrasonic sensor for non-contact level sensing.
* **Safety First:** Implement a **voltage divider** to safely interface the 5V sensor with the 3.3V ESP8266 microcontroller.
* **Smart Home Integration:** Use ESPHome for seamless integration with Home Assistant.
* **Accurate Output:** Calculate and output the tank's water level as a percentage.

---

## 🛠️ Hardware Components

| Component | Purpose | Notes |
| :--- | :--- | :--- |
| **Wemos D1 Mini** (or ESP8266) | Microcontroller & Wi-Fi module | Runs the ESPHome firmware. |
| **JSN-SR04T** | Waterproof Ultrasonic Sensor | Measures distance to the water surface. |
| **Resistors ($\mathbf{1k\Omega}$ and $\mathbf{2k\Omega}$)** | Voltage Divider | **Critical** for safety. See wiring section. |
| **3D Printed Mount** | Sensor Holder | The file **tank_sonar-holder v5.stl** is provided. |

---

## 🔌 Wiring and Pin Connections

**⚠️ CRITICAL SAFETY WARNING:** The JSN-SR04T sensor outputs a **5V** signal on its Echo pin, which will damage the Wemos D1 Mini's 3.3V GPIO pins over time. **A voltage divider is mandatory.**

### Sensor to Wemos D1 Mini Pinout

| Sensor Pin | Wemos D1 Mini Pin | ESP8266 GPIO | Purpose |
| :--- | :--- | :--- | :--- |
| **GND** (Pin 1) | G (Ground) | GND | Common Ground |
| **VCC** (Pin 4) | 5V | 5V | Sensor Power Supply (5V) |
| **Trig** (Pin 3) | **D1** | **GPIO5** | Trigger Pulse Input |
| **Echo** (Pin 2) | **D2** | **GPIO4** | Echo Signal Output (Connect via Voltage Divider) |

### **The Critical Voltage Divider**

The voltage divider steps down the 5V Echo signal to a safe 3.3V level.


* **$\mathbf{R_1}$** ($\mathbf{1k\Omega}$): Placed **in series** between the sensor's **Echo** pin and the Wemos **D2 (GPIO4)** pin.
* **$\mathbf{R_2}$** ($\mathbf{2k\Omega}$): Placed **in parallel** between the Wemos **D2 (GPIO4)** pin and the Wemos **GND** pin.

> **Note on Alternatives:** Using $R_1 = 1.1k\Omega$ and $R_2 = 2.2k\Omega$ is a verified safe alternative.

---

## 💻 ESPHome Configuration (`water-tank.yaml`)

The complete, validated ESPHome configuration is located in the `water-tank.yaml` file.

### 1. Raw Distance Sensor Setup
The `ultrasonic` platform is configured to use the correct pins (`trigger_pin: D1`, `echo_pin: D2`) and assigned the ID `raw_water_distance`.

### 2. Data Filtering
The configuration uses two filters, which are essential for stable readings:
1.  **Lambda Filter:** Discards any reading outside the tank's operational range (less than $0.2m$ or greater than $1.9m$). This prevents noisy data from affecting the final output.
    ```yaml
    # ... inside filters:
    - lambda: |-
        if (x < 0.2 || x > 1.9) {
          return {}; // Discard this invalid reading
        }
        return x;    // This reading is valid, pass it on
    ```
2.  **Median Filter:** Smooths the remaining valid data points to eliminate jitter.

### 3. Water Level Percentage Calculation
A template sensor calculates the final water level percentage based on your tank dimensions. The calibration values used are:

* `dist_empty`: **$1.8$ meters** (Distance from sensor to the bottom of the empty tank).
* `dist_full`: **$0.3$ meters** (Distance from sensor to the water surface when the tank is full).

The formula used:
$$\text{Percentage} = \frac{(\text{dist\_empty} - \text{Current Distance})}{(\text{dist\_empty} - \text{dist\_full})} \times 100$$

---

## 📸 Media and Files

| File | Description |
| :--- | :--- |
| [`water-tank.yaml`](water-tank.yaml) | The complete ESPHome configuration code. |
| [`tank_sonar-holder v5.stl`](tank_sonar-holder%20v5.stl) | STL file for 3D printing the sensor mounting bracket. |
| `images/physical-build.jpg` | Photo of the final assembled unit. |
| `images/home-assistant-card.png` | Screenshot of the final result in Home Assistant. |



---

## 🚀 Flashing and Troubleshooting

1.  **Initial Flash:** Connect the Wemos D1 Mini via USB.
2.  **Firmware Updates (The 'Adopt' Error):** If you encounter the `"Configuration does not match the platform..."` error, you must either click the **"ADOPT"** button in the ESPHome dashboard or manually **"Erase flash"** before attempting a re-flash.

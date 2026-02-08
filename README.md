# 🏎️ Universal EV Smart Load Balancer
### A Modular Home Assistant Ecosystem for Intelligent Solar Charging

This ecosystem is a professional-grade suite of Home Assistant Blueprints designed to synchronize your Electric Vehicle (EV) charging with your home's energy production. By monitoring your **HomeWizard P1 Meter** and (optionally) your **HomeWizard Plugin Battery**, this system ensures you charge using surplus energy while strictly protecting your home from grid overloads.

---

## 💎 Key Features
* **Brand Agnostic**: Works with any EV (Tesla, Kia, Hyundai, etc.) or Charger (Wallbox, Easee, Zaptec) integrated into Home Assistant.
* **Dual-Path Surplus Monitoring**: Independent thresholds for Grid feed-in (P1) and Plugin Battery flow.
* **Safety-First Design**: Includes a "Strict Decrease" logic that kills the charging session if the grid draw is too high at minimum Amps.
* **Zero-Waste Charging**: Dynamically adjusts current (+1/-1 Amp) every 5 minutes based on real-time availability.
* **Cloud Protection**: Intelligent delays prevent the charger from toggling on/off every time a cloud passes.
* **Auto-Pilot**: Automatically starts when surplus is detected and stops when the car is full or unplugged.

---

## 🛠️ Prerequisites & Hardware
To use this system, you need the following:

### 1. Energy Monitoring
* **HomeWizard P1 Meter**: To monitor total house consumption and solar return.
* **HomeWizard Plugin Battery** (*Optional*): To monitor energy flow into/out of your home batteries.

### 2. EV Integration
Your car or charger integration must provide:
* **Current Control**: A `number` entity to set Amperes (e.g., `number.tesla_charge_current`).
* **Start/Stop Switch**: A `switch` or `button` to toggle charging.
* **Connection Sensor**: A sensor showing if the cable is plugged in.
* **SoC & Limit**: Sensors for Battery Percentage (%) and Charge Limit (%).

### 3. Helpers (Manual Step)
Go to **Settings > Devices & Services > Helpers** and create:
1.  **Input Boolean**: `input_boolean.load_balancing_ev` (The "Master Switch").
2.  **Input Boolean**: `input_boolean.load_balancing_ev_sun` (The "Sun Mode" flag).

---

## 📥 Installation Guide

### Step 1: Save the Blueprints
1.  Open your Home Assistant configuration folder.
2.  Navigate to `/blueprints/automation/`.
3.  Create 5 new files and paste the verified YAML code into each:
    * `ev_master_control.yaml`
    * `ev_increase_amps.yaml`
    * `ev_decrease_amps.yaml`
    * `ev_auto_start.yaml`
    * `ev_auto_stop.yaml`
4.  Go to **Developer Tools > YAML** and click **Automations**.

### Step 2: Create the Automations
Follow this specific order to ensure everything links correctly:
1.  **Create "Increase Amps"**: Configure your P1/Grid sensor and surplus thresholds.
2.  **Create "Decrease Amps"**: Link your P1 sensor and set your "Grid Draw" limit (e.g., 250W).
3.  **Create "Master Control"**: Link the two automations you just created. This acts as the "Coordinator."
4.  **Create "Auto Start" & "Auto Stop"**: Set your preferred surplus thresholds to trigger the system.

---

## ⚙️ Understanding Thresholds & Delays
This system uses **Independent Thresholds** and **Intentional Delays** for stability:

| Threshold/Delay | Description | Recommended Value |
| :--- | :--- | :--- |
| **Grid Start** | Watts sent to grid before charging starts. | `-1200W` |
| **Battery Start** | Watts pulled from Plugin Battery before charging starts. | `1200W` |
| **5-Minute Filter** | Time the surplus must be stable before starting or ramping up. | `Fixed` |

### Why the 5-Minute Delay? (Cloud Protection)
The system is built to be "Solar Smooth." Without the 5-minute buffer, your charger would constantly start, stop, and change speeds every time a small cloud passes or you use a high-draw appliance like a toaster. This delay protects your EV's internal contactors and ensures a steady charging session rather than "flapping" on and off.

---

## 🚀 How it Works (The Logic Flow)
1.  **The Trigger**: Your P1 meter shows you are sending `800W` back to the grid.
2.  **Stability Check**: The system waits for **5 minutes**. If the sun stays out, it proceeds. If a cloud passes and the surplus drops below the threshold, the timer resets.
3.  **The Start**: `Auto Start` confirms the cable is connected and flips the `Master Switch`.
4.  **The Wake-Up**: `Master Control` wakes the car via API, sets it to `5 Amps`, and begins the balancing logic.
5.  **The Balancing**: 
    * `Increase Amps` adds +1A every 5 mins if surplus remains stable.
    * `Decrease Amps` drops the current immediately if grid draw exceeds your limit (safety priority).
6.  **The Finish**: You unplug or the car is full. `Auto Stop` detects the drop in power draw, turns off the logic, and resets the car to `13 Amps` for standard charging.

---

## 📥 Direct Installation (One-Click)

Click the buttons below to import the blueprints directly into your Home Assistant instance:

| Blueprint | Import Link |
| :--- | :--- |
| **1. Master Control** | [![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fcrryp%2Fha-hw-ev-loadbalancing-automation%2Fblob%2Fmain%2Fblueprints%2Fautomations%2Fev_master_control.yaml) |
| **2. Increase Amps** | [![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fcrryp%2Fha-hw-ev-loadbalancing-automation%2Fblob%2Fmain%2Fblueprints%2Fautomations%2Fev_increase_amps.yaml) |
| **3. Decrease Amps** | [![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fcrryp%2Fha-hw-ev-loadbalancing-automation%2Fblob%2Fmain%2Fblueprints%2Fautomations%2Fev_decrease_amps.yaml) |
| **4. Auto Start** | [![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fcrryp%2Fha-hw-ev-loadbalancing-automation%2Fblob%2Fmain%2Fblueprints%2Fautomations%2Fev_auto_start.yaml) |
| **5. Auto Stop** | [![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fcrryp%2Fha-hw-ev-loadbalancing-automation%2Fblob%2Fmain%2Fblueprints%2Fautomations%2Fev_auto_stop.yaml) |


## 🆘 Troubleshooting
* **"It’s sunny but nothing is happening!"** Check if the surplus has been steady for a full 5 minutes. The system ignores "short bursts" of sun to keep your hardware safe.
* **Car won't start?** Ensure the `EV Cable Connected` sensor is correctly mapped and showing "on." The system will not attempt a start if it cannot verify the car is plugged in.
* **Automations toggling off prematurely?** Check the `Auto Stop` threshold. If your car draws 0W while asleep or calculating, the system may assume it is finished.
* **API Errors?** The 5-minute delay is designed to respect car API limits (like Tesla) and prevent account lockouts or "Rate Limit" errors.

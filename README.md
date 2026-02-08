# 🏎️ Universal EV Smart Load Balancer
### A Modular Home Assistant Ecosystem for Intelligent Solar Charging

This ecosystem is a professional-grade suite of Home Assistant tools designed to synchronize your Electric Vehicle (EV) charging with your home's energy production. By monitoring your **HomeWizard P1 Meter** and the **HomeWizard Plug-in Battery**, this system ensures you charge using **only** surplus energy. 

While the Plug-in Battery is optional, the system is designed to work with it for the **best results**, allowing for smoother charging transitions and superior self-consumption. The primary goal of this project is **Zero Export & Zero Import**: making sure every watt of solar power goes into your car, and preventing the car from drawing expensive power from the grid when the sun disappears.

---

## ⚠️ Important Disclaimer
* **Plain YAML files:** These have been **extensively tested** and are confirmed to be stable for daily use.
* **Blueprints:** These are currently **UNTESTED**. They are provided for convenience, but for a guaranteed stable experience, please use the manual YAML method described in the installation guide.

---

## 💎 Key Features
* **Brand Agnostic**: Works with any EV (Tesla, Kia, Hyundai, etc.) or Charger (Wallbox, Easee, Zaptec) integrated into Home Assistant.
* **Solar Surplus Focus**: Automatically starts charging only when your solar panels produce more than your house currently needs.
* **The "Minimum-Amp" Cut-off**: If the system hits your minimum set limit (e.g., 5A) and you are still importing power from the grid, it will **stop charging immediately** to avoid buying grid energy.
* **Battery Optimized**: Designed to integrate with the **HomeWizard Plug-in Battery** for the most stable and efficient energy balancing.
* **Zero-Waste Charging**: Dynamically adjusts current (+1/-1 Amp) every 5 minutes based on real-time solar availability.
* **Cloud Protection**: Intelligent 5-minute delays prevent the charger from toggling on/off every time a cloud passes, protecting your car's hardware.
* **Auto-Pilot**: Automatically starts when surplus is detected and stops when the car is full or unplugged.

---

## 🛠️ Prerequisites & Hardware
To use this system, you need the following:

### 1. Energy Monitoring
* **HomeWizard P1 Meter**: To monitor live solar return (Export) and house consumption (Import).
* **HomeWizard Plug-in Battery** (*Optional but Recommended*): To monitor energy flow into/out of your home batteries for optimized balancing.

### 2. EV Integration
Your car or charger integration must provide:
* **Current Control**: A `number` entity to set Amperes (e.g., `number.tesla_charge_current`).
* **Start/Stop Switch**: A `switch` or `button` to toggle charging.
* **Connection Sensor**: A sensor showing if the cable is plugged in.

### 3. Helpers (Manual Step)
Go to **Settings > Devices & Services > Helpers** and create:
1.  **Input Boolean**: `input_boolean.load_balancing_ev` (The "Master Switch").
2.  **Input Boolean**: `input_boolean.load_balancing_ev_sun` (The "Sun Mode" flag).

---

## 📥 Installation Guide

### Option 1: Manual YAML (Recommended & Tested)
1.  Open your Home Assistant configuration folder.
2.  Navigate to `/blueprints/automation/` (or your `automations.yaml`).
3.  Copy the verified YAML code from the `automations/` folder in this repository.
4.  Replace the placeholder entity IDs with your own sensors.
5.  Go to **Developer Tools > YAML** and click **Automations**.

### Option 2: Direct Installation (Blueprints - Untested)
Click the buttons below to import the blueprints directly. **Please note: these are not yet verified for stability.**

| Blueprint | Import Link |
| :--- | :--- |
| **1. Master Control** | [![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fcrryp%2Fha-hw-ev-loadbalancing-automation%2Fblob%2Fmain%2Fblueprints%2Fautomations%2Fev_master_control.yaml) |
| **2. Increase Amps** | [![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fcrryp%2Fha-hw-ev-loadbalancing-automation%2Fblob%2Fmain%2Fblueprints%2Fautomations%2Fev_increase_amps.yaml) |
| **3. Decrease Amps** | [![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fcrryp%2Fha-hw-ev-loadbalancing-automation%2Fblob%2Fmain%2Fblueprints%2Fautomations%2Fev_decrease_amps.yaml) |
| **4. Auto Start** | [![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fcrryp%2Fha-hw-ev-loadbalancing-automation%2Fblob%2Fmain%2Fblueprints%2Fautomations%2Fev_auto_start.yaml) |
| **5. Auto Stop** | [![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fcrryp%2Fha-hw-ev-loadbalancing-automation%2Fblob%2Fmain%2Fblueprints%2Fautomations%2Fev_auto_stop.yaml) |

---

## 🚀 How it Works (The Logic Flow)
The system follows a strict power-tracking logic to ensure you only use solar energy:

1.  **The Start Trigger**: Charging begins if the system detects a stable surplus for 5 minutes of:
    * **-1200W** back to the Grid (Export)
    * **OR 1200W** flowing into the HomeWizard Battery.
    * *This threshold provides the necessary room to start the EV at the minimum 5A load.*

2.  **Stepping Up (Surplus Logic)**: 
    * If the system detects **250W** still flowing into the HomeWizard Battery, it will **increase** the charging speed by **+1 Amp**.
    * It will continue to step up until the maximum limit (e.g., **13A**) is reached.

3.  **Stepping Down (Grid Usage Logic)**: 
    * If the system detects **250W** of usage from the Grid (Import), it will **decrease** the charging speed by **-1 Amp**.

4.  **The Cut-off**: 
    * If the charger is already at its minimum (**5A**) and the grid usage still requires a step down, the system will **STOP** charging entirely. This prevents buying grid power to maintain the car's minimum charging speed.

5.  **Restart**: The system resets and waits for the initial **1200W surplus** (Step 1) to reappear before starting a new cycle.

---
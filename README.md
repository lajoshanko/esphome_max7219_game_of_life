# Game of Life – MAX7219 (ESPHome) #

## UPDATE TO THE MODELS!
I have created an upper corner version for the Lab Rax rack

---

This project drives a **8×32 MAX7219 LED matrix (4 chained modules)** controlled by a **NodeMCU (ESP8266)** 
as a replacement for a **10-inch 1U rack blank plate** with a 3d printed panel. 

It runs a fully automated version of **Conway’s Game of Life**, with:

- Automatic reseeding when the simulation dies, freezes, or enters a loop  
- Adjustable brightness, speed, and initial seed complexity  
- Full Home Assistant control via ESPHome  
- A clean front-panel animation ideal for server racks, studios, or home labs

The firmware continuously generates living pixel patterns—organic, surprising, and perfect for visual ambience in a rack setup.

---

## Features

- **Game of Life simulation** on an 8×32 LED matrix  
- Automatically detects:
  - Dead boards
  - Still lifes
  - Oscillators (periods 1–8)  
- Hands-on Home Assistant controls:
  - Initial cell count
  - Speed
  - Brightness / intensity
  - Manual reseed
  - Generation counter  
- Pure ESPHome implementation (no external libraries needed)

---

## 3D Models

[The upper corne model can be found here](https://makerworld.com/en/models/2043316-1u-10-inch-panel-cover-for-max7219)

<img src="https://github.com/user-attachments/assets/ce65a107-d396-48ae-a17f-53504dc08f17" alt="Alt Text" style="width:50%; height:auto;">
<img src="https://github.com/user-attachments/assets/9d0548b4-b881-4d33-b863-7c8e39546d94" alt="Alt Text" style="width:50%; height:auto;">

[The 10" 1 U panel model can be found here](https://makerworld.com/en/models/2043316-1u-10-inch-panel-cover-for-max7219) for the panel(Still in prototyping phase, so any feedback is welcome)

<img src="https://github.com/user-attachments/assets/085b47c2-a426-47de-8f96-0789923b36ef" alt="Alt Text" style="width:50%; height:auto;">
<img src="https://github.com/user-attachments/assets/2b944c0f-ced1-4b70-b196-891e44c1a80c" alt="Alt Text" style="width:63%; height:auto;">
<img src="https://github.com/user-attachments/assets/9b6e629b-ddf9-4a2e-89ab-f45784e187c8" alt="Alt Text" style="width:33%; height:auto;">

---

# Wiring Guide

This project uses a **NodeMCU ESP8266 dev board** and a **MAX7219 LED matrix**.

## **Connections**

| NodeMCU Pin | MAX7219 Pin |
|-------------|-------------|
| **GPIO14 (D5)** | CLK |
| **GPIO13 (D7)** | MOSI (DIN) |
| **GPIO12 (D6)** | CS |
| 5V | VCC |
| GND | GND |

---

## **ESPHome Wiring Snippet**

```yaml
spi:
  clk_pin: GPIO14      # D5 → CLK
  mosi_pin: GPIO13     # D7 → DIN

display:
  - platform: max7219digit
    cs_pin: GPIO12     # D6 → CS
    num_chips: 4       # 8x32 panel
```

# Home Assistant Controls

The firmware exposes multiple interactive controls and sensors.

---

## Numbers (Adjustable Controls)

### **GOL Initial Groups**  
**Entity:** `number.gol_initial_groups_number`  
Defines how many random “plus-shaped” seed clusters are placed at startup or reseed.  
Higher values = more chaotic starting state.

---

### **GOL Speed**  
**Entity:** `number.gol_speed_number`  
**Range:** 1–10  
- Higher = **faster simulation**  
- Internally mapped to a frame delay

---

### **Matrix Intensity**  
**Entity:** `number.matrix_intensity_number`  
**Range:** 0–15  
Controls the brightness of the MAX7219 matrix.  
Useful for dark environments or power optimization.

---

## Button (Manual Reseed)

### **GOL Reseed**  
**Entity:** `button.gol_reseed_button`  
Immediately clears the simulation and seeds new random cells.

---

## Sensor (Read-Only)

### **GOL Generation**  
**Entity:** `sensor.gol_generation_sensor`  
Counts how many generations have passed in the current simulation.  
Resets during reseed or auto-restart when the simulation gets stuck.

---

# How the Simulation Works

The display lambda runs a full Game of Life engine:

- Computes neighbors for each cell  
- Applies Conway’s rules  
- Tracks last **8 generations** to detect:
  - Still lifes  
  - Death  
  - Oscillators (period 1–8 loops)  
- Automatically reseeds when:
  - The board is empty  
  - The board stops changing  
  - The board enters a repeating loop  

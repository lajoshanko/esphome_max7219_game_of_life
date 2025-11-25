# Game of Life – MAX7219 (ESPHome)

This project drives a **8×32 MAX7219 LED matrix (4 chained modules)** controlled by a **NodeMCU (ESP8266)** 
and is used as a replacement for a **10-inch 1U rack blank plate replacement** with a 3d printed panel. 

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

##� Sensor (Read-Only)

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

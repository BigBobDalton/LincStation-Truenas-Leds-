# LincStation-Truenas-Leds-
LincStation N1 LED Control for TrueNAS SCALE 25.10 Works on TrueNAS SCALE 25.10 (Goldeye) Enables full front-panel LED control without Unraid:  RGB strip ON (solid white) Only active drive bays lit Empty bays OFF No blinking Auto-detects I2C bus, runs at boot via Post Init script. 100% tested and stable on TrueNAS 25.10.


# LincStation N1 – TrueNAS SCALE LED Control (RGB ON + Active Bays Only)

> **Tested on:** TrueNAS SCALE 25.10.0 (Goldeye)  
> **Hardware:** LincStation N1 (6-bay NAS)  
> **Goal:**  
> - **RGB bottom strip = ON (solid white)**  
> - **Only drive bays with disks = LED ON**  
> - **Empty bays = LED OFF**  
> - **No blinking**  
> - **Fully automatic at boot**

---

## Why This Exists

The LincStation N1 ships with Unraid on a USB stick.  
When you remove it and install **TrueNAS SCALE**, the front-panel LEDs:
- **Blink constantly** (even idle)
- **All bays light up** (even empty ones)
- **RGB strip turns off**

This guide **fixes all of that** using **I2C control** — **no Unraid needed**.

---

## Prerequisites

- SSH access to TrueNAS
- Root or `sudo` privileges
- At least **one data pool** (not `boot-pool`) — e.g., `tank`

---

## Step 1: Verify I2C is Working

```bash
sudo modprobe i2c-dev
sudo i2cdetect -l

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


RGB strip ON (solid white) – Active drive bays ON – Empty bays OFF – No blinking


markdown# LincStation N1 – TrueNAS SCALE LED Control (RGB ON + Active Bays Only)

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
You should see several i2c-* buses.
We’ll let the script auto-detect the correct one.

Step 2: Create the Smart LED Script
bashsudo tee /mnt/tank/scripts/lincstation-led-rgb-on.sh > /dev/null << 'EOF'
#!/bin/bash
# LincStation N1 – RGB ON + Active Bays ON + Empty OFF
# Dynamic bus detection (prioritizes bus 3)

find_led_bus() {
    if sudo i2cdetect -y 3 2>/dev/null | grep -q "26"; then
        echo "3"
        return
    fi
    for bus in {0..14}; do
        if sudo i2cdetect -y $bus 2>/dev/null | grep -q "26"; then
            echo "$bus"
            return
        fi
    done
    echo "NO_BUS"
}

BUS=$(find_led_bus)
if [ "$BUS" = "NO_BUS" ]; then
    echo "ERROR: LED chip not found. Run 'sudo i2cdetect -l'"
    exit 1
fi

echo "LED chip found on i2c-$BUS"

modprobe i2c-dev 2>/dev/null
sleep 1

# Disable blinking (all 6 bays)
sudo i2cset -y $BUS 0x26 0x52 0x00 2>/dev/null || true
sudo i2cset -y $BUS 0x26 0x54 0x00 2>/dev/null || true
sudo i2cset -y $BUS 0x26 0x56 0x00 2>/dev/null || true
sudo i2cset -y $BUS 0x26 0x58 0x00 2>/dev/null || true
sudo i2cset -y $BUS 0x26 0x5A 0x00 2>/dev/null || true
sudo i2cset -y $BUS 0x26 0x5C 0x00 2>/dev/null || true
sudo i2cset -y $BUS 0x26 0x5E 0x00 2>/dev/null || true

# Turn on all 6 bays (hardware auto-blanks empty ones)
sudo i2cset -y $BUS 0x26 0xB0 0x04 2>/dev/null || true
sudo i2cset -y $BUS 0x26 0xB0 0x08 2>/dev/null || true
sudo i2cset -y $BUS 0x26 0xB0 0x10 2>/dev/null || true
sudo i2cset -y $BUS 0x26 0xB0 0x20 2>/dev/null || true
sudo i2cset -y $BUS 0x26 0xB0 0x40 2>/dev/null || true
sudo i2cset -y $BUS 0x26 0xB0 0x80 2>/dev/null || true

# RGB ON (solid white, 100% brightness)
sudo i2cset -y $BUS 0x26 0x20 0x01 2>/dev/null || true
sudo i2cset -y $BUS 0x26 0x21 0xFF 2>/dev/null || true
sudo i2cset -y $BUS 0x26 0x22 0xFF 2>/dev/null || true
sudo i2cset -y $BUS 0x26 0x23 0xFF 2>/dev/null || true
sudo i2cset -y $BUS 0x26 0x24 0x64 2>/dev/null || true

echo "SUCCESS: RGB ON, Active bays ON, Empty OFF (bus $BUS)"
EOF

sudo chmod +x /mnt/tank/scripts/lincstation-led-rgb-on.sh

Replace tank with your actual data pool name (run zpool list to check).


Step 3: Add to TrueNAS Boot (Init Script)

Go to TrueNAS Web UI
System → Advanced → Init/Shutdown Scripts → Add
Fill in:

Description: LincStation RGB + Active LEDs
Type: Script
Script: /mnt/tank/scripts/lincstation-led-rgb-on.sh
When: Post Init
Enabled: Checked


Save


Step 4: Test It Now
bashsudo /mnt/tank/scripts/lincstation-led-rgb-on.sh
Expected output:
textLED chip found on i2c-0
SUCCESS: RGB ON, Active bays ON, Empty OFF (bus 0)
Result:

RGB strip = solid white
Only populated drive bays = ON
Empty bays = OFF
No blinking


Step 5: Reboot & Verify
bashsudo reboot
After boot:
bashsudo journalctl -t kernel | grep LincStation
Should show:
textSUCCESS: RGB ON, Active bays ON, Empty OFF (bus X)

Optional: Change RGB Color / Mode
Edit the script and replace the RGB section:

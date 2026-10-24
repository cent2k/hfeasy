# HFeasy Firmware Build Instructions with MQTT Retain Support

## Summary of Changes
- ✅ Modified `src/config.h` - Added `mqtt_retain` config field
- ✅ Modified `src/mqttcli.c` - Updated publish function with retain support  
- ✅ Modified `src/mqttcli.h` - Updated function signature
- ✅ Modified `src/config.c` - Added HTML checkbox for retaining messages

## Prerequisites
1. **ARM toolchain:** `/opt/gcc-arm-none-eabi-7-2018-q2-update/bin/`
2. **ESP32 SDK 4.13** at `./sdk/4.13/`
3. **System requirements:** make, gcc, objcopy

## Build Environment Setup

### Option 1: Install Correct Toolchain (Recommended)
```bash
# Download ARM toolchain from official sources:
wget https://developer.arm.com/-/media/Files/downloads/gnu/7-2018q2/gcc-7-2018-q2-negative-linux-x86_64.tar.gz
tar xzf gcc-7-2018-q2-negative-linux-x86_64.tar.gz
mv gcc-7-2018-q2-negative /opt/gcc-arm-none-eabi-7-2018-q2-update

# Verify installation:
/opt/gcc-arm-none-eabi-7-2018-q2-update/bin/arm-none-eabi-gcc --version
```

### Option 2: Workaround with System Toolchain
If the proper toolchain is unavailable, use this workaround:
```bash
export CROSS_COMPILE=arm-none-eabi-
export GCC_PATH=/usr
cd /home/centurion/hfeasy
make clean
make all
```

Note: You may need to modify CFLAGS to avoid system header conflicts.

## Build Commands

```bash
cd /home/centurion/hfeasy
make clean
make all
```

## Output Files

After successful build, binary files will be in:
- `projects/build_lpxx30/HFEASY.bin` - Serial upload binary
- `projects/build_lpxx30/HFEASY_UPGRADE.bin` - Web upgrade image

## Flash to Device

### Via USB Serial (using ESP-IDF or esptool.py)
```bash
# Install esptool if needed:
pip install esptool

# Upload firmware:
esptool --chip esp32 write_flash 0x1000 projects/build_lpxx30/HFEASY.bin
```

### Via Web Upgrade (built-in OTA)
1. Upload `HFEASY_UPGRADE.bin` to device web interface
2. Reboot device after upload

## Configuration
After flashing, configure MQTT via web interface at `http://<device_ip>/config_mqtt`:
- Server IP/host: Your MQTT broker address
- Port: Usually 1883
- Username/Password: Optional authentication
- Topic: hfeasy_<mac_addr_suffix>
- QoS: 0, 1, or 2
- **Retain message**: Check this box to enable retain flag on all publishes

## Testing MQTT Retain

Once configured, test the retain functionality:

```bash
# Install mosquitto if not already installed
sudo apt install mosquitto

# Start MQTT broker
sudo systemctl start mosquitto

# Connect new subscriber:
mosquitto_sub -h 192.168.1.x -t "stat/POWER" -v

# Device publishes state with RETAIN flag
# New subscriber receives last known state immediately
```

## Troubleshooting

### Build fails with header errors:
- Use correct toolchain at `/opt/gcc-arm-none-eabi-7-2018-q2-update`
- Avoid using system newlib headers by isolating SDK includes

### Device doesn't connect to MQTT:
- Verify device IP address
- Check MQTT broker is running and accessible
- Review `/status` endpoint for connection status
- LED indicator on device shows MQTT connection state

## Files Modified

```
src/config.h          # Added mqtt_retain field
src/mqttcli.c         # Updated publish function
src/mqttcli.h         # Updated function declaration  
src/config.c          # Added HTML checkbox and parsing
MQTT_RETAIN_CHANGES.md # Documentation of changes
```

## Backward Compatibility

- Default behavior unchanged (retain disabled)
- Existing configurations work without modification
- No changes needed for non-MQTT control methods

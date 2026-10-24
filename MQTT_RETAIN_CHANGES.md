# MQTT Retain Support - Changes Summary

This document describes the changes made to add MQTT retain message support to HFeasy firmware.

## What is MQTT Retain?

The MQTT retain flag tells the broker to store the last published message for a topic and deliver it to new subscribers when they connect. This is useful for state synchronization so that devices reconnecting to the network get their latest known state.

## Changes Made

### 1. `src/config.h`
Added `mqtt_retain` field to configuration structure:

```c
struct hfeasy_config {
    // ... existing fields ...
    char      mqtt_topic[30];
    char      mqtt_full_topic[50];
    uint8_t   mqtt_qos;
    uint8_t   mqtt_retain;      /* NEW */
    char      mqtt_on_value[10];
    char      mqtt_off_value[10];
};
```

### 2. `src/mqttcli.h`
Updated function signature:

```c
/* OLD */
void USER_FUNC mqttcli_publish(char *value, char *sufix);

/* NEW */
int USER_FUNC mqttcli_publish(char *value, char *sufix, uint8_t retain);
```

### 3. `src/mqttcli.c`
Modified `mqttcli_publish()` to:
- Accept a `retain` parameter (config + runtime override)
- Set MQTT_PUBLISH_RETAIN flag when enabled
- Return error status instead of void

Also added initialization in `mqttcli_initcfg()`:
```c
state->cfg.mqtt_retain = 0; /* default to no retain */
```

### 4. `src/config.c`
Added:
- HTML checkbox for "Retain message" on MQTT config page
- Parse `retain` parameter from GET request
- Save to `state.cfg.mqtt_retain`
- Show checkbox checked if enabled in the form display

## Usage

### Via Web Interface
1. Navigate to `/config_mqtt` on your HFeasy device
2. Check "Retain message" checkbox
3. Click "Apply" and save configuration

### Via Direct API Call
```
GET /config_mqtt?host=your.mosquitto.org&port=1883&qos=0&retain=1
```

## Build Instructions

To build the firmware:

1. **Prerequisites:**
   - ARM cross-compiler toolchain (arm-none-eabi-gcc or similar)
   - ESP32/ESP8266 SDK version 4.13+
   - Make and objcopy utilities

2. **Build commands:**
   ```bash
   cd /home/centurion/hfeasy
   make clean
   make all
   ```

3. **Output files:**
   - `projects/build_lpxx30/HFEASY.bin` - Binary for serial upload
   - `projects/build_lpxx30/HFEASY_UPGRADE.bin` - Upgrade image for web upload

## Technical Details

The retain flag is now combined with the QoS level in the MQTT PUBLISH packet:
- QoS 0 + Retain: Single delivery, message persists after publish
- QoS 1 + Retain: At-least-once delivery, message persists  
- QoS 2 + Retain: Exactly-once delivery, message persists

This applies to all state publishes (ON/OFF commands) when the retain option is enabled.

## Backward Compatibility

- Default behavior unchanged (`mqtt_retain = 0`)
- Existing configurations continue to work
- No changes required for devices using non-MQTT control methods

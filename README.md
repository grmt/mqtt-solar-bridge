# mqtt-solar-bridge

A small MQTT client that subscribes to a smart plug's power measurements and writes them as `solar.json` to the Toon's built-in web server, where the [SolarPanel](https://github.com/oepi-loepi/solarPanel) app can read them.

## How it works

The smart plug sits in the socket in front of the inverter, measures power output, and publishes its readings via MQTT. This bridge subscribes to those topics and writes a JSON file to `/qmf/www/tsc/solar.json`. The Toon's web server then serves that file at `http://localhost/tsc/solar.json`, which the SolarPanel plugin reads.

```
Smart plug → MQTT broker → mqtt-solar-bridge → solar.json → Toon web server → SolarPanel
```

The MQTT broker (e.g. mosquitto) can run on the Toon itself or anywhere else on the network.

## MQTT topics

The bridge expects the smart plug to publish on these topics (where `smart-plug-pv` is the configurable base topic):

| Topic | Value |
|---|---|
| `smart-plug-pv/power/get` | Current power in W |
| `smart-plug-pv/energycounter_today/get` | Energy today in Wh |
| `smart-plug-pv/energycounter/get` | Total energy in Wh |

## Pre-built binaries

The `bin/` and `lib/` directories contain pre-built binaries for the **Toon** (Freescale i.MX6, ARMv7, musl libc), compiled with WolfSSL for TLS support:

| File | Description |
|---|---|
| `bin/mosquitto` | MQTT broker |
| `bin/mosquitto_pub` | MQTT publish client |
| `bin/mosquitto_sub` | MQTT subscribe client |
| `lib/libwolfssl.so.44.2.0` | WolfSSL TLS runtime |
| `lib/libc.so` | musl libc runtime |

The broker and clients are optional — if you already have mosquitto running on your network, only `mqtt-solar-bridge` itself is needed.

## Installation on Toon

1. Copy everything to the Toon's data partition:
   ```sh
   cp mqtt-solar-bridge /mnt/data/tsc/mqtt-solar-bridge
   cp bin/mosquitto bin/mosquitto_pub bin/mosquitto_sub /mnt/data/tsc/mosquitto/
   cp lib/libwolfssl.so.44.2.0 lib/libc.so /mnt/data/tsc/lib/
   cp mosquitto.conf.example /mnt/data/tsc/mosquitto/mosquitto.conf
   ```

2. Copy and edit the bridge config (only needed if defaults don't work):
   ```sh
   cp mqtt-solar-bridge.conf.example /mnt/data/tsc/mqtt-solar-bridge.conf
   ```

3. The `S99tsc.sh` script in [toon-config](https://github.com/grmt/toon-config) installs the binaries to their runtime locations and starts everything automatically via `/etc/inittab`.

## Configuration

All settings are optional. Without a config file, the bridge connects to `localhost:1883` and subscribes to `smart-plug-pv/#`.

Copy [`mqtt-solar-bridge.conf.example`](mqtt-solar-bridge.conf.example) to `/mnt/data/tsc/mqtt-solar-bridge.conf` and uncomment what you need:

```sh
# MQTT broker host and port (defaults: localhost, 1883)
MQTT_HOST=192.168.1.10
MQTT_PORT=1883

# Credentials (leave commented out if not required)
MQTT_USER=myuser
MQTT_PASS=mypassword

# Base topic of the smart plug (default: smart-plug-pv)
MQTT_BASE_TOPIC=smart-plug-pv

# TLS/MQTTS — set to true to enable; default port becomes 8883
#MQTT_TLS=false
#MQTT_CAFILE=/mnt/data/tsc/certs/ca.crt
```

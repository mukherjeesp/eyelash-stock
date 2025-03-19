# TrackPoint Support for Eyelash Corne

This guide explains how to add TrackPoint support to your Eyelash Corne keyboard.

## Files Overview

1. **trackpoint.overlay** - Device tree overlay that defines the PS/2 hardware interface
2. **trackpoint.conf** - Configuration file with necessary Kconfig options
3. **eyelash_corne_trackpoint.keymap** - Sample keymap with TrackPoint support
4. **build.yaml.trackpoint** - Build configuration example

## Hardware Requirements

To use a TrackPoint with your Eyelash Corne, you'll need:

1. A compatible TrackPoint module (like one from a Lenovo T440 or similar)
2. Connections to the following pins on the RIGHT half:
   - **SCL (Clock)**: P0.16
   - **SDA (Data)**: P0.18
   - **RST (Reset)**: P0.15 (optional if you have a reset circuit)

## TrackPoint Wiring

TrackPoint modules typically have 4-6 pins:
- **VCC** - Connect to 3.3V or 5V (depending on your TrackPoint model)
- **GND** - Connect to GND
- **DATA** - Connect to PIN P0.18
- **CLK** - Connect to PIN P0.16
- **RESET** - Connect to PIN P0.15 (or use a reset circuit)

## Reset Circuit Option

You can either:
1. Use a GPIO pin for reset (P0.15 as configured)
2. Implement a hardware reset circuit:
   - Use a capacitor (1μF) and resistor (10kΩ) in series from VCC to RESET
   - This automatically performs the power-on reset sequence without needing a GPIO pin

## Installation Instructions

1. Copy these files to your ZMK config:
   - `boards/arm/eyelash_corne/trackpoint.overlay`
   - `boards/arm/eyelash_corne/trackpoint.conf`
   - `config/eyelash_corne_trackpoint.keymap` (or integrate into your existing keymap)

2. Build using one of these methods:
   - Rename `build.yaml.trackpoint` to `build.yaml` and build normally
   - Or manually specify the build flags:
     ```
     west build -b eyelash_corne_right -- -DEXTRA_CONF_FILE=boards/arm/eyelash_corne/trackpoint.conf -DOVERLAY_CONFIG=boards/arm/eyelash_corne/trackpoint.overlay -DKEYMAP_FILE="eyelash_corne_trackpoint.keymap"
     ```

## Customizing TrackPoint Behavior

You can adjust TrackPoint settings in two ways:

1. **Runtime Adjustment** - Use the TrackPoint Settings layer (TP_SET)
   - Access this by using the combo defined in keymap (keys 13-17 simultaneously)
   - Adjust sensitivity, speed, and other parameters
   - Settings will be saved to flash after 60 seconds

2. **Compile-time Adjustment** - Edit settings in the keymap file:
   ```
   &mouse_ps2 {
       tp-sensitivity = <135>;      // Adjust sensitivity (higher = more sensitive)
       tp-neg-inertia = <6>;        // Adjust negative inertia
       tp-val6-upper-speed = <152>; // Maximum speed
   };
   ```

## Troubleshooting

If your TrackPoint isn't working:

1. **Check connections** - Verify all wires are correctly connected
2. **Check logs** - Look for "Could not send reset cmd" or other PS/2 errors
3. **Reset issues** - If using a GPIO for reset, try a hardware reset circuit instead
4. **Pin conflicts** - Ensure no other peripherals are using the same pins
5. **Timing issues** - If experiencing Bluetooth disconnections, try using different pins or adjusting interrupt priorities

## Pin Selection Alternatives

If the default pins don't work for your setup, you can modify `trackpoint.overlay` to use different pins:

```c
// Alternative high-frequency pins
#define MOUSE_PS2_PIN_SCL <&gpio0 2 GPIO_ACTIVE_HIGH>   // P0.02
#define MOUSE_PS2_PIN_SDA <&gpio0 4 GPIO_ACTIVE_HIGH>   // P0.04
#define MOUSE_PS2_PIN_SDA_PINCTRL <NRF_PSEL(UART_RX, 0, 4)>
```

Make sure to choose pins that are not already in use for the keyboard matrix, encoder, display, or RGB lighting.

# Component Health Checker

Firmware prototype for an STM32-based electronic component health tester. The current implementation measures resistance using ADC1 on PA0 and a 1 kΩ reference resistor.

## Current measurement setup

Wire the divider as follows:

```text
3.3 V ── 1 kΩ reference ── PA0 (ADC1_IN0) ── unknown resistor ── GND
```

The firmware stores the calculated resistance in `resistance_ohms` in `Core/Src/main.c`. For a 12-bit ADC reading `adc_raw`, it uses:

```text
R_unknown = 1000 Ω × adc_raw / (4095 − adc_raw)
```

At full-scale ADC input, the result is set to `UINT32_MAX` to indicate the measurement is out of range (for example, an open circuit).

## Build

Open this project in STM32CubeIDE and build the `Debug` configuration. The CubeMX configuration is in `Tester_MX.ioc`.

## Safety and limitations

- Keep PA0 between GND and VDDA; do not apply a voltage above VDDA.
- The divider supply should be the same 3.3 V rail used as the ADC reference for the ratio calculation to be accurate.
- This is an early prototype: it currently measures resistance only. Component-specific health checks are not implemented yet.

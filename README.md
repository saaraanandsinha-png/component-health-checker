# Component Health Checker

**A compact embedded test platform for checking electronic components—starting with resistor measurements.**

Component Health Checker is an STM32 firmware prototype designed to grow into a practical bench tool: insert a component, run the appropriate electrical test, and use the result to assess its condition. The current firmware establishes the measurement core by reading a resistor-divider voltage and calculating the unknown resistance.

> **Current status:** resistor measurement is implemented. Component-specific health classifications and support for other component types are future work.

## What it does today

- Samples **ADC1 channel 0 (PA0)** on an STM32F401CCU6 using the existing STM32CubeMX configuration.
- Calculates the unknown resistor value and continuously updates the debugger-visible `resistance_ohms` variable.
- Handles an ADC full-scale reading as out of range by assigning `UINT32_MAX`.

## Measurement circuit

Use the divider orientation below:

```text
3.3 V ── 1 kΩ reference ──┬── PA0 (ADC1_IN0)
                          |
                    Unknown resistor
                          |
                         GND
```

For the 12-bit ADC reading `adc_raw` (0–4095), the firmware calculates:

```text
R_unknown = 1,000 Ω × adc_raw / (4095 − adc_raw)
```

The ratio calculation assumes the divider is powered from the same 3.3 V rail used as the ADC reference. A full-scale reading corresponds to an open-circuit or otherwise out-of-range measurement; the code reports this with `UINT32_MAX`.

## Build and inspect

1. Open the project in **STM32CubeIDE**.
2. Build the `Debug` configuration and flash the STM32F401CCU6.
3. Start a debug session, resume execution, and add `resistance_ohms` to **Live Expressions**.

The CubeMX project configuration is `Tester_MX.ioc`; the measurement code is in `Core/Src/main.c`.

## Safety and measurement notes

- Test components only in an **unpowered circuit**; ideally remove the component or isolate one lead to avoid parallel paths affecting the reading.
- Keep PA0 between GND and VDDA. Never apply a voltage above VDDA.
- ADC quantization, resistor tolerance, supply/reference mismatch, and wiring affect accuracy. Near full scale, the resistance estimate becomes increasingly sensitive to small ADC errors.
- The current firmware measures resistance only. It does not yet decide whether a component is healthy against a nominal value or test diodes, capacitors, transistors, or ICs.

## Roadmap

- Compare measured resistance against a selected nominal value and tolerance.
- Add safe, component-specific test routines and pass/fail guidance.
- Add a user interface for selecting a component and presenting results.

# zephyr-custom-soc
A Zephyr Project workspace containing support for custom SoC. Serves a guide to bring-up support for custom SoC.
The SoC chosen for demonstration is STM32F407VGTx MCU which is an ARM-Cortex MF4 microcontroller. This guide provides base
driver support for SoC (clock-control, serial, gpio, pinctrl, interrupt-controller drivers only).

The custom soc vendor is referred to as **cmcu**. The family to which the custom soc belongs is **cmcuf** and the series is
**cmcus**.

# Bring-up procedure

[1] Develop the required DTS binding files for the peripherals of the SoC under [${ZEPHYR_BASE}/zephyr/dts/bindings].
   The following YAML files for DTS binding files were created for the custom MCU:

_./mtd/cmcu,cmcu-nv-flash-base.yaml
./mtd/cmcu,cmcu-nv-flash.yaml
./flash_controller/cmcu,cmcu-flash-controller.yaml
./clock/cmcu,cmcu-hse-clock.yaml
./clock/cmcu,cmcu-pll-clock.yaml
./clock/cmcu,cmcu-rcc.yaml
./gpio/cmcu,cmcu-gpio.yaml
./serial/cmcu,cmcu-uart-base.yaml
./serial/cmcu,cmcu-usart.yaml
./serial/cmcu,cmcu-uart.yaml
./pinctrl/cmcu,cmcu-pinctrl.yaml
./reset/cmcu,cmcu-rcc-rctl.yaml
./interrupt-controller/cmcu,cmcu-exti.yaml
_

<img width="1102" height="320" alt="image" src="https://github.com/user-attachments/assets/f903de35-8d74-4700-97f7-22a0c768172d" />

The YAML files for DTS binding describe the properties under the node with the specified **compatible**. Each node in the devicetree
has a compatible property. When processing the devicetree of the SoC, the build system looks for a suitable YAML DTS binding file. For
this demonstration, the STM32 YAML DTS binding files are reused.

[2] Write the device tree source and device tree include files for the SoC


. 

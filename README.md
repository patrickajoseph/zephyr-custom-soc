# zephyr-custom-soc
A Zephyr Project workspace containing support for custom SoC. Serves a guide to bring-up support for custom SoC.
The SoC chosen for demonstration is STM32F407VGTx MCU which is an ARM-Cortex MF4 microcontroller. This guide provides base
driver support for SoC (clock-control, serial, gpio, pinctrl, interrupt-controller drivers only).

The custom soc vendor is referred to as **cmcu**. The family to which the custom soc belongs is **cmcuf** and the series is
**cmcus**.

# Bring-up procedure

[1] Define SoCs specific files under [${ZEPHYR_BASE}/zephyr/soc/cmcu/cmcuf] and [${ZEPHYR_BASE}/zephyr/soc/cmcu/cmcuf/cmcus]

**cmcu** stands for custom mcu and cmcu is the vendor name. A SoC family named **cmcuf** (custom MCU family) is provided by
the vendor **cmcu**. Under the custom MCU family, there is a SoC series named **cmcus**.

The family SoC subdirectory ([${ZEPHYR_BASE}/zephyr/soc/cmcu/cmcuf]) contains the following files:

1. CMakeLists.txt             --      Includes all .c and .h files required for SoC configuration.
2. Kconfig                    --      Defines the property SOC_FAMILY_<SOC_FAMILY> (in this case SOC_FAMILY_CMCU) which is used for
                                      selection of other Kconfig properties. It also recursively adds other Kconfig files to the build.    
3. Kconfig.soc                --      This Kconfig file sets a mandatory Kconfig variable named SOC_FAMILY to the required SoC (in this case "cmcu").
4. Kconfig.defconfig          --      This default configuration file is used to setup basic properties required to get the SoC running and also
                                      for basic kernel configuration.
5. soc.yaml                   --      This is a very important file required by the build framework to identify the SoC.

[soc.yaml]

family:
- name: cmcuf
  series:
  - name: cmcus
    socs:
    - name: cmcusoc

This describes basic information about the SoC.

The family SoC subdirectory ([${ZEPHYR_BASE}/zephyr/soc/cmcu/cmcuf/cmcus]) contains the following files:

1. CMakeLists.txt            --      Adds all the C include and source files to the build and invokes the linker.
2. Kconfig                   --      This configuration file selects architecture specific symbols for the SoC.
3. Kconfig.defconfig         --      Typically used to define the stack size of basic threads.
4. Kconfig.soc               --      This configuration file defines two important symbols named SOC and SOC_FAMILY which is required by the build system.
                                     It also defined the SoC specific symbols SOC_CMCUSOC and SOC_SERIES_CMCUS. It also defines the number of IRQs supported
                                     by the SoC using the symbol NUM_IRQS. 
5. pinctrl_soc.h             --      This header file contains the pin control typedef pinctrl_soc_pin_t and utility macros for pin control. This header file
                                     references another header file (${ZEPHYR_BASE}/include/zephyr/dt-bindings/pinctrl/cmcu_pinctrl.h) which contains
                                     the important macro CMCU_PINMUX() which is a macro used in the pin control DTSI file present in
                                     ${ZEPHYR_BASE}/../modules/hal/cmcu/dts/cmcu/cmcus/cmcu-pinctrl.dtsi
6. soc.c                     --      This is a C source file which contains a SoC initialization function named soc_early_init_hook() which is used to initialize
                                     the basic peripherals of the SoC (example: SyTick timer, ART accelerator etc.) 
7. soc.h                     --      This C header file links the SoC specific SDK to the Zephyr directory. For this demonstration, the SoC SDK files are defined under
                                     ${ZEPHYR_BASE}/../modules/hal/cmcu/cmcu_sdk.

[2] Create the SoC specific pin control device tree source include file & SDK support for the SoC


 
[1] Develop the required DTS binding files for the peripherals of the SoC under [${ZEPHYR_BASE}/zephyr/dts/bindings].
   The following YAML files for DTS binding files were created for the custom MCU:

./mtd/cmcu,cmcu-nv-flash-base.yaml
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


<img width="1102" height="320" alt="image" src="https://github.com/user-attachments/assets/f903de35-8d74-4700-97f7-22a0c768172d" />

The YAML files for DTS binding describe the properties under the node with the specified **compatible**. Each node in the devicetree
has a compatible property. When processing the devicetree of the SoC, the build system looks for a suitable YAML DTS binding file. For
this demonstration, the STM32 YAML DTS binding files are reused.

[2] Write the device tree source and device tree include files for the SoC

The DTS files for the SoC have to be created under []${ZEPHYR_BASE}/zephyr/dts/arm/cmcu/cmcus/].
Two files named [cmcus.dtsi] and [cmcusoc.dtsi] were created, one for the SoC series and one for the SoC.
Two helper header files [cmcu_clock.h] and [cmcu_reset.h] ( present in ${ZEPHYR_BASE}/include/zephyr/dt-bindings/clock/cmcu_clock.h & 
${ZEPHYR_BASE}/include/zephyr/dt-bindings/clock/cmcu_reset.h ) are used. The two header files contain help macros CMCU_CLOCK and
CMCU_RESET. The compatible property for each node was updated to use the custom mcu device tree bindings.

[cmcus.dtsi] --> custom SoC series peripheral device tree.
[cmcusoc.dtsi] --> custom SoC specific device tree settings.
. 

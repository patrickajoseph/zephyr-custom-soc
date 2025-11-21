# zephyr-custom-soc
A Zephyr Project workspace containing support for custom SoC. Serves a guide to bring-up support for custom SoC.
The SoC chosen for demonstration is STM32F407VGTx MCU which is an ARM-Cortex MF4 microcontroller. This guide provides base
driver support for SoC (clock-control, serial, gpio, pinctrl, interrupt-controller drivers only).

The custom soc vendor is referred to as **cmcu**. The family to which the custom soc belongs is **cmcuf** and the series is
**cmcus**.

**NOTE :** ${ZEPHYR_BASE} refers to ~/zephyrproject/zephyr.

# Setup procedure

The experimental code is present in ${ZEPHYR_BASE}/samples/basic/minimal.

1.	Build the code using **west build -p always -b cmcu_board**
2.	Flash the board using **bash zephyr-flash-arm.sh**
3.	Start a debug session using **bash zephyr-debug-arm.sh**

# Bring-up procedure

[1] Define SoCs specific files under ${ZEPHYR_BASE}/soc/cmcu/cmcuf and ${ZEPHYR_BASE}/soc/cmcu/cmcuf/cmcus

**cmcu** stands for custom mcu and cmcu is the vendor name. A SoC family named **cmcuf** (custom MCU family) is provided by
the vendor **cmcu**. Under the custom MCU family, there is a SoC series named **cmcus**.

The family SoC subdirectory [${ZEPHYR_BASE}/soc/cmcu/cmcuf] contains the following files:

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

The family SoC subdirectory [${ZEPHYR_BASE}/soc/cmcu/cmcuf/cmcus] contains the following files:

1. CMakeLists.txt            --      Adds all the C include and source files to the build and invokes the linker.
2. Kconfig                   --      This configuration file selects architecture specific symbols for the SoC.
3. Kconfig.defconfig         --      Typically used to define the stack size of basic threads.
4. Kconfig.soc               --      This configuration file defines two important symbols named SOC and SOC_FAMILY which is required by the build system.
                                     It also defined the SoC specific symbols SOC_CMCUSOC and SOC_SERIES_CMCUS. It also defines the number of IRQs supported
                                     by the SoC using the symbol NUM_IRQS. 
5. pinctrl_soc.h             --      This header file contains the pin control typedef pinctrl_soc_pin_t and utility macros for pin control. This header file
                                     references another header file ${ZEPHYR_BASE}/include/zephyr/dt-bindings/pinctrl/cmcu_pinctrl.h which contains
                                     the important macro CMCU_PINMUX() which is a macro used in the pin control DTSI file present in
                                     ${ZEPHYR_BASE}/../modules/hal/cmcu/dts/cmcu/cmcus/cmcu-pinctrl.dtsi
6. soc.c                     --      This is a C source file which contains a SoC initialization function named soc_early_init_hook() which is used to initialize
                                     the basic peripherals of the SoC (example: SyTick timer, ART accelerator etc.) 
7. soc.h                     --      This C header file links the SoC specific SDK to the Zephyr directory. For this demonstration, the SoC SDK files are defined under
                                     ${ZEPHYR_BASE}/../modules/hal/cmcu/cmcu_sdk.

[2] Create the SoC specific pin control device tree source include file & SDK support for the SoC

Each vendor provides HAL support for their SoC in ${ZEPHYR_BASE}/../modules/hal. For this demonstration, the HAL support is provided 
under ${ZEPHYR_BASE}/../modules/hal/cmcu. 

This directory contains the following files and directories:
1. CMakeLists.txt            --      Adds CMakeLists.txt in subdirectories to the build.
2. dts                       --      Contains the pin control device tree source include file.
3. module.yml                --      This YML file is required to let Zephyr know that this is an external module.
4. LICENSE                   --      Simple license file.
5. README.rst                --      HAL documentation.
6. cmcu_sdk                  --      This is the custom SoC SDK which contains a header file containing the registers of the SoC.

Once steps [1] and [2] are done goto ${ZEPHYR_BASE} are run the command shown in the image.

<img width="1477" height="87" alt="image" src="https://github.com/user-attachments/assets/2a3b359b-6bdf-4b49-98c4-21bcbba18585" />

The SoC must now be detectable.
 
[3] Develop the required DTS binding files for the peripherals of the SoC under ${ZEPHYR_BASE}/dts/bindings.
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

[4] Write the device tree source and device tree include files for the SoC

The DTS files for the SoC have to be created under ${ZEPHYR_BASE}/dts/arm/cmcu/cmcus/.
Two files named cmcus.dtsi and cmcusoc.dtsi were created, one for the SoC series and one for the SoC.
Two helper header files cmcu_clock.h and cmcu_reset.h ( present in ${ZEPHYR_BASE}/include/zephyr/dt-bindings/clock/cmcu_clock.h & 
${ZEPHYR_BASE}/include/zephyr/dt-bindings/clock/cmcu_reset.h ) are used. The two header files contain help macros CMCU_CLOCK and
CMCU_RESET. The compatible property for each node was updated to use the custom mcu device tree bindings.

cmcus.dtsi -- custom SoC series peripheral device tree.
cmcusoc.dtsi -- custom SoC specific device tree settings.
. 
[5] Create the board speific files under ${ZEPHYR_BOARD}/boards/cmcu/cmcu_board

The following files are created under the boards directory:

1.  board.cmake            --        The board.cmake file is used to configure the Zephyr runner settings which is used to flash
                                     the firmware into the MCU.
2.  board.yaml             --        Defines the SoC(s) used in the board, the vendor and the name of the board. When using the west build command,
                                     the name of the board given in this .yml file is used.
3.  cmcu_board.dts          --       This is the board DTS file. This file defines the basic pin configurations for the SoC, clock settings of the MCU etc.
                                     This DTS file includes the pin control DTSI provided by the vendor in ${ZEPHYR_BASE}/../modules/hal/cmcu/dts/cmcus/cmcu-pinctrl.dtsi
                                     and the DTSI in the zephyr DTS subdirectory ${ZEPHYR_BASE}/dts/arm/cmcu/cmcus/cmcus.dtsi.
4.  cmcu_board.yaml        --        This is a YAML file describing the type of device, supported peripherals, RAM and FLASH etc.
5.  cmcu_board_defconfig   --        Selects basic configuration setting to get the board working (such as serial peripheral, UART console etc.)
6.  Kconfig.cmcu_board     --        This file contains the configuration variable BOARD_<BOARD_NAME> (BOARD_CMCU_BOARD in this case) which, when selected, selects the SoC
                                      symbol SOC_<SOC> (SOC_CMCUSOC) which then configures the SoC related symbols.

Once steps [1] - [4] are done, run the following command as check if the created board is getting recogonised.

<img width="1477" height="87" alt="image" src="https://github.com/user-attachments/assets/3632a0aa-f529-4bac-84d2-c5970f951f44" />

[6] Understanding how drivers work in Zephyr

There are 2 basic requirements for a driver to be recogonized by the zephyr build system:
1. A driver DTS binding file defined under ${ZEPHYR_BASE}/dts/bindings/<driver_name>.
2. A C source file with "#define DT_DRV_COMPAT <compatible>" where <compatible> is the compatible name defined
   in the driver's device tree binding yaml file. The driver has to be placed under ${ZEPHYR_BASE}/drivers/<driver_name>/<driver_name>_<soc_name>.c
   For example for uart driver there should be a DTS binding yaml file ${ZEPHYR_BASE}/dts/bindings/cmcu/uart/cmcu,cmcu-uart.yaml. Then create a driver source file
   under ${ZEPHYR_BASE}/drivers/serial/uart_cmcu.c. The first line of the source file must be #define DT_DRV_COMPAT cmcu_cmcu_uart.

Each driver repository consists of a common Kconfig file and Kconfig file for each SoC and a CMakeLists.txt file.
The common Kconfig file sources all the induvidual Kconfig files to the build system. The soc specific Kconfig file defines a defines a SoC specific property which is enabled
only if the node having the required compatible property is defined and enabled in the DTS. The CMakeLists.txt file consists of conditional sources which sources the required
C source files only if the SoC specific driver property is selected.

For example: For the serial driver of the custom SoC, a seperate Kconfig file is created under ${ZEPHYR_BASE}/drivers/serial/Kconfig.cmcu which defines a configuration 
UART_CMCU. This symbol has a default value of 'y' and is selected only if the compatible node "cmcu,cmcu-uart" is defined and its status is set to "okay". Since UART_CMCU
is set to 'y', the custom SoC specific serial driver (uart_cmcu.c) C source code is selected and built.

For an SoC to get up and working, the following drivers have to be enabled:
1. clock-control         --    For setting up system clock and enabling/disabling different clock domains and clock gating.
                               Used by every other SoC driver.
3. pinctrl               --    To map a pin of the SoC to a particular peripheral.
4. gpio                  --    Essential for controlling the GPIO pins of the SoC. This driver is also used by pinctrl
                               to configure pin functionality.
5. interrupt-controller  --    Controls the interrupts in the SoC. Enables/disables interrupts.
6. serial                --    Required for getting debug logs from the SoC via UART interface.

 In this repository all the above drivers have been developed to perform basic functionalities to make the SoC up and running.

**How are drivers source code organized**

Each driver C source code file has a DT_DRV_COMPAT which is defined value as the compatible for the device/peripheral and it has to be defined as the first line in the C
source code. Failing to do so does not include the device driver in the build process.

Each device in Zephyr can be described by the device structure.

```
struct device{
    const void* config;
    void* data;
    const void* api;
    const char* name;
};

```

The device.config holds configuration data parsed from the device-tree during build time. The config struct for the device has to be defined in a seperate header file.
The device.data holds run-time configuration & state monitoring variables of the device. The device.api points to the struct containing callbacks for the driver API.
The driver API for the particular SoC is defined using DEVICE_API. In addition to driver initialization function, there can also be a power-management initialization
function for the SoC (which is NOT covered in this repository).

Each instance of the compatible driver is initialized using DEVICE_DT_INST_DEFINE which initializes an instance of the compatible driver. This macro takes the following
arguments:
1. Initialization function
2. Power management function
3. Driver data structure initialized during build time.
4. Driver configuration structure initialized during build time.
5. Stage of initialzation (PRE_KERNEL_1, PRE_KERNEL_2, POST_KERNEL)
6. Initialization priority.
7. Address of driver API.

Calling DEVICE_DT_INST_DEFINE initialzes a device structure during build time.

**Importance of device.api**

Consider the serial driver. The serial driver is included in the application code as ``` #define <zephyr/driver/uart.h> ```.

All functions defined in uart.h are inline functions which take a pointer to the device, reference the API structure and calls the required
API function using the function pointer.

Snippet from ${ZEPHYR_BASE}/include/zephyr/drivers/uart/uart_internal.h

```
static inline int z_impl_uart_poll_in(const struct device *dev, unsigned char *p_char)
{
	const struct uart_driver_api *api = (const struct uart_driver_api *)dev->api;

	if (api->poll_in == NULL) {
		return -ENOSYS;
	}

	return api->poll_in(dev, p_char);
}

```
  
**Critical concepts required to understand SoC driver code**
1. Device tree
2. Kconfig
3. Devicetree binding
4. Devicetree API
5. IRQ API

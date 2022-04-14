
- [Getting Started with the MAX78002EVKIT](#getting-started-with-the-max78000fthr)
  - [Schematic](#schematic)
  - [Developing with Eclipse](#developing-with-eclipse)
    - [Running Eclipse](#running-eclipse)
    - [Creating a I/O Peripheral Project](#creating-a-io-peripheral-project)
    - [Creating a CNN Project from an Example](#creating-a-cnn-project-from-an-example)
  - [Building the SDK Examples using MINGW on Windows](#building-the-sdk-examples-using-mingw-on-windows)
  - [Loading and Running Example Firmware](#loading-and-running-example-firmware)
  - [Debugging Applications with GDB](#debugging-applications-with-gdb)
  - [How to Unlock a MAX78000 That Can No Longer Be Programmed](#how-to-unlock-a-max78000-that-can-no-longer-be-programmed)
  - [Additional SDK Information](#additional-sdk-information)
  - [TFT display](#tft-display)
  - [ML-Focused Approach using Linux](#ml-focused-approach-using-linux)
  - [Installing the Developer Tools (Linux)](#installing-the-developer-tools-linux)
  - [Next Steps:  Loading and running the included **mnist** CNN example](#next-steps--loading-and-running-the-included-mnist-cnn-example)
    - [CNN Boost](#cnn-boost)
    - [Links to mnist and additional CNN examples](#links-to-mnist-and-additional-cnn-examples)
  - [Going beyond the included CNN examples - Advanced Topics](#going-beyond-the-included-cnn-examples---advanced-topics)

---

# Introduction

This quick-start document contains info on getting started with the MAX78002EVKIT evaluation platform.  It supplements the full MAX78002EVKIT datasheet with information on software development and quick tips for getting started.

# Schematic & BOM

The schematic and BOM can be found in the MAX78002EVKIT [Datasheet](TODO).  The EVKIT datasheet is the de-facto hardware reference for the MAX78002EVKIT, while this quick-start guide offers more insight into EVKIT operation and software development.

# First-time Firmware Updates

The firmware for the MAX32625PICO ("PICO") debug adapter should be updated before using the MAX78002EVKIT.  These updates contain bug fixes and improvements that are required to ensure proper operation of the kit.

## Updating the MAX32625PICO ("PICO") Debug Adapter Firmware

1. Locate the "max32625_max78000fthr_if_crc_v1.0.2.bin" file.  

    If you have cloned this repository, the file is in the "MAX78002_Evaluation_Kit/MAX32625PICO_files" folder.  
    
    Alternatively, it can be downloaded from [this](TODO) link.

2. Connect the included micro-USB cable to the "PICO" _without_ connecting the other side of the cable to your host PC yet.

    ![PICO Partially Connected](img/pico_partial_connected.jpg)

3. Press and hold the pushbutton on the top of the "PICO".

    ![PICO Pushbutton](img/pico_pushbutton.jpg)

4. _While holding down the pushbutton on the "PICO"_ connect the other side of the micro-USB cable to your host PC.  

    Keep the pushbutton held down until the LED on the "PICO" blinks and becomes solid.

    ![PICO Connected](img/pico_connected.jpg)

4. A "MAINTENANCE" drive should now appear on your file system.

    ![Maintenance Drive Image](img\MAINTENANCE.jpg)

    Note:  If a DAPLINK drive presents itself instead, retry the connection while holding the pushbutton down.  Holding the pushbutton while connecting the "PICO" will place it in MAINTENANCE mode, allowing its debugger firmware to be reprogrammed.

5. Drag and drop the "max32625_max78000fthr_if_crc_v1.0.2.bin" file onto the MAINTENANCE drive.  This will flash the "PICO" with the updated firmware.

    ![Drag and Drop](img\drag_and_drop.JPG)

    ![Flashing](img\pico_flashing.JPG)

6. Once the flashing is complete, the "PICO" will restart and present itself as a "DAPLINK" drive.

    ![Daplink Drive](img\DAPLINK.jpg)

7. Open the DAPLINK drive.

    ![Daplink Opened](img\DAPLINK_opened.jpg)

8. Open the "DETAILS.TXT" file and verify the contents match the "MAX32625PICO_files/DETAILS.TXT" file in this repository.  Specifically - the "Git SHA" field should match exactly.

    ![Details GIT SHA](img\DETAILS_Git_SHA.jpg)

9. Your "PICO" debugger is now ready to use with the MAX78002EVKIT.

# Powering & Connecting to the Evaluation Kit

## Power Supply
The MAX78002EVKIT requires an external 5V supply power supply.  A 5V wall adapter is supplied in the box.  To power the board, connect the 5V adapter to the "5V IN" barrel jack input J1.  The "5V" status LED (D2) should turn on.

TODO: Pic of J1

SW1 is the power switch for the board.  With 5VIN supplied, flip the switch to power the rest of the board.  The "1V1 CNN" (D4) and "3V3" (D3) status LEDs will turn on and can be used to verify the board is powered on successfully.

TODO:  pic of status LEDs.

## Connecting Debug Adapters

The MAX78002 is a dual-core microcontroller, with both an Arm Cortex M4 core and a RISC-V core.  As a result, the EVKIT comes with two debug adapters: 

A MAX32625PICO:
* For use with the Arm Cortex M4 core
* Connects to the "SWD" debug header (JH8)

![PICO SWD Connected](img/pico_swd_connected.jpg)

An Olimex ARM-USB-OCD-H + ARM-JTAG 20-10 adapter:
* For use with the RISC-V core
* Connects to the "RV JTAG" debug header (JH5)

(TODO: pic of Olimex)

Both debuggers should then connect back to the host PC with the included USB cables.

## Connecting Serial Ports

The MAX78002EVKIT exposes 3 serial ports via micro-USB connectors.

### "USB/UART" (CN2)

This serial port routes UART0 and UART1 from the MAX78002 to an [FT231XS](https://ftdichip.com/products/ft231xs/) UART to USB adapter.  This is the default serial port used by most examples.

To communicate over this port, connect a micro-USB cable from the host PC to CN2.  Most Operating Systems will auto-install the required drivers or already include them on the system, and the FT231XS chip will present itself as a Virtual COM Port (VCP).  

If the port does _not_ present itself, then manual driver installation is required.  See [this](https://ftdichip.com/drivers/vcp-drivers/) page from the official FTDI site.

Maxim's examples use the following serial communication settings by default:
* BAUD Rate:  115200
* Data: 8-bit
* Parity: None
* Stop bits: 1
* Flow control: None

Serial communication settings can be modified in the "MaximSDK/Libraries/Boards/MAX78002/EvKit_V1/Include/board.h" file.  Jumper settings should be checked when modifying the default settings, such as changing from UART0 to UART1.  Refer to the [MAX78002 Datasheet](TODO) for more details on jumper settings.

### "USB/AI" (CN3)

This serial port connects directly to the USB 2.0 Hi-Speed Controller peripheral on the MAX78002.  The serial port will not be functional unless the host firmware running on the MAX78002 sets it up.

### "USB/PWR MON (CN1)

This serial port connects to a secondary MAX32625 microcontroller that manages the power monitoring sub-circuit of the MAX78002EVKIT.  This port will present itself as a standard USB Serial Device.

TODO: Verify USB drivers.

## Connecting Camera Modules

### Pcam 5C

The included Pcam 5C is a CSI camera, and should be connected to the CSI connector J8.

1. Connect the CSI ribbon cable to the Pcam 5C.

    Looking at the bottom of the Pcam 5C board, the blue strip of the ribbon cable should be facing you.

    ![Pcam 5C Bottom](img/pcam_bottom.jpg)

2. Connect the CSI ribbon cable to the CSI connector J8 on the MAX78002EVKIT.

    Slide out the thin darker part of the connector to open it, then insert the ribbon cable.

    The blue strip of the ribbon cable should be facing up.

    Snap the thin darker part of the connector to close it and lock the cable in place.

    ![CSI Connector](img/csi_connector.jpg)

3. Remove the cap from the CSI camera module before using it.

    ![Pcam Top](img/pcam_top.jpg)

# First-time Operation 

The MAX78002EVKIT will come pre-flashed with a "Hello World" example program out of the box.  Powering on the 

# Software Development Quick-Start

## Setup

Embedded development for the MAX78002EVKIT is enabled by the Maxim Microcontrollers SDK ("MaximSDK").

The MaximSDK can be downloaded and installed from the following links:
* [Windows](https://www.maximintegrated.com/en/design/software-description.html/swpart=SFW0010820A)
* [Linux](https://www.maximintegrated.com/en/design/software-description.html/swpart=SFW0018720A)
* [MacOS](https://www.maximintegrated.com/en/design/software-description.html/swpart=SFW0018610A)

More detailed setup instructions for the MaximSDK can be found in the "Embedded Software Development Kit (SDK)" section of the ai8x-synthesis readme [here](https://github.com/MaximIntegratedAI/ai8x-synthesis/blob/develop/README.md#embedded-software-development-kit-sdk).

Follow the instructions above to install and set up the MaximSDK.

## Example Projects

Example projects for the MAX78002EVKIT can be found in the `MaximSDK/Examples/MAX78002` folder of the MaximSDK installation.

![Examples Folder](img/examples_folder.jpg)

Each example project contains a "README.md" file explaining what the example does.  "Out of the box" each example is set up for command-line development, Visual Studio Code, and Eclipse.  In general, the examples are organized into folder names that are based on the peripheral that it demonstrates.

## Debugger Limitations of the MAX78002

Before continuing into the development quick-starts below, it's important to note some limitations when debugging the MAX78002:

* The debugger can not be connected to the MAX78002 _while_ the device is in reset.
* The MAX78002 can not be debugged while the device is in Sleep, Low Power Mode, Micro Power Mode, Standby, Backup, or Shutdown mode.  

    These modes shut down the SWD clock.

    Examples that enter these modes have a 2-second delay in place at the beginning of code execution to give the debugger a window to connect.  For low-power development, it's also recommended to have the same delay in place.

These limitations may make the device difficult (or impossible) for the debugger to connect in certain states.  In such cases, the device can be recovered from the "locked out" firmware by erasing the application-space in the flash memory bank.  See ["How to Unlock a MAX78000 That Can No Longer Be Programmed"](#how-to-unlock-a-max78000-that-can-no-longer-be-programmed).

## Command-Line Development

Every MaximSDK example project contains a core "Makefile" that manages the build system.  This section demonstrate how to build these examples on the command line.  It also demonstrates how to flash and debug over the command line.

This section is presented first because all of the IDEs supported by the MaximSDK rely on a functional command-line toolchain.  Familiarity with the inner workings of the command-line can facilitate better experiences with the IDEs.  However, this is by no means required reading to use those IDEs.

### Building Examples

The MaximSDK build system is a managed build system built around GNU Make.  Every example project contains a `Makefile` that is used to build the source code.  The [GNU Make manual](https://www.gnu.org/software/make/manual/make.html) is a good document to bookmark when working with Makefiles.

1. First, copy the example project to an accessible directory outside of the SDK.  It's strongly recommended to keep the MaximSDK examples "clean" and unmodified in case they need to be referenced again later.

2. Launch your terminal.  On Windows, use the `MaximSDK/Tools/MSYS2/msys.bat` file to launch the MSYS2 terminal, which auto-configures the terminal for the MaximSDK toolchain.  On Linux and MacOS, you should have set up the toolchain for your terminal as part of the setup procedure.

3. "cd" into the location of the copied example project.

4. Run the following command to build the example:

    ```shell
    make -r -j 8 all
    ```

    * "-r" is an option that improves build speed.
    * "-j 8" enabled parallel execution of the build in 8 threads.  In general, this number should be double the number of cores on your machine.
    * "all" is the target for building "all" of the source code.

5. You'll see the source code for the project and the MaximSDK peripheral drivers being compiled on the terminal output.  Finally, the compiled source code will be linked into the build output file (`build/max78002.elf`).

6. The example project is now successfully built and ready to flash and debug.

### Flashing Examples

1. First, build the example project if you have not done so already (See the section above).

2. Connect the "PICO" debugger to the SWD debugger port (See [Connecting Debug Adapters](#connecting-debug-adapters)).

3. Launch your terminal.  On Windows, use the `MaximSDK/Tools/MSYS2/msys.bat` file to launch the MSYS2 terminal.

4. "cd" into the location of the copied example project.

5. Run the one of the following commands to flash the program with OpenOCD.

    * Flash and exit:

        ```shell
        openocd -s $MAXIM_PATH/Tools/OpenOCD/scripts -f interface/cmsis-dap.cfg -f target/max78002.cfg -c "program build/max78002.elf verify exit"
        ```

        Use this command if you just want to flash the program.  OpenOCD will exit on completion.

        Expected output:

        ```shell
        Open On-Chip Debugger 0.11.0+dev-g4cdaa275b (2022-03-02-09:57)
        Licensed under GNU GPL v2
        For bug reports, read
                http://openocd.org/doc/doxygen/bugs.html
        DEPRECATED! use 'adapter driver' not 'interface'
        Info : CMSIS-DAP: SWD  supported
        Info : CMSIS-DAP: Atomic commands supported
        Info : CMSIS-DAP: Test domain timer supported
        Info : CMSIS-DAP: FW Version = 0256
        Info : CMSIS-DAP: Serial# = 044417016af50c6500000000000000000000000097969906
        Info : CMSIS-DAP: Interface Initialised (SWD)
        Info : SWCLK/TCK = 1 SWDIO/TMS = 1 TDI = 0 TDO = 0 nTRST = 0 nRESET = 1
        Info : CMSIS-DAP: Interface ready
        Info : clock speed 2000 kHz
        Info : SWD DPIDR 0x2ba01477
        Info : max32xxx.cpu: Cortex-M4 r0p1 processor detected
        Info : max32xxx.cpu: target has 6 breakpoints, 4 watchpoints
        Info : starting gdb server for max32xxx.cpu on 3333
        Info : Listening on port 3333 for gdb connections
        Info : SWD DPIDR 0x2ba01477
        target halted due to debug-request, current mode: Thread
        xPSR: 0x81000000 pc: 0x0000fff4 msp: 0x20003ff0
        ** Programming Started **
        ** Programming Finished **
        ** Verify Started **
        ** Verified OK **
        shutdown command invoked
        ```

    * Flash and hold:

        ```shell
        openocd -s $MAXIM_PATH/Tools/OpenOCD/scripts -f interface/cmsis-dap.cfg -f target/max78002.cfg -c "program build/max78002.elf verify; init; reset halt"
        ```

        Use this if you want to also debug the program.  OpenOCD will flash the program, reset the MAX78002, and wait for a GDB debugger client connection.

        Expected output:

        ```shell
        Open On-Chip Debugger 0.11.0+dev-g4cdaa275b (2022-03-02-09:57)
        Licensed under GNU GPL v2
        For bug reports, read
                http://openocd.org/doc/doxygen/bugs.html
        DEPRECATED! use 'adapter driver' not 'interface'
        Info : CMSIS-DAP: SWD  supported
        Info : CMSIS-DAP: Atomic commands supported
        Info : CMSIS-DAP: Test domain timer supported
        Info : CMSIS-DAP: FW Version = 0256
        Info : CMSIS-DAP: Serial# = 044417016af50c6500000000000000000000000097969906
        Info : CMSIS-DAP: Interface Initialised (SWD)
        Info : SWCLK/TCK = 1 SWDIO/TMS = 1 TDI = 0 TDO = 0 nTRST = 0 nRESET = 1
        Info : CMSIS-DAP: Interface ready
        Info : clock speed 2000 kHz
        Info : SWD DPIDR 0x2ba01477
        Info : max32xxx.cpu: Cortex-M4 r0p1 processor detected
        Info : max32xxx.cpu: target has 6 breakpoints, 4 watchpoints
        Info : starting gdb server for max32xxx.cpu on 3333
        Info : Listening on port 3333 for gdb connections
        Info : SWD DPIDR 0x2ba01477
        target halted due to debug-request, current mode: Thread
        xPSR: 0x81000000 pc: 0x0000fff4 msp: 0x20003ff0
        ** Programming Started **
        ** Programming Finished **
        ** Verify Started **
        ** Verified OK **
        Info : Listening on port 6666 for tcl connections
        Info : Listening on port 4444 for telnet connections <-- Note:  OpenOCD is now waiting for a GDB client
        ```

### Debugging Examples

1. Flash the example program using the "Flash and hold" command above if you have not done so already.

2. Launch a _new_ terminal.  On Windows, use the `MaximSDK/Tools/MSYS2/msys.bat` file to launch a new MSYS2 terminal.

3. "cd" into the location of the copied example project.

4. Run the following command to launch a GDB client.

    ```shell
    arm-none-eabi-gdb --se=build/max78002.elf
    ```

    * "--se" sets the symbol and executable file to the compiled build output.

    Expected output:

    ```shell
    GNU gdb (GNU Arm Embedded Toolchain 10.3-2021.10) 10.2.90.20210621-git
    Copyright (C) 2021 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.
    Type "show copying" and "show warranty" for details.
    This GDB was configured as "--host=i686-w64-mingw32 --target=arm-none-eabi".
    Type "show configuration" for configuration details.
    For bug reporting instructions, please see:
    <https://www.gnu.org/software/gdb/bugs/>.
    Find the GDB manual and other documentation resources online at:
        <http://www.gnu.org/software/gdb/documentation/>.

    For help, type "help".
    Type "apropos word" to search for commands related to "word"...
    Reading symbols from build/max78002.elf...
    (gdb) 
    ```

9. Next, connect to the OpenOCD server that's running in the other terminal window with the following command.

    ```shell
    target extended-remote localhost:3333
    ```

    Expected output:

    ```shell
    Remote debugging using localhost:3333
    0x0000fff4 in ?? () <-- Note: ?? may be present at this stage, which is OK.
    ```

10. (Optional) Reset the MAX78002.  

    ```shell
    monitor reset halt
    ```

    If you have just come from running the "Flash and hold" command, the MAX78002 should already be reset and halted.
    However, it's always good practice to reset and halt the micro core before debugging.

    Expected output:

    ```shell
    SWD DPIDR 0x2ba01477
    target halted due to debug-request, current mode: Thread
    xPSR: 0x81000000 pc: 0x0000fff4 msp: 0x20003ff0
    ```

11. Set a breakpoint on main.

    ```shell
    b main
    ```

    Expected output:
    
    ```shell
    Breakpoint 1 at 0x10000224: file main.c, line 62.
    Note: automatically using hardware breakpoints for read-only addresses.
    ```

12. Continue the debugger.

    ```shell
    continue
    ```

    Expected output (for the Hello World example):

    ```shell
    Continuing.

    Breakpoint 1, main () at main.c:62
    62          printf("Hello World!\n");
    ```

13. (Optional) Continue debugging.

    At this stage, the debugger is connected properly and the program is halted on the `main` function.  
    You can continue debugging freely from here. 
    See [Common GDB Commands] below for a quick-reference

14. Quit GDB.

    ```shell
    quit
    ```

    Expected output:

    ```shell
    A debugging session is active.

        Inferior 1 [Remote target] will be detached.

    Quit anyway? (y or n) [answered Y; input not from terminal]
    Detaching from program: C:\Users\Jake.Carter\codespace\Hello_World\build\max78002.elf, Remote target
    [Inferior 1 (Remote target) detached]
    ```

15. Quit OpenOCD.  In the terminal window running OpenOCD, press "CTRL + C" to issue the shutdown command.

#### Common GDB Commands

| **Command**                    | **Short Command** | **Description**                                              |
| ------------------------------ | ----------------- | ------------------------------------------------------------ |
| monitor halt                   |                   | Halt the microcontroller.                                    |
| monitor reset halt             |                   | Reset the microcontroller and immediately halt.              |
| monitor max32xxx mass\_erase 0 |                   | Mass erase the flash.                                        |
| file \<filename\>              |                   | Set the program file to debug                                |
| load                           |                   | Flash the current program file                               |
| continue                       | c                 | Continue execution.                                          |
| break \<arg\>                  | b \<arg\>         | Set a breakpoint. Argument can be function\_name, file:line\_number, or \*address. |
| print \<variable\>             | p                 | Print the value of a variable. Variable must be in current scope. |
| backtrace                      | bt                | Print contents of the stack frame.                           |
| step                           | s                 | Execute the next instruction.                                |
| next                           | n                 | Execute the next line of code.                               |
| finish                         | f                 | Continue to the end of the current function.                 |
| info reg                       |                   | Print the values of the ARM registers.                       |
| help                           |                   | Print descriptions for available commands                    |
| help \<cmd\>                   |                   | Print description for given command.                         |
| quit                           | q                 | Quit the GDB client                                          |


## Developing with Visual Studio Code

As of 4/11/2022 the MaximSDK now comes with Visual Studio Code ("VS Code") support via the [VSCode-Maxim](https://github.com/MaximIntegratedTechSupport/VSCode-Maxim) project.

All MAX78002 example projects come "out of the box" with the latest VSCode-Maxim project files and documentation.  This can be found in the `.vscode` folder inside each example.  The example projects all come pre-configured for the MAX78002EVKIT by default.  However - some basic one-time setup is required for Visual Studio Code.  See the Setup section below.

### Setup

Follow the [installation instructions](https://github.com/MaximIntegratedTechSupport/VSCode-Maxim#installation) from the VSCode-Maxim readme.  This is a required step to use the VSCode-Maxim project files.

### Opening Example Projects

1. First, copy the example project from the "Examples/MAX78002" folder in the MaximSDK installation to an accessible directory outside of the SDK.  It's strongly recommended to keep the MaximSDK examples "clean" and unmodified in case they need to be referenced again later.

2. Launch Visal Studio Code.  You should have already configured VS Code for use with the MaximSDK as part of the setup procedure.  If not, follow the Setup section above before continuing.

3. Navigate to "File -> Open Folder..."

    ![Open Folder](https://raw.githubusercontent.com/MaximIntegratedTechSupport/VSCode-Maxim/main/img/file_openfolder.JPG)

4. The project should now be opened in Visual Studio Code.  For example:

    ![VS Code Opened](img/VSCode_helloworld.jpg)

5. Documentation (readme.md) and settings files can be found in the ".vscode" sub-folder of the example project.

    ![VS Code Folder](img/VSCode_folder.jpg)

### Usage

Detailed usage instructions can be found in the ["Usage"](https://github.com/MaximIntegratedTechSupport/VSCode-Maxim#usage) section of the VSCode-Maxim readme.

Additionally, a full [User Guide](https://github.com/MaximIntegratedTechSupport/VSCode-Maxim/blob/main/userguide.md) is available.


## Developing with Eclipse

### Setup

The only setup required to use Eclipse is to ensure that the "Eclipse" component is selected during the MaximSDK installation.  If the MaximSDK is already installed, the component can be added by launching the "MaintenanceTool" application in the root directory of the MaximSDK and selecting "Add or the remove components".

### Running Eclipse

If you are using Microsoft Windows, you must launch Eclipse from the _Windows Start Menu_ or the "MaximSDK/Tools/Eclipse/cdt/eclipse.bat" file. This eclipse.bat file calls the "setenv.bat" script in the root directory of the SDK before launching Eclipse, which makes the MaximSDK toolchain accessible.  Eclipse will fail to find the MaximSDK toolchain binaries unless it's launched with this method.

![Eclipse Link in Start Menu](img/eclipse_start.jpg)

### Importing an Example

1. Launch Eclipse.

2. Use "File -> Import" to open the import wizard.

3. Select "General -> Existing Projects into Workspace" and hit "Next".

    ![Import Menu](img/import.png)

4. "Select root directory" and hit the "Browse" button.

    ![Browse Button](img/eclipse_browse.jpg)

5. Browse to the "Examples/MAX78002" folder in the MaximSDK installation, and then into the example project you would like to import.

    For example, the "Hello_World" Project:

    ![Import Hello World](img/eclipse_helloworld_import.jpg)

    or the Keyword-Spotting ("kws20_demo") CNN project.  CNN projects are found in the "CNN" subfolder.

    ![Import KWS20](img/eclipse_import_kws.jpg)

6. Ensure that "Copy projects into workspace" is always selected when you import.  This will ensure that the MaximSDK's copy of the example remains "clean" and unmodified.

    ![Copy Projects](img/eclipse_copyprojects.jpg)

7.  Select "Finish" to import the project.

8. It should now show up in the Project Explorer.

    ![Project Explorer](img/eclipse_project_explorer.jpg)

9. The Eclipse projects files are configured for the MAX78002EVKIT by default.  This can be verified by opening the project properties and navigating to "C/C++ Build -> Environment"

    ![Project Properties](img/eclipse_project_properties.jpg)

    * "BOARD" should be set to "EvKit_V1"
    * "TARGET" should be set to "MAX78002"

10. The example is now successfully imported.  From here, you may edit, compile, and debug. 


### Creating a New Project

1. Launch Eclipse

2. Ensure that the Eclipse is set to the C/C++ perspective in the top right corner.  Otherwise, the new project wizard will not show up.

    ![Eclipse Perspective](img/eclipse_perspective.jpg)

3. Navigate to "File -> New -> Maxim Microcontrollers".

    ![New Project](img/eclipse_new_project.jpg)

4. Enter the project name and hit "Next".

    ![Project Name](img/eclipse_project_name.jpg)

5. Enter the following settings, optionally changing the "Select example type" option to change the example that the new project will copy.

    ![Project Configuration](img/eclipse_project_configuration.jpg)

6. Select "Finish" to create the new project.  From here, you may edit, compile, and debug.

## How to Unlock a MAX78000 That Can No Longer Be Programmed

Some [debugger limitations](#debugger-limitations-of-the-max78002) may make the device difficult to connect to.  In such cases, the device can be recovered from the "locked out" firmware by erasing the application-space in the flash memory bank.

Before following the procedure below, ensure that you have updated the "PICO" debugger firmware to the latest version.  See ["Updating the MAX32625PICO ("PICO") Debug Adapter Firmware"](#updating-the-max32625pico-"pico"-debug-adapter-firmware)

1. Connect the "PICO" debugger to the MAX78002 EVKIT's SWD debugger port.  See ["Connecting Debug Adapters"](#connecting-debug-adapters) for more detailed instructions on this step.

2. Connect the "PICO" debugger to the host PC with the included micro-USB cable.

3. Open the "PICO"'s DAPLINK drive on the host PC.

    ![DAPLINK Drive](img/DAPLINK.jpg)

4. Open the "DETAILS.TXT" inside of the DAPLINK drive in a text editor.

    ![DETAILS.TXT](img/DAPLINK_opened.jpg)

5. Verify that the "Automation allowed" field is set to 1.

    ![Automation Allowed](img/DETAILS_automation_allowed.jpg)

    If this field is _not_ set to 1, follow the procedure below:

    1. Press and hold the pushbutton on top of the "PICO".

        ![PICO Pushbutton](img/pico_pushbutton.jpg)

    2. _While holding the pushbutton_, drag and drop the "MAX32625PICO_files/auto_on.cfg" file onto the DAPLINK drive.  This file can also be downloaded directly [here](TODO).

        ![auto_on.cfg](img/auto_on.cfg.jpg)

    3. Continue holding the pushbutton until the file is finished transferring over, then release it.

    4. The "PICO" should power cycle, and the DAPLINK drive should re-appear with "Automation allowed" set to 1.

6. Power on the MAX78002EVKIT (if it isn't already).

7. Drag and drop the "MAX32625PICO_files/erase.act" file onto the "PICO's" DAPLINK drive.

    ![Erase.act](img/erase.act.jpg)

8. The "PICO" debugger will mass erase the MAX78002's application-space flash bank, which will completely wipe any application firmware that is programmed on the device.

9. Power cycle the MAX78002EVKIT.  The MAX78002 is now blank and ready to re-program.  The debugger should have no issues connecting to the device in this blank state.

Note:  In order to avoid the locked out state to begin with, it is recommended that during code development, a delay be placed at the beginning of user code in order to give the debug adapter an opportunity to communicate with or halt the processor.  A delay of 2 seconds is ideal so that the debugger can be attached manually.

## Additional SDK Information

The examples are separated by device type. The SDK on GitHub currently only includes the MAX78000. Therefore, the examples will be located in the Examples/MAX78000 folder. For each example, you will find the following files:

- makefile -- This file contains the rules used to build the application with the "make" command. The binaries for each project can be removed with the "make clean" command. Use "make distclean" to remove the binaries for each project and any libraries the project depends on.

- main.c -- This source file contains the entry point for the application.

- \*.c -- These files contain additional source code required by the example if necessary. Many of the examples reside entirely in the main.c file and will not have additional .c files.

- \*.launch, .cproject, and .project -- These files are the project files used in the Eclipse environment. They can be ignored when working with OpenOCD and GDB from the command line. (Note a few examples do not have Eclipse project files yet.)  For more information on using Eclipse, see ["Getting Started with Eclipse"](https://pdfserv.maximintegrated.com/en/an/TUT6245.pdf)

The SDK provides an API for working with the device's components. To use the API, you will need to include the header (\*.h) files in your source code. The API header files for the MAX78000 reside in Libraries/PeriphDrivers/Include/MAX78000/. For convenience, you can include the "mxc.h" file in your source. This file includes the headers for all the supported peripheral libraries. Documentation for the functions contained in the API can be found at Libraries/PeriphDrivers/Documentation/MAX78000/index.html.

# Machine Learning Development

This quick 



## Going beyond the included CNN examples - Advanced Topics

- [AI8X Model Training and Quantization](https://github.com/MaximIntegratedAI/ai8x-synthesis/blob/master/README.md)
- [AI8X Network Loader and RTL Simulation Generator](https://github.com/MaximIntegratedAI/ai8x-synthesis/blob/master/README.md)
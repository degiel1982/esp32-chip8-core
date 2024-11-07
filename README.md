# CHIP-8 Emulator for ESP32

## Overview

This project presents a comprehensive implementation of a CHIP-8 emulator explicitly tailored for the ESP32 microcontroller. By leveraging the ESP32's computational capabilities, this emulator recreates the CHIP-8 environment, enabling the execution of CHIP-8 programs and games as they were originally intended. The emulator encompasses memory management, input handling, CPU instruction processing, and graphical rendering, thereby offering a faithful emulation of the original CHIP-8 system. The utilization of hardware timers guarantees precise synchronization for CPU and GPU operations, thus maintaining the temporal fidelity required for accurate emulation.

The CHIP-8, originally developed in the 1970s, was an interpreted language designed for simple games on 8-bit microcomputers. Due to its relatively small instruction set and straightforward architecture, CHIP-8 has remained an enduring favorite for emulator developers. This project utilizes the ESP32 microcontroller to emulate the CHIP-8 system, allowing for an efficient reproduction of classic gameplay. The ESP32's dual-core processor and hardware timer capabilities make it an ideal platform for high-fidelity emulation. This project serves not only as a nostalgic venture for retro gaming enthusiasts but also as an educational tool for embedded systems and emulator development.

The CHIP-8, originally developed in the 1970s, was an interpreted language designed for simple games on 8-bit microcomputers. Due to its relatively small instruction set and straightforward architecture, CHIP-8 has remained an enduring favorite for emulator developers. This project utilizes the ESP32 microcontroller to emulate the CHIP-8 system, allowing for an efficient reproduction of classic gameplay. The ESP32's dual-core processor and hardware timer capabilities make it an ideal platform for high-fidelity emulation. This project serves not only as a nostalgic venture for retro gaming enthusiasts but also as an educational tool for embedded systems and emulator development.

## Features

- **Comprehensive CHIP-8 Emulation**: The emulator provides full CPU, GPU, memory, and input emulation, covering all 35 original CHIP-8 instructions. This ensures the accurate execution of CHIP-8 programs, maintaining compatibility with a wide range of ROMs.
- **Hardware Timer Integration**: The emulator employs ESP32 hardware timers to manage precise CPU and GPU cycle timing, facilitating smooth and consistent emulation. This use of hardware-level timing ensures the fidelity of program execution in comparison to the original CHIP-8 timing requirements.
- **Singleton Pattern for Class Implementation**: The core class is structured as a singleton, ensuring only one instance of the emulator operates at any given time. This design paradigm prevents state inconsistencies, which are critical for ensuring deterministic emulator behavior.
- **Extensive Documentation for Extendibility**: Each class, method, and functional component is thoroughly documented, providing a rich resource for developers interested in extending the emulator’s capabilities or adapting it to new use cases.
- **Optimized Display Buffer and Dirty Flags Mechanism**: The emulator manages a 64x32 monochrome display using a display buffer coupled with a `dirty_flags` array. This architecture optimizes rendering by only redrawing areas of the screen that have changed, thereby minimizing computational overhead and improving performance. The `dirty_flags` array keeps track of which 8-pixel columns in each row have been modified since the last frame. For example, if a sprite is drawn that modifies pixels in the first and third columns of a particular row, the corresponding `dirty_flags` entries for those columns are set. During the next screen update, only these modified columns are redrawn, significantly reducing the number of operations needed for each frame.

## Table of Contents
- [Getting Started](#getting-started)
- [Dependencies](#dependencies)
- [Class Overview](#class-overview)
  - [Public Methods](#public-methods)
- [Emulator Features](#emulator-features)
- [Building and Flashing](#building-and-flashing)
- [Testing the Emulator](#testing-the-emulator)
- [License](#license)

## Getting Started

To begin using this CHIP-8 emulator, you will need an ESP32 development board. The development environment can be set up using either the Arduino IDE, equipped with the ESP32 board package, or PlatformIO for enhanced flexibility in building and flashing firmware. The ESP32 board offers an excellent balance of performance and cost, making it ideal for embedded applications such as this.

To get started, clone the repository by running the following command in your terminal:

```sh
git clone https://github.com/your-repository-url.git
```

After cloning, open the project in your preferred integrated development environment (IDE), such as Arduino IDE or PlatformIO. Before compiling, make sure that all required dependencies are installed and correctly configured, as this will streamline the process of uploading the project to your ESP32 board.

### Dependencies

- **Arduino.h**: This header is required for the interaction between the emulator and the ESP32 hardware. It provides the foundational functions for GPIO, timing, and other hardware-specific operations. It is needed for the Arduino IDE and ESP32 environment.
- **flag_manager.h**: The `flag_manager` class is employed to manage emulator state flags with atomic operations, ensuring thread safety. This is particularly crucial when interacting with the ESP32 hardware timers, as it avoids race conditions that could compromise the emulator's functionality.

## Class Overview

### chip8_core

The `chip8_core` class represents the core of the CHIP-8 emulation system, handling the entirety of the emulation process. This includes executing CPU instructions, updating the display buffer, managing the emulated memory, and tracking the state of input keys. To prevent state inconsistencies, this class follows the singleton design pattern, ensuring that only one instance of `chip8_core` is active at any given moment. This approach is essential to maintain deterministic emulator behavior, particularly with respect to memory and display states.

#### Public Methods

1. **getInstance()**
   - **Description**: This method provides access to the singleton instance of the `chip8_core` class. By restricting instantiation to a single object, `getInstance()` ensures consistent state management across the entire emulator lifecycle. The singleton pattern prevents conflicts in emulator state management, such as issues with memory access or display rendering that could arise from multiple instances trying to modify the same resources concurrently.
   - **Usage**:
     ```cpp
     chip8_core& emulator = chip8_core::getInstance();
     ```

2. **get_display_buffer()**
   - **Description**: This method returns a pointer to the display buffer, which stores the pixel states for the 64x32 monochrome screen. The display buffer is fundamental for graphical output, as it represents the visual state of the emulated system, which is rendered in each emulation loop.
   - **Usage**:
     ```cpp
     uint8_t* display = emulator.get_display_buffer();
     ```

3. **get_dirty_flags()**
   - **Description**: This method returns the `dirty_flags` array, which is used to track modifications in the display buffer. By updating only the portions of the display that have changed, the emulator achieves efficient rendering performance, particularly on a resource-constrained platform like the ESP32.
   - **Usage**:
     ```cpp
     uint8_t (*flags)[8] = emulator.get_dirty_flags();
     ```

4. **start()**
   - **Description**: Initializes and starts the emulator. If a ROM has been loaded and the emulator is not already running, this method sets up memory, registers, and hardware timers (if enabled). It is a crucial entry point for beginning emulation and ensuring the emulator is in a correct initial state.
   - **Usage**:
     ```cpp
     if (emulator.start()) {
         // Emulator started successfully
     }
     ```

5. **stop()**
   - **Description**: Stops the emulator, resetting its internal state and disabling any active hardware timers. This method is essential for halting emulation safely, preventing any residual background operations that could affect subsequent executions or system stability.
   - **Usage**:
     ```cpp
     if (emulator.stop()) {
         // Emulator stopped successfully
     }
     ```

6. **is_running()**
   - **Description**: Determines whether the emulator is currently running. This is particularly useful for control flow in applications that may need to pause or stop the emulator based on specific conditions.
   - **Usage**:
     ```cpp
     bool running = emulator.is_running();
     ```

7. **loop()**
   - **Description**: This method represents the main emulation loop. It should be called repeatedly to handle both CPU instruction execution and GPU updates. The `loop()` method ensures that the emulation timing is maintained, including instruction execution and display refresh rates.
   - **Usage**:
     ```cpp
     emulator.loop();
     ```

8. **sound()**
   - **Description**: Checks whether the sound timer is active. The CHIP-8 sound timer is a simple mechanism that generates a beep sound when it is non-zero. This method can be used to determine whether a sound should be played, thereby preserving the auditory feedback expected in CHIP-8 programs.
   - **Usage**:
     ```cpp
     if (emulator.sound()) {
         // Play a beep sound
     }
     ```

9. **reset_draw()**
   - **Description**: Resets the draw flag after a frame has been rendered, ensuring that the display only refreshes when necessary. This optimization is important to minimize the workload on the ESP32 by avoiding redundant drawing operations.
   - **Usage**:
     ```cpp
     emulator.reset_draw();
     ```

10. **need_to_draw()**
    - **Description**: Checks if the draw flag is set, indicating that a new frame needs to be rendered. This method works in conjunction with `reset_draw()` to manage efficient screen updates.
    - **Usage**:
      ```cpp
      if (emulator.need_to_draw()) {
          // Draw the new frame
      }
      ```

11. **load_rom(const uint8_t* rom, const size_t rom_size)**
    - **Description**: Loads a CHIP-8 ROM into memory starting at address 0x200, following the original CHIP-8 specification. This method also initializes key parts of the memory, such as loading the default fontset, ensuring that the emulator has all necessary resources to run the loaded program.
    - **Usage**:
      ```cpp
      const uint8_t rom_data[] = { /* ROM data */ };
      emulator.load_rom(rom_data, sizeof(rom_data));
      ```

12. **set_key_state(uint8_t key, bool is_pressed)**
    - **Description**: Updates the state of a specific key. The CHIP-8 system uses a 16-key hexadecimal keypad, and this method allows the emulator to track key presses and releases, thus enabling user interaction.
    - **Usage**:
      ```cpp
      emulator.set_key_state(0x1, true); // Key '1' pressed
      ```

13. **is_key_pressed(uint8_t key)**
    - **Description**: Checks if a specific key is currently pressed. This functionality is fundamental for interpreting user input in real-time while running CHIP-8 programs.
    - **Usage**:
      ```cpp
      bool pressed = emulator.is_key_pressed(0x1);
      ```

14. **enable_hardware_timers()**
    - **Description**: Enables the hardware timers used to manage CPU and GPU cycle timing. Using hardware timers improves the accuracy of the emulation, ensuring that the emulator operates in sync with the original CHIP-8 timing specifications.
    - **Usage**:
      ```cpp
      emulator.enable_hardware_timers();
      ```

15. **is_init_and_ready()**
    - **Description**: Checks if the emulator has been properly initialized and is ready to run. This method verifies that all necessary setup steps, such as memory initialization and ROM loading, have been completed. It differs from `is_running()` as it only checks readiness for emulation and not the actual running state of the emulator.
    - **Description**: Checks if the emulator has been properly initialized and is ready to run. This method is helpful for verifying that all setup steps have been completed before starting the emulation process.
    - **Usage**:
      ```cpp
      if (emulator.is_init_and_ready()) {
          // Emulator is ready to start
      }
      ```

## Emulator Features

The CHIP-8 emulator is equipped with several critical features to replicate the functionality of the original CHIP-8 system accurately. These features include:

- **Memory Management**: The emulator mirrors the original 4KB memory structure used in CHIP-8 systems. The first 512 bytes are typically reserved for system use, including the default font set, while program data is loaded starting at address 0x200. This structured memory allocation is essential for compatibility with legacy CHIP-8 ROMs.
- **Timers**: Two key timers are implemented: a delay timer and a sound timer, both decrementing at a frequency of 60Hz. The precise management of these timers is crucial to maintaining game logic and auditory cues as per the original CHIP-8 specification. By employing the ESP32 hardware timers, the emulator ensures that the timing behavior is both accurate and reliable.
- **Input Handling**: CHIP-8 programs utilize a hexadecimal keypad, and the emulator implements a key state tracking mechanism to determine which keys are pressed at any given time. This system allows for effective user interaction and the correct interpretation of game inputs.
- **Graphics Rendering**: The emulator replicates the CHIP-8's simple monochrome 64x32 display. It employs a display buffer and a `dirty_flags` mechanism to optimize screen rendering by updating only modified sections of the display. This approach minimizes computational load, making the emulator suitable for the resource constraints of the ESP32.

## Building and Flashing

1. **Install the ESP32 Board Package**: Ensure that the ESP32 board package is installed in your development environment, either Arduino IDE or PlatformIO. This package is necessary to compile the firmware and flash it to your ESP32 development board.

2. **Configure the Project**: Open the project in your chosen IDE. Set the board type to match your specific ESP32 model. Accurate board settings ensure that the correct pin mappings and system configurations are used during the build process.

3. **Compile and Upload**: Once configured, compile the project and upload it to the ESP32. Connect your ESP32 to your PC via USB for flashing. The process might take several minutes, depending on the IDE and the size of the firmware.

4. **Running the Emulator**: Upon successful flashing, you can use the `start()` method to initiate the emulator, provided a valid ROM is already loaded. The emulator will begin executing the loaded instructions, handling graphics, input, and timing in a manner consistent with the original CHIP-8 environment.

## Testing the Emulator

To validate the emulator's functionality, you can use a variety of freely available CHIP-8 ROMs, including classic games like Pong, Tetris, and Space Invaders. Load these ROMs using the `load_rom()` method, and initiate the emulation with `start()`. The display buffer outputs can be connected to a screen for visual feedback, while the `set_key_state()` method can be used to simulate user input.

Reliable CHIP-8 ROMs can be found on various retro gaming websites and archives, but users should be mindful of potential copyright issues when downloading and using these ROMs. Only use ROMs that are freely distributed or that you have permission to use.

For a thorough evaluation, compare the emulator's behavior with that of a known reference CHIP-8 interpreter. Such comparison should include analyzing the timing precision, graphical output, and responsiveness to input to ensure accurate emulation.

## License

This project is licensed under the MIT License. You are free to modify and distribute the emulator in accordance with the terms of the license. The MIT License allows for personal and commercial use, as long as the original copyright notice is preserved in all copies or substantial portions of the software.


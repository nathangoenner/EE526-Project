# TensorFlow Lite Micro on STM32F4DISCOVERY

This assignment will teach you how to train, quantize, and deploy a deep learning neural network to a STM32F407VG microcontroller

## Prerequisites

### Hardware

- STM32F4DISCOVERY development kit (https://www.digikey.com/en/products/detail/stmicroelectronics/STM32F407G-DISC1/5824404)
- USB-A to Mini USB type B cable
- Host machine running linux (or a virtual machine)

The STM32F4DISCOVERY dev kit houses a STM32F407VG microcontroller, which is a Cortex-M4 device suitable to running inferences with the TensorFlow Micro Lite library.

### Software

This assignment is designed for use on a Linux system with make, python3.7, gcc, and gdb. You should also install Visual Studio Code with the Platformio extension to flash and debug the microcontroller. Ubuntu is a popular choice for Linux, and can be installed in a virtual machine. However, extra care must be taken to access the USB ports of the machine through virtualization software such as VirtualBox. This will be required to flash and debug the physical target microcontroller.

## Step 1: Train the model

We will begin by training a very simple model as a starting point. The model will be able to take an input, x, and infer the value of the sine of the input, y, using a three layer neural network.

Follow along with the attached create_sine_model.ipynb notebook in Google Collab to generate a C file containing the model.

As part of this process, we will generate a noised sine wave, split samples into train/test/validate, and train our network. We will then convert it to a TensorFlow Lite Micro model, which has less functionality than standard TensorFlow models but is also small and suitable for use on a microcontroller, using less than 20 kB of memory.

Additionally, we will quantize the model to further reduce its memory usage and increase its performance on the Cortex-M4 ARM processor.

## Step 2: Add the new model to the firmware project

The file located at src/sine_model.cpp contains the model in the form of an array of 8-bit unsigned integers. Copy the output of the training notebook and replace the array and length variables in sine_model.cpp.

## Step 3: Build the firmware for the STM32F407VG target

This project is already set up to target the STM32F407VG with Platformio. Review the code in src/main.cpp to get an idea for what is going on with this project.

First, the tensorflow headers are included to bring the library symbols into scope.

On line 71, pointers are initialized that will be later used to interact with our TensorFlow model. A couple constants here define the rate of change of our `x` signal - we will test 20 samples between 0 and 2pi.

In the main function, at line 142, we begin to initialize our TensorFlow model. First, define some error reporting functionality, then load the model from the file modified in the previous step. Then, define the interpreter, allocate tensors, and define the model's input and output vectors. Our input/output will only be of length 1, the size of our last neural network layer, to predict the value of the sine of a single input value.

Finally, there is a `while (1)` loop. When writing firmware for microcontrollers, we don't want to exit our main function because that is essentially equivalent of turning off the microcontroller. So instead, we stay in the main while(1) loop indefinitely. Within this loop we will run inferences on our defined input of 20 input samples between 0 and 2pi.

## Step 4: Flash the microcontroller and enter debug mode for validation

Similar to the previous step, this project is already configured via .vscode/launch.json to attach to a connected microcontroller, flash it, and enter debug mode.

If a UART to USB converter is available, that could be used to access the print statements from the program and verify that the model's output is expected. However, it is also possible to verify through debugging with the on-board ST-Link debugger.

Connect the STM32F4DISCOVERY board to the host machine via USB. Be sure to use the Mini USB type B port on the board, not the Micro USB port near the audio jack. This Mini USB port connects to the onboard ST-Link debugger, allowing us to flash and debug the program.

Then, within Visual Studio Code, press F5. The project should compile and load onto the board. You should get a debugging panel allowing control of instructions and the program should stop at its entry point, awaiting further input.

Place a breakpoint on src/main.ccp at line 218, on the printf statement, by clicking just to the left of the line number in the editor. Then, press the "Continue" button on the debug panel. The program should quickly advance to the breakpoint.

In the 'Debug' view of Visual Studio Code (click the triangle with an insect icon on the left side of the screen), there should be a 'WATCH' panel. Here, you can enter expressions to be queried by GDB from the microcontroller's memory. Add an expression by clicking on the plus sign and type the name of our input variable: `x_val`. Also add a watch point for our output variable, `y_val`. Verify that `y_val` is approximately equal to `sine(x_val)`. Click the 'Continue' button a few more times, watching `x_val` and `y_val` update on each iteration.

Note that after approximately 10 iterations the program will hang in the WWDG_IRQHandler indefinitely. It's possible there is a bug in the TensorFlow library that causes this behavior, due to using a somewhat out of date version of TensorFlow. It would be preferable to use the latest stable release of TensorFlow, however I was not able to get the newer library to compile and link properly while producing a valid output.

## Conclusion

This assignment has outlined the general process behind training, converting, compiling, and flashing an application with machine learning on embedded systems, targeting a Cortex-M4 ARM microcontroller. These devices are low cost and low power, coming with additional restrictions on memory and processing power as a result. Due to this, an appropriately small model and deep learning framework is required, much less powerful when compared to the high power and high compute multi-GPU systems often associated with the machine learning industry.


## References

1. TensorFlow's guide to TensorFlow Lite Micro on microcontrollers: https://www.tensorflow.org/lite/microcontrollers
2. Fahad Mirza's project with TensorFlow on a similar board, the STM32F746DISCOVERY: https://mirzafahad.github.io/2020-06-26-tflite-stm32-part3/
3. Fahad Mirza's github repository referenced for the inference code: https://github.com/mirzafahad/stm32_tflite_sine

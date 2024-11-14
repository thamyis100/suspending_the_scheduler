
---

# STM32 FreeRTOS Scheduler Suspension Demo

This project demonstrates how to handle task scheduling and shared resource contention in an STM32 microcontroller environment using FreeRTOS. The primary focus is on suspending the scheduler to manage access to shared resources between tasks, ensuring atomic access while preventing preemption.

## Overview

In embedded systems with real-time constraints, handling shared resources between tasks is essential to avoid contention issues. This project implements a scenario where two tasks, `GreenLed` and `RedLed`, attempt to access a shared resource simultaneously. The resource access is managed with scheduler suspension to prevent contention and ensure atomic access.

The tasks and resource-access logic are set up as follows:

- **Red LED Task**: Runs at a higher priority and accesses the shared resource frequently.
- **Green LED Task**: Runs at a lower priority, accessing the shared resource less frequently.

A **Blue LED** is illuminated to indicate contention issues when both tasks attempt to access the resource simultaneously.

## Project Setup

The project is configured for an STM32 microcontroller using the STM32CubeMX and STM32CubeIDE environments, with FreeRTOS enabled. 

### Task Implementation

The project includes two tasks:

1. **Red LED Task**: Runs at a higher priority, turns on the red LED, accesses the shared resource, turns off the red LED, and delays for 100ms.
2. **Green LED Task**: Runs at a lower priority, turns on the green LED, accesses the shared resource, turns off the green LED, and delays for 500ms.

### Resource Contention Detection

To detect contention:
- A **StartFlag** is used to signal whether the shared resource is in use.
- If a task finds the resource already in use, it turns on the **Blue LED** to signal contention.
- A **500ms simulated read/write operation** is included in the resource access function.

## Access Control with Scheduler Suspension

The `AccessSharedData()` function is protected using **scheduler suspension** to ensure atomic access:

```c
void AccessSharedData(void) {
    if (StartFlag == 1) {
        // Set Start flag to indicate the resource is in use
        StartFlag = 0;
    } else {
        // Resource contention detected: turn on Blue LED
        HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_RESET);
    }

    // Suspend scheduler to prevent context switches during resource access
    vTaskSuspendAll();
    
    // Simulate read/write operation (500 ms delay)
    SimulateReadWriteOperation();
    
    // Resume scheduler after completing resource access
    xTaskResumeAll();
    
    // Set Start flag back to indicate the resource is free
    StartFlag = 1;

    // Turn off Blue LED if it was turned on
    HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_SET);
}

void SimulateReadWriteOperation(void) {
    volatile uint32_t delay_count = 0;
    const uint32_t delay_target = 1000000; // Adjust this value to approximate 500 ms

    // Dummy loop to simulate processing time
    for (delay_count = 0; delay_count < delay_target; delay_count++) {
        __asm("nop"); // No Operation: Keeps the processor busy without changing code behavior
    }
}
```

### Explanation

- **Suspending the Scheduler**: `vTaskSuspendAll()` is called before accessing the shared resource, preventing any task switching until `xTaskResumeAll()` is called. This ensures the access remains atomic.
- **Resource Contention Handling**: If a task tries to access the resource while `StartFlag` is already set to "Down," it turns on the Blue LED, indicating a contention issue.
- **Simulated Delay**: A `for` loop with a no-operation (`__NOP`) is used to simulate a 500ms delay without using `HAL_Delay`, as `HAL_Delay` can be affected by the RTOS tick and potentially interfere with timing during critical sections.

## Prerequisites

- **Hardware**: STM32 microcontroller (with three LEDs connected to GPIO pins).
- **Software**: STM32CubeMX, STM32CubeIDE, FreeRTOS.

## Usage

1. Compile and flash the code to your STM32 board.
2. Run the program with both `RedLed` and `GreenLed` tasks active.
3. Observe the LED behavior:
   - **Red LED** and **Green LED** indicate task execution.
   - **Blue LED** lights up when a contention issue is detected due to simultaneous access attempts.

## Expected Outcome

The **Blue LED** should not light up, indicating that there were no contentioin problem again.

## Conclusion

This project provides a clear demonstration of using scheduler suspension in FreeRTOS to handle shared resource contention in real-time tasks. Suspending the scheduler around the shared resource access ensures that no other task can preempt the current task, preserving data integrity. This approach is useful for critical sections where atomic access is required without interference from other tasks.

---

## Video Demo Exercise 6

https://github.com/user-attachments/assets/d3289e31-6ac8-4f97-9e63-5f1ba3259391

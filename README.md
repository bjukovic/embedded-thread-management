# STM32 FreeRTOS Thread Management and Scheduling

A practical Operating Systems project demonstrating **thread management, task scheduling, multitasking, and inter-task communication** on an STM32 microcontroller using **FreeRTOS and CMSIS-RTOS2**. The project also includes an interactive **HTML/CSS/JavaScript scheduling simulator** for visualizing task states, CPU scheduling, Round Robin execution, and synchronization.

## Overview

This project was developed for the **Operating Systems (CS307)** course at the **International University of Sarajevo**.

The STM32 implementation demonstrates how an embedded system can manage multiple concurrent tasks using an RTOS. The system contains several tasks responsible for LED control, calculations, and UART communication. Tasks communicate using **message queues** and **thread flags**, while the FreeRTOS scheduler manages their execution.

The project also includes a web-based simulator that provides a visual representation of task scheduling and helps demonstrate concepts such as:

* Multitasking
* Thread/task management
* Priority-based scheduling
* Round Robin scheduling
* Context switching
* Task states
* CPU utilization
* I/O blocking
* Message queues
* Thread synchronization

The hardware implementation and simulator complement each other by connecting theoretical operating-system concepts with a practical embedded-system implementation.

---

## Features

### STM32 FreeRTOS Implementation

* Multiple concurrent RTOS tasks
* FreeRTOS-based scheduling
* CMSIS-RTOS2 API
* Three LED control tasks
* Calculation task
* UART communication task
* Idle/default task
* Message queue for inter-task communication
* Thread flags for task synchronization
* GPIO control
* USART2 serial communication
* Preemptive priority-based scheduling
* Round Robin execution between tasks with equal priority
* Delayed/blocked task states using `osDelay()`

The system consists of six main threads/tasks, including the idle task, three LED tasks, a calculation task, and a UART task.

### HTML Scheduling Simulator

The project also contains an interactive web-based simulator implemented with:

* HTML
* CSS
* JavaScript

The simulator can visualize task scheduling in different modes and provides controls for scheduling mode, quantum length, and simulation speed. It represents task states, CPU activity, I/O blocking, and idle periods.

---

## System Architecture

The STM32 application consists of several independent RTOS tasks:

```text
                    ┌─────────────────────┐
                    │   FreeRTOS Kernel   │
                    │      Scheduler      │
                    └──────────┬──────────┘
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
        ▼                      ▼                      ▼
 ┌─────────────┐        ┌─────────────┐       ┌─────────────┐
 │ LED Task 1  │        │ LED Task 2  │       │ LED Task 3  │
 │    PC4      │        │    PC5      │       │    PC6      │
 └─────────────┘        └─────────────┘       └─────────────┘
        │
        │
        ▼
 ┌─────────────┐       Message Queue       ┌─────────────┐
 │ Calculation │ ─────────────────────────> │ UART Task   │
 │    Task     │                            │   USART2    │
 └─────────────┘       Thread Flag         └─────────────┘
                              │
                              ▼
                       UART Transmission
```

The calculation task produces values and places them into a message queue. It then sets a thread flag to notify the UART task. The UART task waits for this notification, retrieves the value from the queue, and transmits it through USART2.

---

## Hardware

The embedded implementation uses an STM32 microcontroller development board.

### Main Components

* STM32 Nucleo-F401RE or STM32F4 Discovery board
* LEDs
* USB connection for programming/debugging
* UART interface

### GPIO Configuration

| Component | Pin    | Function             |
| --------- | ------ | -------------------- |
| LED 1     | PC4    | LED task 1           |
| LED 2     | PC5    | LED task 2           |
| LED 3     | PC6    | LED task 3           |
| UART      | USART2 | Serial communication |

The three LED tasks toggle their corresponding GPIO pins at regular intervals. USART2 is configured for serial communication at **115200 baud, 8 data bits, no parity, and 1 stop bit**.

---

## Software and Technologies

### Embedded System

* **C**
* **STM32CubeIDE**
* **STM32 HAL**
* **CMSIS-RTOS2**
* **FreeRTOS**
* GPIO
* USART2
* Message Queues
* Thread Flags

### Web Simulator

* **HTML**
* **CSS**
* **JavaScript**

The STM32 application uses STM32CubeIDE and the HAL library together with CMSIS-RTOS2/FreeRTOS for task creation, scheduling, communication, and synchronization.

---

## RTOS Tasks

### 1. Idle Task

The idle/default task runs with low priority and provides the system with an idle execution state when other tasks are not ready to run.

### 2. LED Task 1

Controls the LED connected to **PC4**.

The task periodically toggles the GPIO state and then enters a delayed state.

### 3. LED Task 2

Controls the LED connected to **PC5**.

Like the first LED task, it toggles the LED and uses `osDelay()` between executions.

### 4. LED Task 3

Controls the LED connected to **PC6**.

It follows the same periodic execution pattern as the other LED tasks.

The three LED tasks use a **500 ms delay** between toggles.

### 5. Calculation Task

The calculation task periodically increments a counter by **2**.

The resulting value is:

1. Generated by the calculation task.
2. Placed into the message queue.
3. Followed by a thread-flag notification to the UART task.
4. Retrieved by the UART task.
5. Transmitted through USART2.

The calculation task uses a low priority and delays for **1000 ms** between executions.

### 6. UART Task

The UART task waits for the appropriate thread flag.

When the calculation task signals the UART task:

1. The UART task receives the thread flag.
2. It retrieves the value from the message queue.
3. It transmits the value through USART2.

This creates a producer-consumer communication pattern between the calculation and UART tasks.

---

## Task Scheduling

FreeRTOS uses **priority-based preemptive scheduling** to determine which ready task should execute.

When a higher-priority task becomes ready, it can preempt a lower-priority task. Tasks can also enter a blocked state when they call functions such as `osDelay()` or wait for synchronization events.

Tasks with equal priority can share processor time using **Round Robin scheduling**.

A simplified task-state flow can be represented as:

```text
              ┌──────────┐
              │  Ready   │
              └────┬─────┘
                   │
                   ▼
              ┌──────────┐
              │ Running  │
              └────┬─────┘
                   │
          ┌────────┴────────┐
          │                 │
       osDelay()        Preemption
          │                 │
          ▼                 ▼
     ┌──────────┐       ┌──────────┐
     │ Blocked  │       │  Ready   │
     └────┬─────┘       └──────────┘
          │
          │ Delay expires
          ▼
     ┌──────────┐
     │  Ready   │
     └──────────┘
```

The RTOS tick and context-switching mechanisms allow the scheduler to switch between tasks as their states and priorities change.

---

## Inter-Task Communication

Two main mechanisms are used for communication and synchronization between tasks.

### Message Queue

The calculation task places generated values into a message queue:

```text
Calculation Task
       │
       │ Generated value
       ▼
 Message Queue
       │
       │ Retrieved value
       ▼
    UART Task
```

The message queue allows data to be passed between tasks without requiring them to execute simultaneously.

### Thread Flags

After placing a value into the message queue, the calculation task sets a thread flag.

The UART task waits for this flag before attempting to retrieve the data.

This provides synchronization between the producer and consumer tasks.

---

## HTML Scheduling Simulator

The project includes an interactive HTML-based simulator designed to visualize the behavior of scheduled tasks.

The simulator demonstrates concepts that are also present in the STM32 implementation.

### Simulator Features

* Sequential scheduling mode
* Parallel/Round Robin mode
* Adjustable quantum length
* Adjustable simulation speed
* Task state visualization
* CPU activity visualization
* I/O blocking
* Idle periods
* Queue management
* Task state transitions
* Synchronization behavior

The JavaScript implementation manages task transitions, queue behavior, and synchronization while the interface provides a visual representation of the scheduling process.

---

## Project Structure

A recommended repository structure is:

```text
stm32-freertos-thread-scheduling/
│
├── STM32/
│   ├── Core/
│   │   ├── Inc/
│   │   └── Src/
│   │
│   ├── Drivers/
│   │
│   ├── *.ioc
│   ├── .project
│   └── .cproject
│
├── HTML-Simulator/
│   ├── index.html
│   ├── style.css
│   └── script.js
│
├── docs/
│   └── images/
│
├── README.md
│
├── Operating Systems Project 1 Report.docx
│
└── .gitignore
```

The exact STM32 source structure may differ depending on the STM32CubeIDE project configuration.

---

## How the STM32 Application Works

The main program follows this general sequence:

```text
System Initialization
        │
        ▼
HAL Initialization
        │
        ▼
Clock Configuration
        │
        ▼
GPIO Initialization
        │
        ▼
USART2 Initialization
        │
        ▼
FreeRTOS Initialization
        │
        ▼
Create Tasks
        │
        ▼
Create Message Queue
        │
        ▼
Start FreeRTOS Kernel
        │
        ▼
      Scheduler
        │
        ├───────────────┐
        │               │
        ▼               ▼
    LED Tasks       Calculation
                        │
                        ▼
                 Message Queue
                        │
                        ▼
                  Thread Flag
                        │
                        ▼
                    UART Task
                        │
                        ▼
                  USART2 Output
```

The application initializes the hardware and RTOS objects before starting the FreeRTOS kernel. Once the scheduler starts, the RTOS takes responsibility for task execution and scheduling.

---

## Scheduling Concepts Demonstrated

This project demonstrates several important Operating Systems concepts in an embedded environment.

### Multitasking

Multiple tasks can exist within the same application and execute according to their scheduling requirements.

### Preemptive Scheduling

A higher-priority ready task can interrupt a lower-priority running task.

### Round Robin Scheduling

Tasks with equal priority can share CPU time through time-sliced execution.

### Context Switching

The RTOS can switch the processor from one task to another when scheduling conditions require it.

### Blocking

Tasks can temporarily stop executing when waiting for a delay, synchronization event, or other resource.

### Inter-Task Communication

Message queues allow tasks to exchange data.

### Synchronization

Thread flags provide a mechanism for notifying and synchronizing tasks.

These concepts are demonstrated both through the STM32 implementation and the accompanying simulator.

---

## Running the STM32 Project

### Requirements

* STM32CubeIDE
* Compatible STM32 development board
* USB cable
* STM32CubeIDE-compatible toolchain
* Serial terminal for observing USART2 output

### Steps

1. Clone or download the repository.

2. Open **STM32CubeIDE**.

3. Import the STM32 project into the workspace.

4. Open the `.ioc` configuration file if hardware configuration needs to be inspected or regenerated.

5. Build the project.

6. Connect the STM32 development board through USB.

7. Flash the application to the board.

8. Open a serial terminal configured for:

```text
Baud rate: 115200
Data bits: 8
Parity: None
Stop bits: 1
```

9. Observe the LEDs and UART output.

The expected behavior includes periodic LED toggling and UART transmission of values generated by the calculation task.

---

## Running the HTML Simulator

The simulator can be run directly from a web browser.

1. Navigate to the `HTML-Simulator` directory.
2. Open `index.html` in a modern web browser.
3. Select the desired scheduling mode.
4. Adjust the simulation speed and quantum length if available.
5. Start the simulation.
6. Observe task execution, CPU activity, task states, and blocking behavior.

No server-side component is required for the basic HTML/CSS/JavaScript simulator.

---

## Learning Objectives

The project was designed to provide practical experience with:

* Embedded operating systems
* RTOS concepts
* Thread/task management
* Task scheduling
* Preemptive scheduling
* Round Robin scheduling
* Context switching
* Inter-task communication
* Synchronization
* GPIO programming
* UART communication
* STM32 development
* FreeRTOS
* CMSIS-RTOS2
* Visualization of operating-system concepts

The project connects theoretical scheduling concepts with a practical embedded implementation and a visual simulation environment.

---

## Project Report

The complete project report is included in the repository:

**Operating Systems Project 1 Report.docx**

The report contains the detailed system description, hardware and software configuration, task implementation, scheduling concepts, simulator description, and references.

---

## Authors

**Operating Systems — CS307, Fall 2025**

* Hena Šehović
* Berina Juković
* Berin Žunić

---

## References

The project report references the following technologies and documentation:

* FreeRTOS documentation
* STM32CubeIDE documentation
* STM32 HAL documentation
* CMSIS-RTOS2 documentation

See the included project report for the complete reference list.

---

## License

This project was developed as an academic project for the **Operating Systems (CS307)** course at the **International University of Sarajevo**.

The project is intended primarily for educational and demonstration purposes.

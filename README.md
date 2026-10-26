# Sensorless BLDC Motor Drive Using Back-EMF Zero-Crossing Detection

> A custom-designed sensorless three-phase BLDC motor controller using a MOSFET-based inverter, IR2104 gate drivers, Back-EMF zero-crossing detection, six-step commutation, and PWM speed control.

![Project Status](https://img.shields.io/badge/Status-Under%20Development-orange)
![Platform](https://img.shields.io/badge/Platform-Embedded%20Systems-blue)
![PCB](https://img.shields.io/badge/PCB-KiCad-green)
![Supply](https://img.shields.io/badge/Supply-12V-red)

## Project Overview

This project focuses on designing and developing a **Sensorless Brushless DC (BLDC) Motor Drive** capable of controlling a three-phase BLDC motor without using Hall-effect position sensors.

Instead of directly measuring rotor position, the controller uses **Back Electromotive Force (Back-EMF) zero-crossing detection** to estimate rotor position and determine the appropriate commutation timing.

The motor drive consists of a three-phase inverter built using six N-channel MOSFETs, three IR2104 half-bridge gate driver ICs, and a microcontroller-based control system.

The project initially uses an Arduino Uno for control development, with the long-term goal of integrating the microcontroller, power regulation, sensing circuitry, and inverter on a single custom-designed PCB.

The system is intended to operate from a 12 V DC supply and drive an A2212 1000KV BLDC motor, initially without a propeller.

**Project Status:** Hardware and controller development in progress. Complete sensorless operation and final PCB validation are not yet claimed.

## Project Objectives

- Develop a sensorless three-phase BLDC motor drive.
- Eliminate the need for Hall-effect rotor position sensors.
- Implement Back-EMF zero-crossing detection.
- Develop six-step electronic commutation.
- Design a three-phase inverter using six N-channel MOSFETs.
- Integrate IR2104 half-bridge gate driver ICs.
- Implement PWM-based motor speed control.
- Develop an open-loop startup and closed-loop commutation transition.
- Monitor motor control parameters using an LCD.
- Design a custom PCB using KiCad.
- Integrate the microcontroller and driver on a single PCB.
- Operate the system using a single 12 V DC input.

## Hardware Components

| Component | Quantity | Purpose |
|---|---:|---|
| A2212 1000KV BLDC Motor | 1 | Three-phase motor |
| Microcontroller | 1 | Motor control and commutation |
| IR2104 Gate Driver IC | 3 | High-side and low-side MOSFET driving |
| 06N03LA N-Channel MOSFET | 6 | Three-phase inverter switching |
| Buck Voltage Regulator | 1 | Microcontroller power supply |
| Back-EMF Sensing Network | 3 | Floating-phase voltage sensing |
| DC-Link Capacitors | As required | Supply filtering |
| Bootstrap Capacitors | 3 | High-side gate driver supply |
| Bootstrap Diodes | 3 | Bootstrap charging |
| Gate Resistors | 6 | MOSFET gate switching control |
| Pull-Down Resistors | As required | Defined control states |
| LCD Display | 1 | Motor parameter monitoring |
| NTC Temperature Sensor | 1 | Temperature monitoring |
| Terminal Blocks | As required | Power and motor connections |
| 12 V DC Supply | 1 | Main power input |
| Custom PCB | 1 | Hardware integration |

*Component ratings, regulator choice, protection components, and final microcontroller selection are subject to design verification.*

## System Architecture

```text
                 12 V DC POWER SUPPLY
                          |
             +------------+------------+
             |                         |
             v                         v
       DC-Link Filter           Buck Regulator
             |                         |
             v                         v
      Three-Phase Inverter       Microcontroller
       (6 N-MOSFETs)                   |
             ^                         |
             |                         +---- PWM and Commutation
       3 x IR2104 Drivers <------------+
             |
             v
        BLDC Motor
        Phase U, V, W
             |
             v
       Back-EMF Sensing
             |
             +-----------------------> Microcontroller
                                            |
                                            +--> LCD Display
                                            |
                                            +<-- NTC Sensor
```

The microcontroller controls the inverter through the gate drivers and receives feedback from the Back-EMF sensing circuit.

## Working Principle

### 1. Three-Phase Inverter

The motor controller uses a three-phase inverter consisting of six N-channel MOSFETs arranged as three half-bridges.

Each half-bridge controls one motor phase:

- Phase U
- Phase V
- Phase W

The MOSFETs switch the 12 V DC input to generate the required phase excitation sequence.

### 2. IR2104 Gate Driver

Three IR2104 gate driver ICs are used to control the inverter MOSFETs.

Each IR2104 drives one half-bridge and provides high-side and low-side gate drive outputs.

The driver uses a bootstrap circuit to supply the high-side gate drive.

The microcontroller provides the required input control signals, while the driver manages complementary switching with its specified dead-time behavior.

The shutdown input can be used to disable the driver outputs.

### 3. Six-Step Commutation

The BLDC motor is controlled using six-step commutation.

During each commutation step:

- One motor phase is driven high.
- One motor phase is driven low.
- The remaining phase is left floating.

| Step | High Phase | Low Phase | Floating Phase |
|---|---|---|---|
| 1 | U | V | W |
| 2 | U | W | V |
| 3 | V | W | U |
| 4 | V | U | W |
| 5 | W | U | V |
| 6 | W | V | U |

The switching sequence produces a rotating stator magnetic field that drives the rotor.

The exact sequence and polarity must be matched to the motor wiring and desired rotation direction.

### 4. Back-EMF Zero-Crossing Detection

A rotating BLDC motor generates Back Electromotive Force in its windings.

During six-step commutation, the unenergized phase is monitored to estimate rotor position.

The floating-phase voltage is compared against the motor's virtual neutral reference.

A zero-crossing event occurs when the floating-phase Back-EMF crosses the neutral reference.

After detecting a valid zero crossing, the controller schedules the next commutation after approximately 30 electrical degrees.

This allows sensorless commutation without Hall-effect sensors.

### 5. Motor Startup

Back-EMF detection is not reliable when the motor is stationary because the generated Back-EMF is approximately zero.

Therefore, the controller requires an initial open-loop startup sequence.

The proposed startup process is:

1. Initialize the microcontroller and gate drivers.
2. Apply a controlled rotor alignment sequence if required.
3. Begin six-step commutation at a low frequency.
4. Gradually increase the commutation frequency.
5. Monitor the floating-phase Back-EMF.
6. Detect stable zero-crossing events.
7. Transition to sensorless closed-loop commutation.

The startup parameters require practical tuning for the selected motor and load.

### 6. PWM Speed Control

Pulse Width Modulation is used to regulate the effective voltage applied to the motor.

The PWM duty cycle is calculated as:

```text
Duty Cycle (%) = (PWM ON Time / PWM Period) x 100
```

Increasing the duty cycle generally increases the applied motor voltage and available torque.

The actual motor speed depends on supply voltage, load, commutation timing, and motor characteristics.

### 7. LCD Monitoring

The proposed LCD interface displays:

- Active commutation step
- PWM value
- PWM duty cycle
- Back-EMF detection status
- Motor operating status

Future development may include estimated RPM and temperature readings.

### 8. Temperature Monitoring

An NTC temperature sensor is planned for monitoring the motor or power-stage temperature.

The proposed logic stops motor operation when the measured temperature exceeds a defined limit.

Operation may resume after cooling below a lower threshold, provided the fault condition has cleared and restart is safe.

## Custom PCB Design

KiCad is used for schematic design and PCB development.

The PCB design includes:

- Microcontroller circuitry
- Three IR2104 gate drivers
- Six MOSFET power stage
- Bootstrap components
- DC-link filtering
- Back-EMF sensing networks
- Buck voltage regulation
- LCD interface
- Temperature sensor interface
- Power and motor terminals
- Decoupling capacitors
- Control and signal connections

### PCB Design Considerations

- Minimize high-current switching loop areas.
- Use sufficiently wide copper paths for motor current.
- Place DC-link capacitors close to the inverter.
- Keep gate-drive loops short.
- Separate sensitive Back-EMF sensing traces from noisy switching nodes.
- Provide suitable grounding and decoupling.
- Maintain safe spacing between switching nodes and control signals.
- Consider MOSFET thermal dissipation.
- Provide accessible test points for debugging.

## Software Architecture

The planned firmware is organized into the following functions:

```text
System Initialization
        |
        v
Gate Driver Configuration
        |
        v
Open-Loop Motor Startup
        |
        v
Back-EMF Detection
        |
        v
Zero-Crossing Validation
        |
        v
Commutation Timing Calculation
        |
        v
Six-Step Commutation
        |
        v
PWM Speed Regulation
        |
        +------> LCD Monitoring
        |
        +------> Temperature Monitoring
        |
        v
Continuous Motor Control
```

## Technologies Used

| Category | Technology |
|---|---|
| Motor | A2212 1000KV BLDC |
| Initial Controller | Arduino Uno |
| Final Controller | MCU selection in progress |
| Programming | Embedded C/C++ |
| Development Environment | Arduino IDE |
| PCB Design | KiCad |
| Gate Driver | IR2104 |
| Power Switches | N-Channel MOSFETs |
| Control Technique | Six-Step Commutation |
| Rotor Position Estimation | Back-EMF Zero-Crossing |
| Speed Control | PWM |
| Power Supply | 12 V DC |
| Monitoring | LCD Interface |

## Development Status

| Development Activity | Status |
|---|---|
| Project concept and architecture | Completed |
| Component selection | In progress |
| Three-phase inverter schematic | Designed; verification pending |
| IR2104 driver circuit | Designed; verification pending |
| Back-EMF sensing design | In progress |
| Arduino control development | In progress |
| Open-loop startup validation | Pending |
| Sensorless commutation validation | Pending |
| Single-PCB MCU integration | In progress |
| PCB fabrication and assembly | Pending |
| Complete motor testing | Pending |

## Expected Results

The intended system will:

- Start and operate a three-phase BLDC motor.
- Detect Back-EMF zero-crossing events.
- Perform sensorless six-step commutation.
- Control motor speed using PWM.
- Display important motor control parameters.
- Operate from a single 12 V DC input.
- Integrate the controller and inverter into one custom PCB.

These are design targets and should not be interpreted as completed experimental results.

## Applications

- BLDC Motor Control Research
- Embedded Systems Learning
- Power Electronics Education
- Robotics
- Small Electric Drives
- Motor Controller Prototyping
- Sensorless Control Research
- Custom ESC Development

## Challenges

- Detecting low-amplitude Back-EMF during startup
- Filtering PWM switching noise
- Selecting accurate zero-crossing thresholds
- Achieving reliable startup under different loads
- Preventing incorrect MOSFET switching
- Managing power-stage heating
- Reducing electrical interference
- Designing reliable high-side gate drive circuits
- Transitioning from open-loop to closed-loop control

## Future Improvements

- Complete hardware fabrication and validation
- Integrate the microcontroller and inverter on a single PCB
- Implement reliable sensorless startup
- Add motor RPM estimation
- Improve Back-EMF signal filtering
- Implement overcurrent and undervoltage protection
- Add configurable speed control
- Improve MOSFET thermal management
- Add serial monitoring and debugging
- Optimize commutation timing
- Explore advanced motor control techniques

## Safety Considerations

This project involves high-current switching circuits and a rotating electrical machine.

- Use a current-limited DC supply during initial testing.
- Test without a propeller during early development.
- Verify MOSFET gate-drive signals before connecting the motor.
- Confirm that complementary switches cannot conduct simultaneously.
- Provide appropriate fusing and fault shutdown.
- Check component voltage and current ratings.
- Ensure the bootstrap supply operates within driver specifications.
- Avoid touching the power stage during operation.
- Secure the motor mechanically before testing.
- Do not rely on firmware alone for critical electrical protection.

## Repository Structure

```text
Sensorless-BLDC-Motor-Drive/
|
|-- README.md
|-- LICENSE
|
|-- hardware/
|   |-- schematics/
|   |-- pcb-layout/
|   |-- gerber-files/
|   |-- bom/
|
|-- firmware/
|   |-- arduino-prototype/
|   |-- integrated-controller/
|
|-- images/
|
|-- videos/
|
|-- documentation/
```

*This structure represents the intended organization. Folders will be populated as development progresses.*

## Learning Outcomes

Through this project, I am developing practical knowledge in:

- BLDC motor operation
- Power electronic converters
- Three-phase inverter design
- MOSFET switching
- Gate driver circuits
- Back-EMF sensing
- Sensorless motor control
- Six-step commutation
- PWM speed control
- Embedded programming
- PCB schematic and layout design
- Hardware integration
- Electrical troubleshooting

The project helps connect theoretical concepts from electrical machines, power electronics, and embedded systems with practical motor controller development.

## Author

**GitHubKailash**

Electrical and Electronics Engineering

GitHub: https://github.com/GitHubKailash

## License

This project is intended to be released under the MIT License. See the LICENSE file once added to the repository.


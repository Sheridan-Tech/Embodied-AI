# GrowBot

These notes are an independent technical summary and implementation study based on publicly available materials. This repository is not affiliated with, endorsed by, or sponsored by the original video creator. Source links are provided for attribution. Working reconstruction based on public notes, observed claims, and our own implementation planning.

GrowBot is a low-cost embodied-AI robot concept built around a simple physical body, local motor-control policies, cloud-based reasoning models, reinforcement-learning training, memory loops, and predictive state modeling.

The central idea is to separate fast physical control from slower reasoning. Low-level movement runs locally and continuously, while higher-level interpretation, personality, planning, and code generation run through LLMs on a server.

## Source References

### Videos

* **How Nature Solved Robotics**
  [https://www.youtube.com/watch?v=S67z2aekBrI](https://www.youtube.com/watch?v=S67z2aekBrI)

* **Growbot Website**
  [https://artoftheproblem.com/pages/growbot](https://artoftheproblem.com/pages/growbot)

* **Ray Summit 2025 Keynote: Physical AI Turing Test with Jim Fan from NVIDIA**
  [https://www.youtube.com/watch?v=7fDiui8cAVQ](https://www.youtube.com/watch?v=7fDiui8cAVQ)

### Simulation Software

* **MuJoCo**
  MuJoCo is a free and open-source physics engine intended to support robotics research and development.
  [https://mujoco.org/](https://mujoco.org/)

* **Isaac Lab**
  Used in the GrowBot notes as the 3D physics simulation environment for building and training a digital twin.

### Compute

* **Google Colab with H100 GPU access**
  The notes describe renting an H100 GPU through Google Colab for a few hours at an estimated cost of roughly **$10 to $15** to run massively parallel reinforcement-learning simulations.

## Core Concept

The GrowBot design is based on reducing embodied AI to the simplest practical physical platform:

* A minimal two-legged robot body.
* A low-cost processing chip.
* A camera for vision.
* An IMU for motion and orientation sensing.
* A microphone, speaker, and light ring for interaction.
* A small battery for untethered operation.
* Local neural-network motor policies for balance and movement.
* Cloud-connected LLMs for reasoning, memory, personality, image understanding, and dynamic behavior generation.

The creator’s claim is that this approach condenses work that previously resembled a large, expensive research program into a robot that can be built at home for approximately **$80 to $100**.

## Hardware and Bill of Materials

The body is intentionally simple so the project can focus on intelligence, training, and control rather than complex mechanical design.

| Subsystem            | Component                                |                     Example Part / Notes | Approximate Cost |
| -------------------- | ---------------------------------------- | ---------------------------------------: | ---------------: |
| Brain                | Processing chip with wireless capability |             Raspberry Pi Zero 2 W, 3, or 4 |             ~$15+ |
| Legs / Actuation     | 2 simple servo motors                    |           Tower Pro SG90 9g micro servos |         ~$3 each |
| Vision               | 5-megapixel camera                       |                  Raspberry Pi 5MP camera |              ~$5 |
| Motion / Touch Sense | IMU sensor                               |                             MPU-6050 IMU |             <$10 |
| Audio Input          | Digital microphone                       |              INMP441 MEMS I2S microphone |    Not specified |
| Audio Output         | Speaker                                  |                            Small speaker |    Not specified |
| Visual Output        | Light ring                               |           WS2812 7-bit 5050 RGB LED ring |    Not specified |
| Power                | Small drone battery                      | VICMILE 7.4V 1200mAh 2S 35C LiPo battery |    Not specified |

**Estimated total cost:** approximately **$80 to $100**.

### Most Important Sensor: IMU

The IMU is treated as the most important sensor in the design because it gives the robot a way to physically “feel” its own movement. It measures:

* Acceleration.
* Rotation.
* Motion across three axes.
* Orientation changes.
* Physical instability, falling, tilt, and recovery conditions.

This motion data becomes the key input for both the fast local motor-control layer and the slower AI reasoning layer.

## Software and AI Architecture

GrowBot uses a two-layer intelligence architecture:

1. **Fast unconscious layer**, local motor reflexes.
2. **Slow conscious layer**, cloud-connected reasoning, memory, code generation, and personality.

This mirrors a biological distinction between rapid motor control and slower deliberative reasoning.

## Layer 1: Fast Unconscious Motor Control

The fast layer handles basic movement, balance, posture, and recovery.

### Role

* Maintain balance.
* Control posture.
* Execute walking, standing, spinning, or recovery behaviors.
* React quickly to physical motion changes.
* Run without waiting for an LLM response.

### Implementation

* Uses small neural networks, also described as **policy networks**.
* Runs locally on the low-cost processing chip.
* Executes approximately **50 times per second**.
* Takes recent IMU sensor history as input.
* Outputs direct motor actions to the servos.

### Input and Output

**Input:**

* The 5 most recent IMU readings.
* These readings provide a short history of acceleration, rotation, and body motion.

**Output:**

* Direct motor commands.
* Servo actions for movement, standing, balancing, readjustment, and recovery.

The important design principle is that motor control is not manually scripted. The fast layer learns useful physical behaviors through reinforcement learning and then executes them locally.

## Layer 2: Slow Conscious Reasoning Layer

The slower layer handles higher-level cognition, including interpretation, interaction, planning, and behavioral creativity.

### Role

* Interpret sensor data.
* Process images from the camera.
* Generate logical commands.
* Manage conversation and personality.
* Write new code for behaviors.
* Blend generated behaviors with trained motor policies.
* Read and write memory.
* Extract lessons from past experience.

### Cloud-Connected LLM Server

The robot connects wirelessly to a server that calls multiple large language models.

The notes emphasize that the LLM receives **raw IMU data directly**, meaning acceleration and rotation values are sent to the model rather than being translated first into simplified labels such as “falling,” “standing,” or “tilting.”

According to the notes, fast models can process this sensor stream and respond in approximately **4 seconds**, while some command and image-processing tasks can complete in roughly **1 second**.

### Direct Motor Access

The LLM is given access to the robot’s motors through code execution.

This allows the model to:

* Write movement code dynamically.
* Execute new actions on the fly.
* Compose behaviors that were not hard-coded in advance.
* Combine generated movement code with trained local policy networks.
* Create more complex and expressive motion than the fast layer alone.

This architecture allows the robot to use learned reflexes for stability while using LLM-generated code for creative or situational behaviors.

## AI Models and Tools

The GrowBot notes describe using multiple AI models, each selected for a different function.

| Model / Tool  | Role                                                                                                                                     |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Gemini Flash  | Fast processing, image understanding, and logical command generation. Described as responding in about 1 second for some tasks.          |
| Claude Sonnet | Used as the “sweet spot” for complex logic, memory extraction, cleanup, and higher-quality reasoning.                                    |
| Claude Haiku  | Used for fast responses, but described by the creator as more dramatic and occasionally disobedient.                                     |
| Mammoth       | API aggregator used to access and switch between different models cost-effectively. Notes mention pricing starting around €10 per month. |
| Isaac Lab     | 3D physics simulator used to build the digital twin and train policies.                                                                  |
| MuJoCo        | Free and open-source robotics physics engine, potentially relevant for digital twin simulation and reinforcement-learning experiments.   |
| Google Colab  | Used to rent GPU compute, including H100 access, for simulation training.                                                                |

## Reinforcement-Learning Training Process

The GrowBot notes emphasize that lifelike walking and balance are not pre-programmed manually. Instead, the robot learns motor skills through reinforcement learning.

## Step 1: Build a Digital Twin

Create a virtual 3D replica of the physical robot in a simulator such as Isaac Lab or a similar robotics physics environment.

The digital twin should represent:

* Robot geometry.
* Servo placement.
* Joint limits.
* Mass distribution.
* Leg configuration.
* IMU placement.
* Physical constraints.
* Contact with ground surfaces.

## Step 2: Define Training Goals

The simulated robot is given specific objectives, such as:

* Stand upright.
* Balance.
* Recover from being knocked over.
* Move toward light.
* Spin.
* Readjust on uneven surfaces.
* Recreate a target motion.

## Step 3: Run Massively Parallel Simulations

The digital robot attempts the goal millions of times in simulation.

Each attempt gives the policy network feedback about whether its motor outputs helped or hurt performance.

The training process keeps and reinforces network changes that improve the robot’s ability to achieve the goal.

## Step 4: Use Cloud GPU Compute

The notes describe using rented GPU compute, such as an H100 through Google Colab, to accelerate the simulation process.

Estimated cost from the notes:

* **$10 to $15** for a few hours of H100 access.
* Potentially enough to train a usable policy in the same day.

## Step 5: Transfer Neural Weights to the Physical Robot

After the policy network is trained in simulation, transfer the learned neural weights to the physical robot.

The intended result is that the physical robot can immediately perform the trained behavior, such as:

* Balancing.
* Spinning.
* Standing.
* Recovering from falls.
* Adjusting to different real-world surfaces.

This is the sim-to-real transfer stage.

## Advanced Cognitive Features

GrowBot is described as more than a toy because it includes memory, self-reflection, and self-improvement loops.

## Memory System

The AI has a digital memory that it can read from and write to.

This allows the robot to:

* Remember prior interactions.
* Adapt to specific people.
* Build profiles of people it interacts with.
* Modify behavior based on past experience.
* Learn preferences and reactions.
* Associate physical events with social or behavioral meaning.

Example from the notes:

* The robot could learn to “play dead” when touched.

## Dreaming Loop

The notes describe a periodic “dreaming” process used to clean and consolidate memory.

Because memory logs can become cluttered with redundant, stale, or conflicting information, the robot periodically sends its memory log to a more capable model, such as Claude Sonnet.

The model then:

* Cleans up the memory.
* Extracts high-level lessons.
* Resolves conflicts where possible.
* Refines future strategies.
* Updates personality-relevant behavior patterns.
* Converts raw logs into more useful long-term knowledge.

## Self-Improvement and the Mimic Game

The robot can use its own motion data as feedback for self-improvement.

In the described “mimic game,” the robot:

1. Reads its own motion data.
2. Writes or adjusts movement programs.
3. Attempts to recreate specific physical motions.
4. Measures whether the resulting motion matches the intended motion.
5. Iterates based on the difference between intended and observed movement.

This creates a loop where the robot can experiment with its own body and improve its movement capabilities over time.

## The Missing Piece: A Robotic Cerebellum

The GrowBot notes identify predictive physical imagination as a frontier-level missing piece.

The issue is sensor delay. Physical sensors always lag reality, with the notes estimating this delay at roughly **one-tenth of a second**.

A robot that only reacts to sensor data is therefore always reacting to the recent past. For more advanced motor control, the robot needs to predict what will happen next.

## Predictive State Modeling

The proposed solution is to add mini-networks that predict immediate future physical states.

These predictive branches would estimate:

* Next body position.
* Momentum.
* Rotation.
* Physical stability.
* Ground interaction.
* Expected IMU readings.
* Motor response.
* Near-future body dynamics.

## Error Feedback Loop

The robot compares prediction against reality:

1. Predict the next physical state.
2. Observe the actual sensor outcome.
3. Compare prediction to reality.
4. Calculate the prediction error.
5. Feed the error back into the system.
6. Improve future motor coordination.

The notes compare this to the function of the human cerebellum, which supports predictive motor coordination rather than simple reaction.

## Consolidated System Blueprint

The full GrowBot system can be summarized as follows:

```text
Physical Robot
  ├─ Raspberry Pi (Zero 2 W, 3, or 4) or similar low-cost wireless processor
  ├─ 2 servo-driven legs
  ├─ 5MP camera
  ├─ MPU-6050 IMU
  ├─ microphone, speaker, and light ring
  └─ small LiPo drone battery

Fast Local Control Layer
  ├─ small policy networks
  ├─ runs locally on the robot
  ├─ executes around 50 Hz
  ├─ input: recent IMU history
  └─ output: direct servo commands

Cloud Reasoning Layer
  ├─ LLM server connection
  ├─ raw IMU data ingestion
  ├─ image processing
  ├─ code generation
  ├─ memory management
  ├─ personality and interaction
  └─ dynamic behavior composition

Training Pipeline
  ├─ build digital twin
  ├─ simulate in Isaac Lab, MuJoCo, or similar
  ├─ train with reinforcement learning
  ├─ run massively parallel simulations on rented GPU compute
  └─ transfer trained weights to the physical robot

Advanced Learning Loops
  ├─ read/write memory
  ├─ dreaming-based memory cleanup
  ├─ mimic-game self-improvement
  └─ predictive cerebellum-style state modeling
```

## Practical Build Sequence

A practical build path would be:

1. Assemble the minimal hardware platform.
2. Validate power, servo control, camera, microphone, speaker, LEDs, and IMU readings.
3. Stream IMU data locally and confirm clean acceleration and rotation measurements.
4. Implement basic local servo movement.
5. Build a simple digital twin in Isaac Lab, MuJoCo, or another robotics simulator.
6. Train one narrow policy first, such as standing, balancing, or spinning.
7. Transfer the trained policy to the physical robot.
8. Add a wireless server connection for LLM control.
9. Send raw IMU data and camera data to the reasoning layer.
10. Allow the LLM to generate controlled motor behavior code.
11. Add memory read/write capabilities.
12. Add periodic memory cleanup, or “dreaming.”
13. Add self-improvement loops using motion data feedback.
14. Experiment with predictive state modeling to reduce the impact of sensor lag.

## Important Design Principles

* Keep the physical robot simple so intelligence and control can be the focus.
* Use the IMU as the primary physical feedback sensor.
* Do not hand-code lifelike motion when it can be learned in simulation.
* Separate fast local reflexes from slower reasoning.
* Keep safety boundaries around LLM-generated motor code.
* Use simulation before real-world motor experiments.
* Treat memory as an evolving system that needs cleanup and consolidation.
* Use prediction, not just reaction, for higher-quality physical control.

## Open Questions and Risks

These are the main unresolved or implementation-sensitive areas:

* Whether the low-cost processing chip can reliably run the required local policy networks at the target rate.
* How well the simulated policy transfers to the real robot without additional tuning.
* Whether the servo hardware has enough precision, torque, and repeatability for the trained behaviors.
* How to safely constrain LLM-generated motor code.
* How raw IMU data should be formatted for LLM interpretation.
* Whether LLM response latency is acceptable for anything beyond high-level behavior generation.
* How to prevent memory from accumulating false, redundant, or contradictory information.
* How to design the cerebellum-style predictive model so it improves motor control rather than adding instability.
* Whether MuJoCo, Isaac Lab, or another simulator is the best fit for the first implementation.

## Summary

GrowBot is a compact embodied-AI robot architecture built around a minimal physical platform, local reinforcement-learned motor policies, cloud-based LLM cognition, memory consolidation, and predictive physical modeling.

Its most important architectural idea is the division between fast local motor reflexes and slower cloud-based reasoning. The fast layer keeps the robot physically stable, while the slow layer provides interpretation, personality, memory, and creative behavior generation.

The long-term frontier is the addition of a cerebellum-like predictive system that allows the robot to anticipate near-future physical states instead of merely reacting to delayed sensor data.

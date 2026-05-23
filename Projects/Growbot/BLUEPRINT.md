# GrowBot Build Blueprint from “How Nature Solved Robotics”

## Source Status

This document consolidates the available build information for GrowBot from the video “How Nature Solved Robotics,” the existing GrowBot project notes, and the current repository materials.

The video appears to be organized around these major sections:

* The robot that did not want to be alone.
* Meet GrowBot.
* Building the approximately $80 body.
* Learning to move, described as System 1.
* First successful movement moments.
* Handing the controls to AI.

A full transcript was not available during extraction, so this document should be treated as a practical reconstruction rather than a verbatim transcript.

## Project Goal

GrowBot is a low-cost embodied-AI robot intended to demonstrate how a simple physical body can be combined with:

* Local reflex-like motor policies.
* Digital twin simulation.
* Reinforcement learning.
* Cloud-based LLM reasoning.
* Raw sensor interpretation.
* Dynamic code-generated behaviors.
* Memory and self-improvement loops.
* Predictive state modeling inspired by biological motor systems.

The main architectural idea is to split intelligence into two layers:

1. A fast local motor-control layer that runs continuously and handles balance, posture, and reflexes.
2. A slower cloud-connected reasoning layer that handles interpretation, planning, personality, memory, vision, and behavior generation.

## Physical Robot Architecture

GrowBot is intentionally mechanically simple. The design uses a minimal body so the intelligence, control, and learning pipeline can be the focus.

### Core Body Requirements

The robot body should include:

* A small onboard processor with wireless capability.
* Two servo-driven legs.
* A camera.
* An IMU.
* A microphone.
* A speaker.
* A light ring.
* A small battery.
* A lightweight printed or fabricated body structure.

## Bill of Materials

| Subsystem       | Component                                 | Example / Current Candidate              | Notes                                                                                  |
| --------------- | ----------------------------------------- | ---------------------------------------- | -------------------------------------------------------------------------------------- |
| Main processor  | Low-cost wireless processing board        | Raspberry Pi Zero 2 W                    | Handles local runtime, wireless connection, sensor I/O, and communication with server. |
| Actuation       | 2 micro servos                            | Tower Pro SG90 9g micro servos           | Drives the two legs. Verify torque, backlash, current draw, and stall behavior.        |
| Vision          | 5MP camera                                | Raspberry Pi 5MP camera                  | Used for visual input to the reasoning layer.                                          |
| Motion sensing  | IMU                                       | MPU-6050                                 | Measures acceleration and angular velocity. Primary physical feedback sensor.          |
| Audio input     | Digital microphone                        | INMP441 MEMS I2S microphone              | Used for voice/audio interaction.                                                      |
| Audio output    | Small speaker                             | TBD                                      | Used for robot responses and personality.                                              |
| Visual output   | LED light ring                            | WS2812 7-bit 5050 RGB LED ring           | Used for expression/status feedback.                                                   |
| Power           | Small LiPo drone battery                  | VICMILE 7.4V 1200mAh 2S 35C LiPo battery | Requires regulation for Pi, servos, sensors, and LEDs.                                 |
| Mechanical body | 3D printed or lightweight fabricated body | TBD                                      | Needs servo mounts, leg geometry, battery retention, camera mount, and IMU mounting.   |

Estimated target cost: approximately $80 to $100.

## Mechanical Design Notes

The current project notebook references a refined short-leg standing configuration:

* Fixed refined leg length: 64 mm.
* Sole size: 21 x 13 mm.
* Rocker lift: 1.2 mm.
* Flat-middle fraction: 0.46.
* Current best standing policy checkpoint: `standing_balance_v1_30M`.
* Local CAD reference from the original author’s environment: `Legs 64mm final refined r1p2 f@p46.step`.

These details are important because the learned policy is tied to body geometry. If the geometry changes substantially, the policy may not transfer cleanly.

## Sensor Architecture

### IMU

The IMU is the most important sensor in the GrowBot design because it provides the robot’s internal sense of motion.

The IMU provides:

* Acceleration data.
* Rotation / angular velocity data.
* Short-term body motion history.
* Tilt and fall information.
* Recovery-state feedback.

The local motor policy uses recent IMU readings as its primary observation input.

### Camera

The camera supports the slower reasoning layer.

Likely uses:

* Scene understanding.
* Human interaction.
* Object recognition.
* Light-seeking or target-seeking behaviors.
* Contextual behavior selection.

### Microphone, Speaker, and Light Ring

These are interaction components rather than core locomotion components.

They support:

* Voice input.
* Voice/audio output.
* Status display.
* Personality cues.
* Emotional or expressive feedback.

## Control Architecture

GrowBot should be treated as a hybrid control system.

```text
Sensors
  ├─ IMU
  ├─ Camera
  ├─ Microphone
  └─ Optional future sensors

Fast Local Control Layer
  ├─ Policy network
  ├─ Runs around 50 Hz
  ├─ Uses recent IMU history
  └─ Outputs servo commands

Slow Reasoning Layer
  ├─ LLM server
  ├─ Vision processing
  ├─ Raw IMU interpretation
  ├─ Memory
  ├─ Personality
  ├─ Code generation
  └─ High-level behavior composition

Actuators / Outputs
  ├─ Left leg servo
  ├─ Right leg servo
  ├─ Speaker
  └─ LED ring
```

## System 1: Fast Local Motor Control

The fast layer is the reflex layer.

### Purpose

* Stand.
* Balance.
* Recover from small disturbances.
* Keep the feet under the body.
* Maintain posture.
* Execute simple learned behaviors.
* Avoid dependence on slow LLM response times.

### Runtime Rate

Target rate: approximately 50 Hz.

At 50 Hz, each control step corresponds to approximately 20 ms.

### Observation Input

The notes describe using a stack of recent IMU readings.

Known values from the current notebook:

* Observation dimension: 13.
* Stack length: 5.
* Episode length: 400 steps.
* Episode duration: approximately 8 seconds at 50 Hz.

The effective policy input should therefore include short temporal context, not just a single instantaneous sensor reading.

### Policy Output

The policy outputs direct motor commands for the servos.

Likely outputs:

* Left servo position or normalized command.
* Right servo position or normalized command.

The exact output format still needs to be confirmed from implementation code.

## System 2: Slow AI Reasoning Layer

The slow layer handles interpretation, personality, high-level behavior, and creative action generation.

### Purpose

* Process camera images.
* Interpret sensor streams.
* Understand interaction context.
* Generate commands.
* Write behavior code.
* Select or blend trained policies.
* Maintain memory.
* Clean and consolidate memory.
* Give the robot a consistent personality.

### Raw IMU to LLM

A notable design choice is sending raw IMU values directly to the LLM rather than manually translating the data into high-level labels.

Instead of sending only:

```text
robot is falling forward
robot is tilted left
robot is standing
```

The system sends physical values such as:

```text
accelerometer_x
accelerometer_y
accelerometer_z
gyro_x
gyro_y
gyro_z
recent history
```

This allows the LLM to infer patterns from raw motion data, although it should not be trusted for fast closed-loop control.

### Direct Motor Code Generation

The LLM layer is allowed to generate code that produces new motor behaviors.

This layer can:

* Write small motor scripts.
* Generate motion sequences.
* Compose behavior routines.
* Call trained motor policies.
* Blend generated behaviors with learned reflexes.

This is powerful but needs strong safety constraints.

## AI Models and Tools Mentioned

| Tool / Model  | Role                                                                              |
| ------------- | --------------------------------------------------------------------------------- |
| Gemini Flash  | Fast model for image understanding and logical command generation.                |
| Claude Sonnet | Higher-quality model for complex logic, memory extraction, cleanup, and strategy. |
| Claude Haiku  | Faster model, but described as less reliable in behavior.                         |
| Mammoth       | API aggregator for switching between models cost-effectively.                     |
| Isaac Lab     | Robot learning and simulation environment.                                        |
| MuJoCo        | Robotics physics simulator option.                                                |
| Google Colab  | Cloud GPU environment for training runs.                                          |
| H100 GPU      | Used or proposed for accelerated reinforcement-learning simulation.               |

## Digital Twin and Simulation Pipeline

The robot learns movement in simulation before transferring the policy to hardware.

### Step 1: Model the Robot

Create a digital twin that captures:

* Body mass.
* Leg geometry.
* Servo locations.
* Joint limits.
* Inertia.
* Foot shape.
* Contact surfaces.
* IMU placement.
* Action limits.
* Observation vector.

For the standing-balance configuration, preserve the 64 mm refined short-leg geometry if trying to reproduce the current notes.

### Step 2: Define the Task

Initial task should be standing balance only.

Do not reopen walking or gait logic until stable standing is solved.

The notebook explicitly describes the goal as fine-tuning the standing policy for the refined short-leg baseline while avoiding gait/walking logic.

Standing policy goals:

* Preserve the clean standing basin already found on hardware.
* Improve recovery from mild forward or face-tip disturbances.
* Keep feet under the body.
* Stay simple.
* Keep the task standing-only.

### Step 3: Train in Stages

The notebook describes a staged training plan:

```text
Run name:
standing_balance_v2_refined64_forward_recovery

Warm start:
True

Warm-start checkpoint:
standing_balance_v1_30M.pkl

Planned total timesteps:
40,000,000

Stage 1:
10,000,000 steps

Stage 2:
15,000,000 steps

Stage 3:
15,000,000 steps

Stage 3 enabled:
False
```

Because Stage 3 is disabled, the active training plan appears to be Stage 1 plus Stage 2, or approximately 25 million active timesteps.

### Step 4: Colab-Safe PPO Defaults

The notebook references Colab-safe PPO defaults:

```text
TRAIN_NUM_ENVS = 2048
TRAIN_BATCH_SIZE = 1024
TRAIN_NUM_MINIBATCHES = 16
TRAIN_NUM_UPDATES_PER_BATCH = 4
TRAIN_NUM_EVALS_PER_STAGE = None
EPISODE_STEPS = 400
OBS_DIM = 13
STACK = 5
```

There is also a note that if training stalls, reduce the number of training environments to 1024.

### Step 5: Export Policy

After training, export the trained policy checkpoint.

Expected artifact:

```text
standing_balance_v2_refined64_forward_recovery.pkl
```

or similar.

### Step 6: Deploy to Hardware

The policy should run locally on the robot’s onboard processor or be converted into a lightweight runtime format.

Possible deployment formats:

* Python inference loop.
* ONNX.
* TensorFlow Lite.
* TorchScript.
* Custom NumPy implementation for a small MLP.

For a Raspberry Pi Zero 2 W, a small MLP or lightweight ONNX/TFLite deployment is likely more realistic than a large framework-heavy runtime.

## Recommended Hardware Bring-Up Sequence

Before attempting AI behavior, validate the robot as a normal embedded/robotics project.

1. Power the Pi reliably.
2. Power servos from a separate regulated rail.
3. Tie grounds together.
4. Validate battery voltage under servo load.
5. Confirm camera operation.
6. Confirm microphone input.
7. Confirm speaker output.
8. Confirm LED ring control.
9. Confirm IMU I2C communication.
10. Log raw IMU data at the target control rate.
11. Calibrate IMU offsets.
12. Verify servo command range and neutral positions.
13. Confirm servo current draw and brownout behavior.
14. Implement a dead-simple manual servo test.
15. Implement a scripted stand or pose test.
16. Only then run learned policy inference.

## Recommended Software Components

### On-Robot Software

Minimum local stack:

* Raspberry Pi OS Lite or similar Linux distribution.
* Python runtime.
* I2C support for IMU.
* Camera interface.
* Servo control library or PWM driver.
* WebSocket or HTTP client for server communication.
* Local policy inference loop.
* Safety watchdog.
* Logging.

### Server Software

Minimum server stack:

* LLM API access layer.
* Model routing layer.
* Prompt templates for sensor interpretation.
* Memory storage.
* Vision processing path.
* Code-generation sandbox.
* Robot command protocol.
* Safety filter for generated motor commands.

### Training Software

Possible stack:

* Isaac Lab for robot learning.
* MuJoCo as an alternative or complementary simulator.
* PPO for policy training.
* Google Colab or cloud GPU environment.
* Checkpointing and evaluation scripts.
* Policy export pipeline.

## Suggested Repository Structure

```text
Projects/Growbot/
├── README.md
├── BOM.md
├── VIDEO_EXTRACTION_BLUEPRINT.md
├── hardware/
│   ├── wiring.md
│   ├── power.md
│   ├── parts.md
│   └── mechanical-design.md
├── cad/
│   ├── source/
│   └── exports/
├── firmware/
│   ├── imu_reader.py
│   ├── servo_test.py
│   ├── policy_runner.py
│   └── safety_watchdog.py
├── server/
│   ├── llm_router.py
│   ├── memory.py
│   ├── prompts/
│   └── robot_api.py
├── simulation/
│   ├── isaac_lab/
│   ├── mujoco/
│   ├── assets/
│   └── environments/
├── training/
│   ├── configs/
│   ├── notebooks/
│   ├── checkpoints/
│   └── exports/
├── experiments/
│   ├── standing_balance/
│   ├── forward_recovery/
│   └── sim_to_real/
└── notes/
    ├── video_notes.md
    ├── open_questions.md
    └── implementation_log.md
```

## Safety Constraints for LLM Motor Control

Do not let the LLM directly command arbitrary servo values without constraints.

Use a safety layer that limits:

* Servo angle range.
* Servo velocity.
* Servo acceleration.
* Maximum routine duration.
* Maximum duty cycle.
* Fall-risk postures.
* Repeated impacts.
* Battery voltage thresholds.
* Servo temperature or current where measurable.

Recommended command protocol:

```json
{
  "behavior_name": "gentle_wave",
  "duration_ms": 1200,
  "commands": [
    {"t_ms": 0, "left": 0.0, "right": 0.0},
    {"t_ms": 300, "left": 0.2, "right": -0.2},
    {"t_ms": 600, "left": -0.2, "right": 0.2},
    {"t_ms": 1200, "left": 0.0, "right": 0.0}
  ]
}
```

The robot runtime should validate this command before execution.

## Memory and Dreaming Loop

GrowBot includes a memory loop so it can adapt to people and situations.

### Memory Should Store

* Interaction events.
* User preferences.
* Robot behavior outcomes.
* Motion experiments.
* Successful behaviors.
* Failed behaviors.
* Environmental observations.
* Safety incidents.
* Learned associations.

### Dreaming / Consolidation

The “dreaming” loop periodically sends accumulated memory to a stronger model for cleanup.

The model should:

* Remove duplicates.
* Resolve contradictions.
* Extract durable lessons.
* Summarize successful behaviors.
* Summarize failed behaviors.
* Update personality notes.
* Produce compact long-term memory.

## Self-Improvement / Mimic Game

The robot can improve by comparing intended motion to observed motion.

Basic loop:

1. Generate or select a target motion.
2. Execute a constrained motor routine.
3. Record IMU response.
4. Compare observed motion to intended motion.
5. Score the match.
6. Modify the behavior.
7. Try again.

This should begin with safe low-energy behaviors only.

## Predictive State Modeling / Cerebellum Idea

A key frontier idea is adding a predictive model that anticipates the next physical state before sensor feedback arrives.

The problem is sensor and actuation latency. If the robot only reacts to current sensor data, it is always reacting to the recent past.

A cerebellum-like model would predict:

* Next IMU values.
* Next body orientation.
* Momentum.
* Stability margin.
* Expected contact state.
* Expected result of motor commands.

The robot would then compare:

```text
predicted next state
vs.
actual next state
```

The prediction error becomes a learning signal.

## Minimum Reproducible Version

The first practical MVP should not attempt full personality, voice, memory, and self-improvement.

Build in this order:

1. Hardware assembly.
2. Power validation.
3. Servo test.
4. IMU logging.
5. Simple static pose.
6. Digital twin.
7. Standing-only policy.
8. Sim-to-real standing transfer.
9. Forward-recovery policy refinement.
10. LLM-generated high-level commands.
11. Memory.
12. Dreaming loop.
13. Mimic game.
14. Predictive state model.

## Current Known Gaps

The following details still need to be extracted, reverse-engineered, or designed:

* Exact body CAD files.
* Exact wiring diagram.
* Exact power regulation design.
* Exact servo PWM control method.
* Exact local policy network architecture.
* Exact observation vector definition.
* Exact action scaling.
* Exact reward function.
* Exact Isaac Lab or MuJoCo environment code.
* Exact LLM server prompts.
* Exact API protocol between robot and server.
* Exact memory schema.
* Exact safety filter for generated motor code.
* Exact trained policy checkpoint files.

## Recommended Next Documentation Tasks

1. Create `BOM.md` with links, prices, and alternates.
2. Create `wiring.md` showing Pi, IMU, servos, microphone, LEDs, speaker, and battery regulation.
3. Create `simulation/README.md` explaining the digital twin pipeline.
4. Convert the notebook notes into valid Python config or YAML.
5. Create `training/configs/standing_balance_v2_refined64_forward_recovery.yaml`.
6. Create `hardware/mechanical-design.md` documenting the 64 mm leg geometry.
7. Create `server/README.md` documenting the LLM control layer.
8. Create `safety.md` before enabling LLM-generated motor code.

## Practical First Implementation Target

The best first target is:

> A two-servo robot with a 64 mm refined leg geometry that can stand and recover from mild forward tipping using a locally running policy trained in simulation, while logging IMU data for future LLM interpretation and self-improvement.

Do not start with walking.

Do not start with open-ended LLM motor control.

Start with standing balance, forward recovery, and safe policy deployment.

# Speed Racer RL

Speed Racer RL is a reinforcement learning research project centered on a **2D top-down racing simulator written entirely in C++**. The goal of the project is to explore how a learning agent can acquire driving behavior from geometric perception and sparse rewards, rather than to build a traditional racing game.

The simulator uses **LIDAR-style raycasting** as its primary sensory input and trains a **Deep Q-Network (DQN)** using **LibTorch**, the official PyTorch C++ API. Training is performed in headless mode for efficiency, while trained agents can be replayed visually using **Raylib**.

The project prioritizes **simulation fidelity, learning stability, and reproducibility**, with an emphasis on reinforcement learning system design rather than presentation or gameplay.

---

## Project Overview

The repository builds two primary executables, each serving a distinct purpose within the reinforcement learning workflow.

### Training Executable (`racing_trainer.cpp`)

The training executable runs reinforcement learning experiments inside a headless simulation loop.

Key responsibilities include:

- Executing the simulation without rendering for maximum throughput
- Training a vanilla DQN using experience replay
- Periodically saving model checkpoints
- Exposing training hyperparameters directly in code for rapid experimentation

This executable is intended to run unattended during long training sessions.

### Replay Executable (`racing_replay.cpp`)

The replay executable loads a trained model and evaluates it in a visual environment.

Its purpose is to:

- Render the racetrack and vehicle state
- Visualize LIDAR-style perception rays in real time
- Enable qualitative analysis of learned driving behavior

Replay is strictly for inspection and analysis and does not affect training.

---

## Environment Design

The environment is a deterministic, step-based 2D racing world with a fully custom physics and progression system.

Core characteristics include:

- Pixel-based top-down racing track
- Handwritten physics model implementing:
  - Acceleration and steering dynamics
  - Friction and drag forces
  - Wall and surface collision handling
- Checkpoint-based progress measurement
- Lap counting and terminal race conditions

All track images and environment assets are stored in:

``

assets/

---

## Observation Space

At every simulation step, the agent receives a fixed-size numerical observation vector with **18 dimensions**. The observation space is designed to be compact while encoding sufficient geometric and kinematic information.

The state representation consists of:

- Normalized vehicle speed
- Sine and cosine of the vehicle heading
- Normalized world position `(x, y)`
- Thirteen LIDAR-style raycasts spanning **−90° to +90°** relative to the vehicle’s forward direction

### Obstacle Encoding

Each raycast is converted into a continuous “danger” value:

danger = 1 / ((distance / reference_distance) + 0.1)

- Values are clamped to the range `[0, 1]`
- Larger values correspond to closer obstacles

This encoding makes nearby hazards disproportionately important while preventing distant geometry from dominating the state.

---

## Action Space

The agent interacts with the environment using **seven discrete actions**:

1. Accelerate forward  
2. Reverse  
3. Steer left  
4. Steer right  
5. Accelerate while steering left  
6. Accelerate while steering right  
7. No input  

This minimal action space keeps the learning problem tractable while still allowing expressive vehicle control.

---

## Reward Design

The reward function is shaped to encourage **efficient lap completion rather than raw speed**. The agent is rewarded for forward progress and clean driving while being penalized for unsafe or stagnant behavior.

Reward components include:

- Positive reward for progress toward the next checkpoint
- Small incentive for maintaining forward speed
- Explicit rewards for checkpoint crossings and lap completion
- Terminal reward for finishing the race
- Time-based penalty to discourage stalling
- Penalties for wall collisions and driving off-track
- Anti-idle penalty to prevent degenerate stationary policies

Episodes terminate early if the vehicle becomes stuck or fails to make meaningful progress.

---

## Training Dynamics

Across long training runs, agent behavior tends to evolve in several stages:

- Early training produces slow, cautious driving with few collisions
- Intermediate policies increase speed but exhibit unstable steering
- Well-trained models achieve smooth control, consistent lap completion, and stable multi-lap performance

Checkpoint-based reward shaping plays a significant role in stabilizing learning and improving convergence.

---

## Repository Layout


.
├── assets/               # Track images and environment assets
├── sampleModels/         # Example trained neural network checkpoints
├── CMakeLists.txt
├── LICENSE
├── README.md
├── analyze_training.cpp  # Training log analysis utilities
├── dqn.h                 # Deep Q-Network implementation
├── main.cpp              # Shared simulator utilities
├── racing_replay.cpp     # Visual replay executable
├── racing_trainer.cpp    # Headless training executable
└── replay_buffer.h       # Experience replay buffer

---

## Build Requirements

The following tools and libraries are required to build the project:

- C++ compiler supporting **C++17** or newer
- **CMake 3.20+**
- **LibTorch** (PyTorch C++ API)
- **Raylib**
- MSVC on Windows or GCC/Clang on Linux or macOS
- Git

---

## Windows Setup Guide

### Visual Studio Build Tools

Install Visual Studio Build Tools with the **Desktop development with C++** workload enabled. Ensure the MSVC compiler, Windows SDK, and CMake tools are selected.

### CMake

Install CMake and add it to your system PATH. Verify installation:


cmake --version

### Git

Install Git and verify:


git --version

### LibTorch

Download the **C++ LibTorch distribution** from the PyTorch website. Do not use `pip`, as Python bindings are incompatible.

Extract LibTorch to:


C:\libtorch

Verify that the following file exists:


C:\libtorch\share\cmake\Torch\TorchConfig.cmake

### Raylib

Install Raylib using vcpkg:


git clone https://github.com/microsoft/vcpkg
cd vcpkg
bootstrap-vcpkg.bat
vcpkg install raylib:x64-windows

---

## Building the Project

Open an **x64 Native Tools Command Prompt for Visual Studio 2022** and run:


git clone --recursive https://github.com/YOUR_USERNAME/speed-racer-rl.git
cd speed-racer-rl
cmake -B build ^
-DCMAKE_PREFIX_PATH="C:\libtorch" ^
-DCMAKE_TOOLCHAIN_FILE=[path-to-vcpkg]/scripts/buildsystems/vcpkg.cmake
cmake --build build --config Release

Both the training and replay executables will be produced in the build directory.

---

## Visual Replay

To visualize a trained model:


./racing_replay sampleModels/best.pt

The replay window displays the vehicle trajectory, track boundaries, and perception rays in real time.

---

## Training a Model

To begin training:


./racing_trainer

Training runs headless and periodically saves checkpoints. Hyperparameters such as learning rate, epsilon decay, and episode length can be modified directly in `racing_trainer.cpp`.

---

## Sample Checkpoints

The `sampleModels/` directory contains example models representing various stages of learning, from cautious early policies to stable multi-lap driving behavior. These models can be used for replay, testing, and comparison.

---

## Future Directions

Potential extensions include:

- Double and Dueling DQN variants
- Policy-gradient baselines such as PPO or SAC
- Curriculum learning across increasingly complex tracks
- Domain randomization for robustness
- Multi-agent or competitive racing scenarios

---

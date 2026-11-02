# voice-controlled-robot-ros2
Offline voice-controlled TurtleBot3 in Gazebo using ROS 2 Humble and Vosk speech recognition
# Voice-Controlled Robot using ROS 2, Gazebo and Vosk

A simulated TurtleBot3 that follows spoken commands. Audio is converted to text offline using Vosk, and the text is turned into movement commands in ROS 2.

**Demo video:** YOUR_VIDEO_LINK

## How it works

```
commands.wav -> [speech_node + Vosk] -> /voice_text -> [command_node] -> /cmd_vel -> TurtleBot3 in Gazebo
```

- **speech_node** reads a recorded audio file, recognises speech with Vosk (limited to a fixed list of command words), and publishes the text on `/voice_text`.
- **command_node** subscribes to `/voice_text` and publishes velocity commands on `/cmd_vel` for words like *forward*, *back*, *left*, *right* and *stop*.

## Supported commands

| Say | Robot action |
|---|---|
| "go forward" / "forward" | Moves forward |
| "back" | Moves backward |
| "turn left" / "left" | Turns left |
| "turn right" / "right" | Turns right |
| "stop" | Stops |

## Tech stack

ROS 2 Humble, Gazebo (Classic), TurtleBot3 (burger), Vosk (small English model), Python 3.10, Ubuntu 22.04

## Setup

```bash
# Simulation packages
sudo apt install ros-humble-gazebo-ros-pkgs ros-humble-turtlebot3 \
  ros-humble-turtlebot3-gazebo ffmpeg

# Vosk and model
pip3 install vosk
wget https://alphacephei.com/vosk/models/vosk-model-small-en-us-0.15.zip
unzip vosk-model-small-en-us-0.15.zip && mv vosk-model-small-en-us-0.15 ~/vosk-model
```

Create a ROS 2 package named `voice_robot_pkg` (ament_python), place `command_node.py` and `speech_node.py` inside it, register both as `console_scripts` in `setup.py`, then build:

```bash
cd ~/ros2_ws && colcon build && source install/setup.bash
```

## Run

Record the commands, with a pause between each one, and convert the file to 16 kHz mono WAV:

```bash
ffmpeg -y -i recording.m4a -vn -ar 16000 -ac 1 -sample_fmt s16 ~/commands.wav
```

Then, in three terminals:

```bash
# Terminal 1: simulation
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py

# Terminal 2: command node
ros2 run voice_robot_pkg command_node

# Terminal 3: speech node
ros2 run voice_robot_pkg speech_node --ros-args \
  -p model_path:=$HOME/vosk-model -p wav_path:=$HOME/commands.wav
```

## Results

The robot followed all four spoken commands (go forward, turn left, turn right, stop) in the Gazebo simulation. Limiting Vosk to a fixed command list made recognition more reliable than open vocabulary, where "turn right" was sometimes misheard.

## Limitations and future work

- Input is a pre-recorded audio file, not live microphone input
- Fixed command vocabulary
- Planned: live microphone input, and speed or distance commands

## Repository contents

- `command_node.py`, `speech_node.py`: the two ROS 2 nodes
- `setup.py`, `package.xml`: package files
- Screenshots of the simulation and the ROS node graph
<img width="837" height="532" alt="Screenshot from 2026-11-02 17-26-39" src="https://github.com/user-attachments/assets/e95f0239-f417-4190-8c6c-1c29408f5909" />
<img width="799" height="550" alt="Screenshot from 2026-11-02 17-26-57" src="https://github.com/user-attachments/assets/f7b39efb-af43-4290-b629-3b18d69cd1de" />
<img width="1327" height="912" alt="Screenshot from 2026-11-02 15-30-21" src="https://github.com/user-attachments/assets/353a4e77-fc58-4cab-baac-73063e28fb2f" />

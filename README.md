# Projectile-motion-simulator
A python simulator that calculates and visualizes projectile motion - input lunch angle and speed to get max height , range, flight time, and a trajectory graph.
# 🚀 Projectile Motion Simulator

A simple Python program that simulates projectile motion based on user-input launch angle and speed. It calculates key physics values and visualizes the trajectory with a graph.

## Features

- Takes launch **angle** and **speed** as input
- Calculates:
  - **Maximum height**
  - **Distance traveled** (range)
  - **Flight time**
- Draws and saves the **trajectory graph** (`trajectory.png`)

## Example

> "How far does a rocket go if launched at 60° with 20 m/s velocity?"

```
Enter launch angle (degrees): 60
Enter launch speed (m/s): 20

--- Results ---
Maximum height:    15.29 m
Distance traveled: 35.31 m
Flight time:       3.53 s
```

## How It Works

The simulator uses standard projectile motion equations (assuming no air resistance):

- `vx = v * cos(θ)` — horizontal velocity component
- `vy = v * sin(θ)` — vertical velocity component
- `Flight time = 2 * vy / g`
- `Max height = vy² / (2g)`
- `Range = vx * flight time`

where `g = 9.81 m/s²` (acceleration due to gravity).

## Requirements

- Python 3
- `matplotlib`

Install dependencies:

```bash
pip install matplotlib
```

## Usage

```bash
python3 projectile_motion_simulator.py
```

Follow the prompts to enter a launch angle and speed. The program will print the results and display/save a trajectory graph.

## License

Feel free to use, modify, and share this project.

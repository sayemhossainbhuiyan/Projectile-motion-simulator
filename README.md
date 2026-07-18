# 🚀 Projectile Motion Simulator

An interactive, terminal-driven Python simulator that models the motion of a projectile — with real-time animated graphs, multi-planet gravity, air resistance, trajectory comparisons, and CSV data export.

Built as a physics learning tool: enter a launch angle and speed, and watch the projectile fly, then explore how height, velocity, and time all relate to one another.

---

## ✨ Features

- **🎞️ Animated trajectory** — watch the projectile move along its path in real time, not just a static plot
- **📍 Key point markers** — launch (green), maximum height (blue), and landing (red) are all labeled directly on the graph
- **📊 In-plot stats box** — angle, velocity, max height, range, and flight time displayed right on the chart
- **⚖️ Compare two launches** — plot two trajectories (e.g. 30° vs 60°) on the same graph to compare side by side
- **🌍 Multiple planets** — simulate launches on Earth, the Moon, or Mars, each with accurate gravity
- **📈 Velocity vs Time graph** — see how speed changes throughout the flight, with initial/final velocity marked
- **📈 Height vs Time graph** — see the height profile over time, with max height and landing time marked
- **💨 Air resistance (drag) toggle** — switch between ideal (no drag) physics and a realistic, numerically-integrated drag model, then compare them on one graph
- **📄 CSV export** — save every simulated time step (time, x, y, velocity) to `projectile_data.csv` for use in Excel, Google Sheets, or other tools

---

## 🖥️ Demo Output

Running the simulator produces:

1. `trajectory.png` — the animated flight path with launch/max-height/landing markers
2. `velocity_vs_time.png` — velocity over time
3. `height_vs_time.png` — height over time
4. `trajectory_comparison.png` — side-by-side comparison (compare mode or drag vs. ideal)
5. `projectile_data.csv` — full simulation data, optional

---

## 🛠️ Requirements

- Python 3.x
- [matplotlib](https://matplotlib.org/)

Install dependencies:

```bash
pip install matplotlib
```

*(`math` and `csv` are part of the Python standard library — no extra install needed.)*

---

## ▶️ Usage

Clone the repo and run the script:

```bash
git clone https://github.com/sayemhossainbhuiyan/Projectile-motion-simulator.git
cd Projectile-motion-simulator
python3 projectile_motion_simulator.py
```

You'll be guided through a few prompts:

```
🚀 Projectile Motion Simulator

Choose a planet:
  1. Earth (g = 9.81 m/s^2)
  2. Moon (g = 1.62 m/s^2)
  3. Mars (g = 3.71 m/s^2)
Enter choice (1-3): 1

What would you like to do?
  1. Simulate a single launch
  2. Compare two launches
Enter choice (1-2): 1

Enter launch angle (degrees): 45
Enter launch speed (m/s): 30
Enable air resistance? (y/n): n
```

Depending on your choices, the simulator will show an animated trajectory, a velocity-vs-time graph, a height-vs-time graph, and/or a side-by-side comparison — then optionally export the results to CSV.

---

## 🧮 The Physics

**Ideal motion (no air resistance)** uses standard kinematics (SUVAT) equations:

```
vx = v * cos(θ)              (constant throughout flight)
vy(t) = v * sin(θ) - g * t
x(t) = vx * t
y(t) = vy0 * t - 0.5 * g * t²
v(t) = sqrt(vx² + vy(t)²)
```

**With air resistance**, drag depends on the projectile's instantaneous velocity, so there's no closed-form solution. Instead, the simulator numerically integrates the motion step by step (Euler method), applying a drag force proportional to velocity squared at each tiny time step:

```
F_drag = -k * |v| * v      (opposes motion, reduces vx and vy each step)
```

This produces the expected realistic behavior: shorter range, lower max height, shorter flight time, and a steeper descent compared to the ideal case.

---

## 📂 Project Structure

```
Projectile-motion-simulator/
├── projectile_motion_simulator.py   # main script
└── README.md
```

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome. Feel free to open a pull request or file an issue.

## 📜 License

This project is open source and available for educational use. Add a license of your choice (e.g. MIT) if you plan to distribute it publicly.



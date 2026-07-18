# Projectile-motion-simulator
A python simulator that calculates and visualizes projectile motion - input lunch angle and speed to get max height , range, flight time, and a trajectory graph.
"""
Projectile Motion Simulator
----------------------------
Takes a launch angle and speed, then calculates:
    - Maximum height
    - Distance traveled (range)
    - Flight time
and draws the trajectory graph.

Example question this answers:
    "How far does a rocket go if launched at 60 degrees with X velocity?"
"""

import math
import matplotlib.pyplot as plt

g = 9.81  # acceleration due to gravity (m/s^2)


def get_float(prompt):
    """Keep asking until the user enters a valid number."""
    while True:
        try:
            return float(input(prompt))
        except ValueError:
            print("Please enter a valid number.")


def simulate_projectile(angle_deg, speed):
    """Return (max_height, range_, flight_time, xs, ys) for the given launch."""
    angle_rad = math.radians(angle_deg)
    vx = speed * math.cos(angle_rad)
    vy = speed * math.sin(angle_rad)

    flight_time = (2 * vy) / g
    max_height = (vy ** 2) / (2 * g)
    range_ = vx * flight_time

    # Build trajectory points for plotting
    num_points = 200
    xs, ys = [], []
    for i in range(num_points + 1):
        t = flight_time * i / num_points
        x = vx * t
        y = vy * t - 0.5 * g * t ** 2
        xs.append(x)
        ys.append(max(y, 0))  # don't let it dip below ground due to rounding

    return max_height, range_, flight_time, xs, ys


def plot_trajectory(xs, ys, angle_deg, speed):
    plt.figure(figsize=(8, 5))
    plt.plot(xs, ys, color="crimson", linewidth=2)
    plt.title(f"Projectile Trajectory (angle={angle_deg}°, speed={speed} m/s)")
    plt.xlabel("Horizontal Distance (m)")
    plt.ylabel("Height (m)")
    plt.grid(True, linestyle="--", alpha=0.6)
    plt.fill_between(xs, ys, color="crimson", alpha=0.1)
    plt.tight_layout()
    plt.savefig("trajectory.png", dpi=150)
    print("\nTrajectory graph saved as 'trajectory.png'")
    plt.show()


def main():
    print("🚀 Projectile Motion Simulator\n")
    angle = get_float("Enter launch angle (degrees): ")
    speed = get_float("Enter launch speed (m/s): ")

    max_height, range_, flight_time, xs, ys = simulate_projectile(angle, speed)

    print("\n--- Results ---")
    print(f"Maximum height:   {max_height:.2f} m")
    print(f"Distance traveled: {range_:.2f} m")
    print(f"Flight time:      {flight_time:.2f} s")

    plot_trajectory(xs, ys, angle, speed)


if __name__ == "__main__":
    main()

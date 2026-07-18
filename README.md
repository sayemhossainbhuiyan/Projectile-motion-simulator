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
import csv
import matplotlib.pyplot as plt
import matplotlib.animation as animation

PLANETS = {
    "1": ("Earth", 9.81),
    "2": ("Moon", 1.62),
    "3": ("Mars", 3.71),
}


def choose_planet():
    """Ask the user to pick a planet and return (name, gravity_value)."""
    print("Choose a planet:")
    for key, (name, g_value) in PLANETS.items():
        print(f"  {key}. {name} (g = {g_value} m/s^2)")

    while True:
        choice = input("Enter choice (1-3): ").strip()
        if choice in PLANETS:
            return PLANETS[choice]
        print("Please enter 1, 2, or 3.")


def get_float(prompt):
    """Keep asking until the user enters a valid number."""
    while True:
        try:
            return float(input(prompt))
        except ValueError:
            print("Please enter a valid number.")


def simulate_projectile(angle_deg, speed, g):
    """Return (max_height, range_, flight_time, xs, ys, times, velocities) for the given launch."""
    angle_rad = math.radians(angle_deg)
    vx = speed * math.cos(angle_rad)
    initial_vy = speed * math.sin(angle_rad)

    flight_time = (2 * initial_vy) / g
    max_height = (initial_vy ** 2) / (2 * g)
    range_ = vx * flight_time

    # Build trajectory points, plus velocity-at-each-time-step, in the same loop
    num_points = 200
    xs, ys = [], []
    times, velocities = [], []
    for i in range(num_points + 1):
        t = flight_time * i / num_points
        x = vx * t
        y = initial_vy * t - 0.5 * g * t ** 2
        xs.append(x)
        ys.append(max(y, 0))  # don't let it dip below ground due to rounding

        vy = initial_vy - g * t
        v = math.sqrt(vx ** 2 + vy ** 2)
        times.append(t)
        velocities.append(v)

    return max_height, range_, flight_time, xs, ys, times, velocities


def simulate_projectile_with_drag(angle_deg, speed, g, drag_coefficient=0.02, dt=0.01):
    """Step-by-step (Euler) simulation with air resistance.

    Air resistance is modeled as a drag force opposing velocity, proportional
    to speed squared: F_drag = -drag_coefficient * |v| * v (per unit mass).
    Because drag depends on velocity at each instant, this can't be solved
    with the closed-form SUVAT equations, so we integrate it numerically
    step by step instead.

    Returns (max_height, range_, flight_time, xs, ys, times, velocities).
    """
    angle_rad = math.radians(angle_deg)
    vx = speed * math.cos(angle_rad)
    vy = speed * math.sin(angle_rad)
    x, y, t = 0.0, 0.0, 0.0

    xs, ys, times, velocities = [x], [y], [t], [speed]

    while True:
        current_speed = math.sqrt(vx ** 2 + vy ** 2)

        ax = -drag_coefficient * current_speed * vx
        ay = -g - drag_coefficient * current_speed * vy

        vx += ax * dt
        vy += ay * dt
        x += vx * dt
        y += vy * dt
        t += dt

        if y < 0:
            y = 0  # clamp to ground
            xs.append(x)
            ys.append(y)
            times.append(t)
            velocities.append(math.sqrt(vx ** 2 + vy ** 2))
            break

        xs.append(x)
        ys.append(y)
        times.append(t)
        velocities.append(math.sqrt(vx ** 2 + vy ** 2))

    max_height = max(ys)
    range_ = xs[-1]
    flight_time = times[-1]

    return max_height, range_, flight_time, xs, ys, times, velocities


def animate_trajectory(xs, ys, angle_deg, speed, range_, flight_time, planet_name="Earth"):
    """Animate the projectile moving along its trajectory, then save a final image."""
    fig, ax = plt.subplots(figsize=(8, 5))
    ax.set_title(f"Projectile Trajectory on {planet_name} (angle={angle_deg}°, speed={speed} m/s)")
    ax.set_xlabel("Horizontal Distance (m)")
    ax.set_ylabel("Height (m)")
    ax.grid(True, linestyle="--", alpha=0.6)
    ax.set_xlim(0, max(xs) * 1.05 if max(xs) > 0 else 1)
    ax.set_ylim(0, max(ys) * 1.2 if max(ys) > 0 else 1)

    # Line for the trail already traveled, and a dot for the current position
    trail_line, = ax.plot([], [], color="crimson", linewidth=2)
    point, = ax.plot([], [], marker="o", color="crimson", markersize=8)

    # Launch point
    ax.scatter(xs[0], ys[0], color="green", s=80, label="Launch")
    ax.text(xs[0], ys[0] + 2, "Launch", color="green")

    # Highest point
    max_index = ys.index(max(ys))
    ax.scatter(xs[max_index], ys[max_index], color="blue", s=80, label="Max Height")
    ax.text(xs[max_index], ys[max_index] + 2,
            f"Max Height\n{max(ys):.2f} m",
            color="blue", ha="center")

    # Landing point
    ax.scatter(xs[-1], ys[-1], color="red", s=80, label="Landing")
    ax.text(xs[-1], ys[-1] + 2, "Landing", color="red", ha="center")

    ax.legend()

    # Text box with launch parameters and results, drawn inside the plot
    stats_text = (
        f"Angle: {angle_deg}°\n"
        f"Velocity: {speed} m/s\n\n"
        f"Max Height: {max(ys):.2f} m\n"
        f"Range: {range_:.2f} m\n"
        f"Flight Time: {flight_time:.2f} s"
    )
    ax.text(
        0.02, 0.98, stats_text,
        transform=ax.transAxes,
        fontsize=9,
        verticalalignment="top",
        bbox=dict(boxstyle="round", facecolor="white", edgecolor="gray", alpha=0.85)
    )

    def init():
        trail_line.set_data([], [])
        point.set_data([], [])
        return trail_line, point

    def update(frame):
        trail_line.set_data(xs[:frame + 1], ys[:frame + 1])
        point.set_data([xs[frame]], [ys[frame]])
        return trail_line, point

    anim = animation.FuncAnimation(
        fig, update, frames=len(xs), init_func=init,
        interval=20, blit=True, repeat=False
    )

    plt.tight_layout()
    plt.show()

    # Save a static image of the completed trajectory as well
    ax.fill_between(xs, ys, color="crimson", alpha=0.1)
    fig.savefig("trajectory.png", dpi=150)
    print("\nTrajectory graph saved as 'trajectory.png'")


def compare_trajectories(traj1, traj2, planet_name):
    """Animate two trajectories on the same graph for comparison.

    traj1 and traj2 are each a dict with keys:
        angle, speed, max_height, range_, flight_time, xs, ys, color, label
    """
    fig, ax = plt.subplots(figsize=(8, 5))
    ax.set_title(f"Trajectory Comparison on {planet_name}")
    ax.set_xlabel("Horizontal Distance (m)")
    ax.set_ylabel("Height (m)")
    ax.grid(True, linestyle="--", alpha=0.6)

    max_x = max(max(traj1["xs"]), max(traj2["xs"]))
    max_y = max(max(traj1["ys"]), max(traj2["ys"]))
    ax.set_xlim(0, max_x * 1.05 if max_x > 0 else 1)
    ax.set_ylim(0, max_y * 1.2 if max_y > 0 else 1)

    trails = {}
    points = {}
    for traj in (traj1, traj2):
        trail_line, = ax.plot([], [], color=traj["color"], linewidth=2, label=traj["label"])
        point, = ax.plot([], [], marker="o", color=traj["color"], markersize=8)
        trails[traj["label"]] = trail_line
        points[traj["label"]] = point

    ax.legend()

    stats_text = (
        f"{traj1['label']}: Range {traj1['range_']:.2f} m, "
        f"Max Height {traj1['max_height']:.2f} m, Time {traj1['flight_time']:.2f} s\n"
        f"{traj2['label']}: Range {traj2['range_']:.2f} m, "
        f"Max Height {traj2['max_height']:.2f} m, Time {traj2['flight_time']:.2f} s"
    )
    ax.text(
        0.02, 0.98, stats_text,
        transform=ax.transAxes,
        fontsize=8,
        verticalalignment="top",
        bbox=dict(boxstyle="round", facecolor="white", edgecolor="gray", alpha=0.85)
    )

    total_frames = max(len(traj1["xs"]), len(traj2["xs"]))

    def init():
        for line in trails.values():
            line.set_data([], [])
        for point in points.values():
            point.set_data([], [])
        return list(trails.values()) + list(points.values())

    def update(frame):
        for traj in (traj1, traj2):
            idx = min(frame, len(traj["xs"]) - 1)
            trails[traj["label"]].set_data(traj["xs"][:idx + 1], traj["ys"][:idx + 1])
            points[traj["label"]].set_data([traj["xs"][idx]], [traj["ys"][idx]])
        return list(trails.values()) + list(points.values())

    anim = animation.FuncAnimation(
        fig, update, frames=total_frames, init_func=init,
        interval=20, blit=True, repeat=False
    )

    plt.tight_layout()
    plt.show()

    fig.savefig("trajectory_comparison.png", dpi=150)
    print("\nComparison graph saved as 'trajectory_comparison.png'")


def plot_velocity_time(times, velocities, planet_name):
    """Plot velocity vs time, marking the initial and final velocity."""
    fig, ax = plt.subplots(figsize=(8, 5))
    ax.plot(times, velocities, color="blue", linewidth=2)
    ax.set_title("Velocity vs Time")
    ax.set_xlabel("Time (seconds)")
    ax.set_ylabel("Velocity (m/s)")
    ax.grid(True, linestyle="--", alpha=0.6)

    # Mark initial velocity
    ax.scatter(times[0], velocities[0], color="green", s=80, label="Initial Velocity")
    ax.text(times[0], velocities[0] + 1,
            f"Initial: {velocities[0]:.2f} m/s", color="green")

    # Mark final velocity (just before landing)
    ax.scatter(times[-1], velocities[-1], color="red", s=80, label="Final Velocity")
    ax.text(times[-1], velocities[-1] + 1,
            f"Final: {velocities[-1]:.2f} m/s", color="red", ha="right")

    ax.legend()
    plt.tight_layout()
    plt.show()

    fig.savefig("velocity_vs_time.png", dpi=150)
    print("Velocity vs Time graph saved as 'velocity_vs_time.png'")
    print(f"(Simulated on {planet_name})")


def plot_height_time(times, ys, planet_name):
    """Plot height vs time, marking the maximum height and the landing point."""
    fig, ax = plt.subplots(figsize=(8, 5))
    ax.plot(times, ys, color="blue", linewidth=2, label="Height")
    ax.set_title("Height vs Time")
    ax.set_xlabel("Time (seconds)")
    ax.set_ylabel("Height (meters)")
    ax.grid(True, linestyle="--", alpha=0.6)

    # Mark maximum height
    max_index = ys.index(max(ys))
    ax.scatter(times[max_index], ys[max_index], color="green", s=80, label="Max Height")
    ax.text(times[max_index], ys[max_index] + 1,
            f"Max Height: {ys[max_index]:.2f} m", color="green", ha="center")

    # Mark landing point
    ax.scatter(times[-1], ys[-1], color="red", s=80, label="Landing")
    ax.text(times[-1], ys[-1] + 1,
            f"Landing: t={times[-1]:.2f} s", color="red", ha="right")

    ax.legend()
    plt.tight_layout()
    plt.show()

    fig.savefig("height_vs_time.png", dpi=150)
    print("Height vs Time graph saved as 'height_vs_time.png'")
    print(f"(Simulated on {planet_name})")


def export_to_csv(times, xs, ys, velocities, filename="projectile_data.csv"):
    """Save one row per time step with Time, X, Y, and Velocity to a CSV file."""
    with open(filename, "w", newline="") as f:
        writer = csv.writer(f)
        writer.writerow(["Time (s)", "X Position (m)", "Y Position (m)", "Velocity (m/s)"])
        for t, x, y, v in zip(times, xs, ys, velocities):
            writer.writerow([f"{t:.4f}", f"{x:.4f}", f"{y:.4f}", f"{v:.4f}"])

    print(f"Simulation data exported successfully to {filename}.")


def main():
    print("🚀 Projectile Motion Simulator\n")

    planet_name, g = choose_planet()

    print("\nWhat would you like to do?")
    print("  1. Simulate a single launch")
    print("  2. Compare two launches")
    mode = input("Enter choice (1-2): ").strip()

    if mode == "2":
        print(f"\n-- Launch 1 (blue) on {planet_name} --")
        angle1 = get_float("Enter launch angle (degrees): ")
        speed1 = get_float("Enter launch speed (m/s): ")
        max_height1, range_1, flight_time1, xs1, ys1, times1, velocities1 = simulate_projectile(angle1, speed1, g)

        print(f"\n-- Launch 2 (red) on {planet_name} --")
        angle2 = get_float("Enter launch angle (degrees): ")
        speed2 = get_float("Enter launch speed (m/s): ")
        max_height2, range_2, flight_time2, xs2, ys2, times2, velocities2 = simulate_projectile(angle2, speed2, g)

        traj1 = {
            "angle": angle1, "speed": speed1, "max_height": max_height1,
            "range_": range_1, "flight_time": flight_time1,
            "xs": xs1, "ys": ys1, "color": "blue", "label": f"{angle1}°",
        }
        traj2 = {
            "angle": angle2, "speed": speed2, "max_height": max_height2,
            "range_": range_2, "flight_time": flight_time2,
            "xs": xs2, "ys": ys2, "color": "red", "label": f"{angle2}°",
        }

        print("\n--- Results ---")
        print(f"Launch 1 ({angle1}°): Max height {max_height1:.2f} m, "
              f"Range {range_1:.2f} m, Flight time {flight_time1:.2f} s")
        print(f"Launch 2 ({angle2}°): Max height {max_height2:.2f} m, "
              f"Range {range_2:.2f} m, Flight time {flight_time2:.2f} s")

        compare_trajectories(traj1, traj2, planet_name)
    else:
        angle = get_float("Enter launch angle (degrees): ")
        speed = get_float("Enter launch speed (m/s): ")

        drag_choice = input("Enable air resistance? (y/n): ").strip().lower()

        if drag_choice == "y":
            max_height, range_, flight_time, xs, ys, times, velocities = \
                simulate_projectile_with_drag(angle, speed, g)
            ideal_max_height, ideal_range, ideal_flight_time, ideal_xs, ideal_ys, _, _ = \
                simulate_projectile(angle, speed, g)

            print("\n--- Results (with air resistance) ---")
            print(f"Maximum height:   {max_height:.2f} m")
            print(f"Distance traveled: {range_:.2f} m")
            print(f"Flight time:      {flight_time:.2f} s")

            print("\n--- Results (ideal, no air resistance) ---")
            print(f"Maximum height:   {ideal_max_height:.2f} m")
            print(f"Distance traveled: {ideal_range:.2f} m")
            print(f"Flight time:      {ideal_flight_time:.2f} s")

            ideal_traj = {
                "angle": angle, "speed": speed, "max_height": ideal_max_height,
                "range_": ideal_range, "flight_time": ideal_flight_time,
                "xs": ideal_xs, "ys": ideal_ys, "color": "blue", "label": "Ideal (no drag)",
            }
            drag_traj = {
                "angle": angle, "speed": speed, "max_height": max_height,
                "range_": range_, "flight_time": flight_time,
                "xs": xs, "ys": ys, "color": "red", "label": "With Air Resistance",
            }
            compare_trajectories(ideal_traj, drag_traj, planet_name)
            plot_velocity_time(times, velocities, planet_name)
            plot_height_time(times, ys, planet_name)
        else:
            max_height, range_, flight_time, xs, ys, times, velocities = simulate_projectile(angle, speed, g)

            print("\n--- Results ---")
            print(f"Maximum height:   {max_height:.2f} m")
            print(f"Distance traveled: {range_:.2f} m")
            print(f"Flight time:      {flight_time:.2f} s")

            animate_trajectory(xs, ys, angle, speed, range_, flight_time, planet_name)
            plot_velocity_time(times, velocities, planet_name)
            plot_height_time(times, ys, planet_name)

        save_choice = input("\nSave simulation data to CSV? (y/n): ").strip().lower()
        if save_choice == "y":
            export_to_csv(times, xs, ys, velocities)


if __name__ == "__main__":
    main()


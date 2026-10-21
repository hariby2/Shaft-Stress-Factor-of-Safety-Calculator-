import math
import matplotlib.pyplot as plt


def calculate_shaft_stress(
    torque_nm,
    bending_moment_nm,
    diameter_mm,
    yield_strength_mpa,
):
    """Calculate nominal stresses for a solid circular shaft."""

    if diameter_mm <= 0:
        raise ValueError("Diameter must be positive.")
    if yield_strength_mpa <= 0:
        raise ValueError("Yield strength must be positive.")
    if torque_nm < 0 or bending_moment_nm < 0:
        raise ValueError("Load magnitudes cannot be negative.")

    # Convert units to SI.
    diameter = diameter_mm / 1000
    torque = torque_nm
    bending_moment = bending_moment_nm

    # Solid circular shaft equations.
    bending_stress = (
        32 * bending_moment
        / (math.pi * diameter**3)
    )

    shear_stress = (
        16 * torque
        / (math.pi * diameter**3)
    )

    # von Mises stress for combined bending and torsion.
    von_mises = math.sqrt(
        bending_stress**2 + 3 * shear_stress**2
    )

    # Stresses in Pa; convert to MPa.
    bending_mpa = bending_stress / 1e6
    shear_mpa = shear_stress / 1e6
    von_mises_mpa = von_mises / 1e6

    factor_of_safety = yield_strength_mpa / von_mises_mpa

    return {
        "bending_stress_mpa": bending_mpa,
        "shear_stress_mpa": shear_mpa,
        "von_mises_mpa": von_mises_mpa,
        "factor_of_safety": factor_of_safety,
    }


def main():
    print("=== Shaft Stress Calculator ===")

    torque = float(input("Torque (N.m): "))
    moment = float(input("Bending moment (N.m): "))
    diameter = float(input("Shaft diameter (mm): "))
    yield_strength = float(input("Yield strength (MPa): "))

    try:
        result = calculate_shaft_stress(
            torque, moment, diameter, yield_strength
        )
    except ValueError as error:
        print("Input error:", error)
        return

    print("\n--- Results ---")
    print(f"Bending stress: "
          f"{result['bending_stress_mpa']:.2f} MPa")
    print(f"Torsional shear stress: "
          f"{result['shear_stress_mpa']:.2f} MPa")
    print(f"von Mises stress: "
          f"{result['von_mises_mpa']:.2f} MPa")
    print(f"Factor of safety: "
          f"{result['factor_of_safety']:.2f}")

    if result["factor_of_safety"] < 1:
        print("Warning: nominal stress exceeds yield strength.")
    else:
        print("Nominal stress is below yield strength.")

    labels = ["Bending", "Shear", "von Mises"]
    values = [
        result["bending_stress_mpa"],
        result["shear_stress_mpa"],
        result["von_mises_mpa"],
    ]

    plt.bar(labels, values, color=["royalblue", "orange", "green"])
    plt.ylabel("Stress (MPa)")
    plt.title("Shaft Stress Analysis")
    plt.tight_layout()
    plt.show()


if __name__ == "__main__":
    main()

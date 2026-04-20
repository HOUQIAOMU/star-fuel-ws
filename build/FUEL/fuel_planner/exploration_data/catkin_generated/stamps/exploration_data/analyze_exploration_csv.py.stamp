#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import argparse
import csv
import os
from dataclasses import dataclass
from pathlib import Path
from typing import List, Tuple

import matplotlib

matplotlib.use("Agg")
import matplotlib.pyplot as plt


@dataclass
class RunData:
    name: str
    times: List[float]
    areas: List[float]

    @property
    def final_time(self) -> float:
        return self.times[-1] if self.times else 0.0

    @property
    def final_area(self) -> float:
        return self.areas[-1] if self.areas else 0.0


def _parse_stem(stem: str) -> Tuple[int, int]:
    if "_" in stem:
        main_part, suffix = stem.split("_", 1)
    else:
        main_part, suffix = stem, "0"

    try:
        main_key = int(main_part)
    except ValueError:
        main_key = 0
    try:
        suffix_key = int(suffix)
    except ValueError:
        suffix_key = 0
    return main_key, suffix_key


def load_csv(file_path: Path) -> RunData:
    times: List[float] = []
    areas: List[float] = []
    with file_path.open("r", newline="") as f:
        reader = csv.DictReader(f)
        required = {"elapsed_time_sec", "coverage_area_m2"}
        if reader.fieldnames is None or not required.issubset(set(reader.fieldnames)):
            raise ValueError(f"CSV format invalid: {file_path}")

        for row in reader:
            times.append(float(row["elapsed_time_sec"]))
            areas.append(float(row["coverage_area_m2"]))
    return RunData(name=file_path.stem, times=times, areas=areas)


def plot_group_curves(runs: List[RunData], title: str, output_path: Path) -> None:
    plt.figure(figsize=(10, 6), dpi=180)
    for run in runs:
        plt.plot(run.times, run.areas, linewidth=1.8, label=run.name)
    plt.title(title)
    plt.xlabel("Time (s)")
    plt.ylabel("Coverage Area (m^2)")
    plt.grid(True, alpha=0.3)
    plt.legend(fontsize=8, ncol=2)
    plt.tight_layout()
    plt.savefig(output_path)
    plt.close()


def plot_statistics(first_runs: List[RunData], last_runs: List[RunData], output_path: Path) -> None:
    first_final_area = [r.final_area for r in first_runs]
    last_final_area = [r.final_area for r in last_runs]
    first_final_time = [r.final_time for r in first_runs]
    last_final_time = [r.final_time for r in last_runs]

    fig, axes = plt.subplots(1, 2, figsize=(12, 5), dpi=180)

    axes[0].boxplot(
        [first_final_area, last_final_area],
        labels=["First 5", "Last 5"],
        patch_artist=True,
        boxprops={"facecolor": "#CFE8FF"},
    )
    axes[0].set_title("Final Coverage Area Distribution")
    axes[0].set_ylabel("Area (m^2)")
    axes[0].grid(True, axis="y", alpha=0.3)

    axes[1].boxplot(
        [first_final_time, last_final_time],
        labels=["First 5", "Last 5"],
        patch_artist=True,
        boxprops={"facecolor": "#FFD8B1"},
    )
    axes[1].set_title("Final Time Distribution")
    axes[1].set_ylabel("Time (s)")
    axes[1].grid(True, axis="y", alpha=0.3)

    fig.tight_layout()
    fig.savefig(output_path)
    plt.close(fig)


def main() -> None:
    parser = argparse.ArgumentParser(description="Analyze exploration CSV runs.")
    parser.add_argument(
        "--input-dir",
        default=str(Path(__file__).resolve().parent.parent / "resources"),
        help="Directory containing exploration CSV files.",
    )
    parser.add_argument(
        "--output-dir",
        default="",
        help="Directory to store analysis images. Defaults to input dir.",
    )
    args = parser.parse_args()

    input_dir = Path(args.input_dir).resolve()
    output_dir = Path(args.output_dir).resolve() if args.output_dir else input_dir
    output_dir.mkdir(parents=True, exist_ok=True)

    csv_files = sorted(input_dir.glob("*.csv"), key=lambda p: _parse_stem(p.stem))
    if len(csv_files) < 10:
        raise RuntimeError(f"Need at least 10 CSV files, found {len(csv_files)} in {input_dir}")

    runs = [load_csv(path) for path in csv_files]
    first_five = runs[:5]
    last_five = runs[-5:]

    plot_group_curves(
        first_five,
        "Exploration Curves (First 5 Runs)",
        output_dir / "analysis_first5_curves.png",
    )
    plot_group_curves(
        last_five,
        "Exploration Curves (Last 5 Runs)",
        output_dir / "analysis_last5_curves.png",
    )
    plot_statistics(first_five, last_five, output_dir / "analysis_boxplots.png")

    print("Saved:")
    print(output_dir / "analysis_first5_curves.png")
    print(output_dir / "analysis_last5_curves.png")
    print(output_dir / "analysis_boxplots.png")


if __name__ == "__main__":
    main()

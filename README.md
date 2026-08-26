# Brake Pad Design Optimization Project

This project leverages Design of Experiments (DoE) and Latin Hypercube Sampling to optimize the performance parameters of brake pads under varied material compositions, specifically evaluating 5%, 10%, and 15% Cocopeat mixtures. The experimental data tracks structural performance metrics across different geometric and force configurations to optimizing torque delivery.

## Project Structure

  * `BrakePadOptm.ipynb`: Core Jupyter notebook containing data analysis, optimization algorithms, and performance modeling.
  * `FivePercentCocopeat.csv`: Design points and finite element analysis results for a 5% Cocopeat brake pad composition.
  * `TenPercentCocopeat.csv`: Design points and finite element analysis results for a 10% Cocopeat brake pad composition.
  * `FifteenPercentCocopeat.csv`: Design points and finite element analysis results for a 15% Cocopeat brake pad composition.

## Experimental Design & Parameters

The datasets are generated using a Central Composite Design (CCD) framework with randomized Latin Hypercube Sampling. Each composition contains 15 unique design points mapping input variables to structural performance responses:

### Input Parameters (Design Variables)

  * **P1 - Inner Diameter:** Outer boundary constraint for the internal ring geometry.
  * **P7 - Joint Force Magnitude (N):** Operational loading force applied to the brake pad assembly.
  * **P9 - Parameter 1 (degree):** Angular orientation/geometric parameter.
  * **P10 - Parameter 2 (radian):** Rotational geometry or directional parameter.
  * **P11, P14, P15:** ANSYS Parametric Design Language (APDL) Command Arguments mapping internal logic.

### Output Responses (Performance Metrics)

  * **P5 - Equivalent Stress Maximum (MPa):** Peak Von-Mises stress experienced by the component.
  * **P6 - Equivalent Plastic Strain Maximum (mm/mm):** Measurement of permanent structural deformation.
  * **P16 - my\_Torque:** Resulting braking torque generation.

## Key Composition Insights

### 5% Cocopeat Mixture

  * Characterized by balanced stress distributions with a maximum recorded stress of approximately 5.43 MPa.
  * Demonstrates low but present plastic deformation under higher load profiles, peaking around 0.00545 mm/mm.
  * Highest torque achieved: 1247.83 N-m (Design Point 5).

### 10% Cocopeat Mixture

  * Shows increased structural stiffness with higher peak stress thresholds (up to 7.93 MPa under intense load profiles).
  * Successfully maintains 0 mm/mm equivalent plastic strain across all 15 evaluated design points, indicating strong elastic recovery properties.
  * Highest torque achieved: 1320.13 N-m (Design Point 9).

### 15% Cocopeat Mixture

  * Exhibits higher compliance variations, with maximum stress levels hitting up to 13.95 MPa in specific configurations.
  * Like the 10% mixture, it shows excellent macro-structural resistance to permanent deformation with 0 mm/mm plastic strain across the matrix.
  * Highest torque achieved: 1175.01 N-m (Design Point 5).

Would you like me to generate a summary of the best-performing design configurations for each dataset to include in an "Optimal Results" section?



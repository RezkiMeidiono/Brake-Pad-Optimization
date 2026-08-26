# Finding the Maximum Braking Torque with Continuous Features

The notebook trains `best_gpr`, a Gaussian Process Regression model that predicts braking torque from four design features:

- `percentage`: Cocopeat percentage
- `InnerDiameter`: Inner diameter
- `ActuationForce`: Actuation force
- `SweepAngle`: Sweep angle

The model is a surrogate for the FEA results. After it has been fitted, it can estimate torque for feature values that were not explicitly present in the CSV files. This makes continuous optimization possible.

## Important distinction: plotting versus optimization

The contour and sensitivity-analysis cells use `np.linspace` to create a finite set of candidate values. For example:

```python
x_line = np.linspace(X[feature].min(), X[feature].max(), 100)
```

The largest value found on that line is only the largest value among those 100 samples. It is not necessarily the maximum between two samples.

To search the continuous design space, define an objective function that returns the negative predicted torque and minimize it. Minimizing the negative is equivalent to maximizing the torque.

## Continuous optimization code

Run this cell after the cells that create `X`, `scaler_X`, and `best_gpr`:

```python
from scipy.optimize import differential_evolution

# Keep the search inside the range represented by the training data.
feature_bounds = [
    (X["percentage"].min(), X["percentage"].max()),
    (X["InnerDiameter"].min(), X["InnerDiameter"].max()),
    (X["ActuationForce"].min(), X["ActuationForce"].max()),
    (X["SweepAngle"].min(), X["SweepAngle"].max()),
]


def negative_predicted_torque(feature_values):
    candidate = pd.DataFrame(
        [feature_values],
        columns=["percentage", "InnerDiameter", "ActuationForce", "SweepAngle"],
    )
    candidate_scaled = scaler_X.transform(candidate)
    predicted_torque = best_gpr.predict(candidate_scaled)[0]
    return -predicted_torque


optimization_result = differential_evolution(
    negative_predicted_torque,
    bounds=feature_bounds,
    seed=42,
    polish=True,
)

optimal_features = pd.Series(
    optimization_result.x,
    index=["percentage", "InnerDiameter", "ActuationForce", "SweepAngle"],
)
maximum_torque = -optimization_result.fun

print("Continuous optimum:")
print(optimal_features)
print(f"Predicted maximum braking torque: {maximum_torque:.2f} N-m")
print(f"Optimization successful: {optimization_result.success}")
```

## How the code works

1. `feature_bounds` limits every feature to its observed minimum and maximum. This avoids asking the surrogate model to extrapolate far outside the available data.
2. `negative_predicted_torque` converts the optimizer's numeric input into a one-row `DataFrame` with the same feature order used during training.
3. `scaler_X.transform` applies the already-fitted feature scaling. The optimizer must use the same scaler as the GPR model.
4. `best_gpr.predict` estimates the braking torque for the continuous candidate point.
5. `differential_evolution` searches the whole bounded four-dimensional region without requiring gradients. `polish=True` performs a local refinement near the best candidate.
6. The negative objective is converted back with `-optimization_result.fun`, giving the predicted maximum torque.

## Checking prediction uncertainty

A high predicted torque is more trustworthy when the GPR uncertainty is also low. The following optional check reports the model's standard deviation at the proposed optimum:

```python
optimal_scaled = scaler_X.transform(optimal_features.to_frame().T)
optimal_mean, optimal_std = best_gpr.predict(optimal_scaled, return_std=True)

print(f"Predicted torque: {optimal_mean[0]:.2f} N-m")
print(f"Prediction standard deviation: {optimal_std[0]:.2f} N-m")
print(
    f"Approximate 95% interval: "
    f"{optimal_mean[0] - 1.96 * optimal_std[0]:.2f} to "
    f"{optimal_mean[0] + 1.96 * optimal_std[0]:.2f} N-m"
)
```

The result is the maximum predicted torque of the surrogate model within the selected bounds, not a replacement for physical or FEA validation. The final feature combination should be validated with a new simulation or experiment.

## Installation

If SciPy is not available in the notebook environment, install it once before running the optimization cell:

```python
%pip install scipy
```

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

The notebook runs this cell after the cells that create `X`, `scaler_X`, `best_gpr`, and `features`:

```python
from scipy.optimize import differential_evolution

features = ['percentage', 'InnerDiameter', 'ActuationForce', 'SweepAngle']
feature_units = {
    'percentage': '%',
    'InnerDiameter': 'mm',
    'ActuationForce': 'N',
    'SweepAngle': 'degree',
}

# Keep the search inside the observed design space.
bounds = [(X[feature].min(), X[feature].max()) for feature in features]


def objective(values):
    candidate = pd.DataFrame([values], columns=features)
    candidate_scaled = scaler_X.transform(candidate)
    predicted_torque = best_gpr.predict(candidate_scaled)[0]
    return -predicted_torque  # differential_evolution minimizes


optimization = differential_evolution(
    objective,
    bounds=bounds,
    seed=42,
    polish=True,
)

# Predict torque and uncertainty at the best continuous candidate.
optimal_candidate = pd.DataFrame([optimization.x], columns=features)
optimal_candidate_scaled = scaler_X.transform(optimal_candidate)
torque_mean, torque_std = best_gpr.predict(
    optimal_candidate_scaled,
    return_std=True,
)

optimal_torque = float(torque_mean[0])
uncertainty = float(torque_std[0])
torque_lower = optimal_torque - 1.96 * uncertainty
torque_upper = optimal_torque + 1.96 * uncertainty
continuous_optimum = dict(zip(features, optimization.x))
```

## How the code works

1. `bounds` limits every feature to its observed minimum and maximum. This avoids asking the surrogate model to extrapolate far outside the available data.
2. `objective` converts the optimizer's numeric input into a one-row `DataFrame` with the same feature order used during training.
3. `scaler_X.transform` applies the already-fitted feature scaling. The optimizer must use the same scaler as the GPR model.
4. `best_gpr.predict` estimates the braking torque for the continuous candidate point.
5. `differential_evolution` searches the whole bounded four-dimensional region without requiring gradients. `polish=True` performs a local refinement near the best candidate.
6. The optimized feature values are evaluated again with `return_std=True` so the result includes prediction uncertainty.

## Displaying the results

The notebook separates the recommended design from the model diagnostics. This makes it easier to read the feature settings without confusing their physical bounds with the torque confidence interval.

```python
optimal_design = pd.DataFrame({
    'Parameter': features,
    'Unit': [feature_units[feature] for feature in features],
    'Lower Bound': [bound[0] for bound in bounds],
    'Optimal Value': [continuous_optimum[feature] for feature in features],
    'Upper Bound': [bound[1] for bound in bounds],
})

optimization_summary = pd.DataFrame({
    'Metric': [
        'Predicted braking torque',
        'Prediction standard deviation',
        'Approximate 95% lower limit',
        'Approximate 95% upper limit',
        'Optimization successful',
    ],
    'Value': [
        f'{optimal_torque:.3f}',
        f'{uncertainty:.3f}',
        f'{torque_lower:.3f}',
        f'{torque_upper:.3f}',
        'Yes' if optimization.success else 'No',
    ],
    'Unit': ['N-m', 'N-m', 'N-m', 'N-m', ''],
})

print('Continuous optimum design')
display(optimal_design.round(3).reset_index(drop=True))

print('Optimization result')
display(optimization_summary)
```

The design table reports each feature's observed range and the continuous value selected by the optimizer. The result table reports the predicted torque, its standard deviation, an approximate 95% interval, and whether the optimizer completed successfully. The notebook uses `round(3)` and formatted strings for readable output without requiring pandas' optional styling dependencies.

The result is the maximum predicted torque of the surrogate model within the selected bounds, not a replacement for physical or FEA validation. The final feature combination should be validated with a new simulation or experiment.

## Installation

If SciPy is not available in the notebook environment, install it once before running the optimization cell:

```python
%pip install scipy
```

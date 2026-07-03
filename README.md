# Prostate MRI Shimming

A MATLAB toolkit for correcting B0 field inhomogeneity in prostate MRI using custom shim coils. Developed during a research internship at Cedars-Sinai Medical Center.

## Background

Rectal air around the prostate creates off-resonance B0 field distortions that degrade MRI image quality and hamper prostate cancer identification. This toolkit designs and optimizes shim coil configurations to counteract those distortions by computing the magnetic field contribution of each coil via the Biot-Savart law, then solving for the optimal drive currents using constrained least-squares optimization.

## Functions

| File | Description |
|---|---|
| `Apply_Manual_Mask.m` | Interactively draws a region-of-interest (ROI) mask over the prostate from MRI magnitude maps |
| `Plot_Coils_9_vars.m` | Visualizes coil geometry relative to the ROI and body mask in 3D |
| `BiotSavart.m` | Computes the z-component of the magnetic field (Bz) produced by each shim coil at every voxel in the FOV using the Biot-Savart law |
| `Solve_Shim_Current.m` | Solves for optimal shim coil drive currents via constrained linear least squares (`lsqlin`) |
| `Bz_Calc.m` | Reconstructs the total shimmed field by superimposing per-coil Bz fields scaled by their solved currents |

## Shimming Pipeline

The functions are applied in the following order:

1. **`Apply_Manual_Mask`** -- Draw an ROI mask around the prostate region using an MRI magnitude image
2. **`Plot_Coils_9_vars`** -- Verify coil placement relative to the ROI and body
3. **`BiotSavart`** -- Compute the per-coil Bz field map across the entire FOV
4. **`Solve_Shim_Current`** -- Determine optimal drive currents to minimize B0 variation within the ROI
5. **`Bz_Calc`** -- Compute the resulting shimmed field and assess residual inhomogeneity

## Coil Parameterization

Each coil is described by 9 variables:

| Parameter | Unit | Description |
|---|---|---|
| `xc`, `yc`, `zc` | meters | Coil center coordinates |
| `r` | meters | Coil radius |
| `abratio` | -- | Elliptical aspect ratio (b = abratio × r) |
| `angX`, `angY`, `angZ` | degrees | Rotation angles about each axis |
| `I` | amperes | Drive current (set to 1 for field basis computation) |

## Installation

1. Download and install [MATLAB](https://www.mathworks.com/products/matlab.html)
2. Install the **Image Processing Toolbox** and **Parallel Computing Toolbox** via **Home > Add-Ons > Get Add-Ons**
3. Clone this repository and open the folder in MATLAB

## Usage

```matlab
% 1. Load your B0 phase map and magnitude map into the workspace
% 2. Draw the ROI mask
Apply_Manual_Mask

% 3. Define your coil matrix (N x 9) and visualize placement
Plot_Coils_9_vars(coilMatrix, prostateMask, bodyMask)

% 4. Compute per-coil Bz field maps
bodyBzEachCoil = FastBiotSavartZMulti(coilMatrix, prostateMask)

% 5. Solve for optimal shim currents (DClimit in amperes)
current = solveShimCurrent(bodyBzEachCoil, B0map, prostateMask, DClimit)

% 6. Reconstruct the shimmed field
sumBz = BzCalc(B0map, bodyBzEachCoil, current, prostateMask)
```

## Test Data

A sample B0 phase map and magnitude map from a physical phantom scan are provided in `UNIC_B0MapInVIVO_BH.mat`. Load this into MATLAB to test the pipeline without patient data.

## License

```
Copyright 2021 Eric Tang

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```


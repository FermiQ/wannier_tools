# Documentation for `src/berry.f90`

## Overview

The file `src/berry.f90` in the WannierTools package is dedicated to the calculation of the Berry phase along a specified path in k-space. The Berry phase is a geometric phase acquired by the Bloch wavefunction of an electron as it adiabatically traverses a closed loop in momentum space. It is a fundamental quantity for understanding various topological phenomena in condensed matter physics, such as the quantum Hall effect and the existence of topological insulators.

This file provides two main subroutines for this purpose: `berryphase` and `berryphase_atomic`, which differ in the gauge choice for the Hamiltonian construction.

## Key Components

### Subroutine `berryphase`

-   **Purpose**: Calculates the Berry phase along a user-defined k-path using the **lattice gauge** for the Hamiltonian.
-   **Input**:
    *   The k-path is defined by `k3points_Berry` (array of k-points) and `NK_Berry` (number of points defining the path segments), which are read from the input file and stored in the `para` module.
    *   `Nk_seg` (number of k-points to discretize each segment of the path, also from `para` as `Nk`).
    *   `NumOccupied` (number of occupied bands, from `para`).
-   **Method**:
    1.  **Path Discretization**: The input k-path (defined by `NK_Berry` points) is discretized into `NK_Berry_tot = (NK_Berry-1) * Nk_seg` k-points.
    2.  **Hamiltonian and Eigenvectors**: For each k-point along the discretized path:
        *   The bulk Hamiltonian $H(\mathbf{k})$ is constructed using `ham_bulk_latticegauge(k, uk)`.
        *   The Hamiltonian is diagonalized using `eigensystem_c` to obtain eigenvalues and eigenvectors (`uk`). The eigenvectors are stored in `Eigenvector(:,:,ik)`.
    3.  **Overlap Calculation**: The Berry phase is calculated by evaluating the product of overlaps between Bloch wavefunctions of adjacent k-points along the path:
        $P = \prod_{ik=1}^{NK_{Berry\_tot}-1} \langle u(\mathbf{k}_{ik}) | u(\mathbf{k}_{ik+1}) angle$
        For a closed loop, the final overlap is $\langle u(\mathbf{k}_{NK_{Berry\_tot}}) | u(\mathbf{k}_1) angle$.
        The phase factor $e^{-i \mathbf{b} \cdot \mathbf{r}_j}$ (where $\mathbf{b} = \mathbf{k}_{ik+1} - \mathbf{k}_{ik}$ and $\mathbf{r}_j$ is the Wannier center) is incorporated into the overlap calculation due to the lattice gauge:
        $	ext{overlap} = \sum_j u^*_{dag}(i,j) u_k(j,i) e^{-i \mathbf{b} \cdot \mathbf{r}_j}$.
        (Note: The original code uses `ratio= cos(2*pi*br)- zi*sin(2*pi*br)`, which is $e^{-i 2\pi \mathbf{b} \cdot \mathbf{r}_j}$, assuming $\mathbf{b}$ and $\mathbf{r}_j$ are in direct coordinates for the phase $2\pi \mathbf{b} \cdot \mathbf{r}_j$).
    4.  **Summation over Occupied Bands**: The total Berry phase (often related to the Chern number or Z2 invariant) is obtained by summing the phases from all occupied bands: $\Phi = \sum_{n=1}^{NumOccupied} 	ext{arg}(P_n)$.
-   **Output**:
    *   Prints the calculated Berry phase (in units of $\pi$) to `WT.out`.
    *   Writes the k-path and the total Berry phase to `kpath_berry.txt`.
-   **Warnings**: The code includes warnings to ensure the user increases `NK1` (likely meaning `Nk_seg` or k-point density) for convergence and that the k-path forms a closed loop or connects points differing by a reciprocal lattice vector.

### Subroutine `berryphase_atomic`

-   **Purpose**: Calculates the Berry phase along a user-defined k-path using the **atomic gauge** for the Hamiltonian.
-   **Input**: Same as `berryphase`.
-   **Method**:
    *   The overall procedure (path discretization, diagonalization, overlap product) is very similar to `berryphase`.
    *   The key difference is the use of `ham_bulk_atomicgauge(k, uk)` for constructing the Hamiltonian.
    *   When calculating the overlap for a closed loop (i.e., connecting the last point back to the first), a phase factor $e^{-i \mathbf{b} \cdot \mathbf{r}_j}$ is applied to the overlap $\langle u(\mathbf{k}_{NK_{Berry\_tot}}) | u(\mathbf{k}_1) angle$, where $\mathbf{b} = \mathbf{k}_1 - \mathbf{k}_{NK_{Berry\_tot}}$ (effectively the reciprocal lattice vector closing the loop). For intermediate steps along the path, this explicit phase factor in the overlap is not needed with the atomic gauge, as the phase is already incorporated into the definition of $H(\mathbf{k})$ and thus the eigenvectors.
-   **Output**: Same as `berryphase`.

## Important Variables/Constants

This file relies on several parameters and arrays defined in `src/module.f90` (via `use para`):

-   `NK_Berry`: Number of k-points defining the segments of the Berry phase calculation path. Read from the `KPATH_BERRY` card in `wt.in`.
-   `k3points_Berry(:, :)`: Array storing the coordinates of the k-points defining the path.
-   `Nk` (used as `Nk_seg`): Number of discrete k-points along each segment of the path.
-   `Num_wann`: Number of Wannier functions.
-   `NumOccupied`: Number of occupied bands.
-   `Origin_cell%wannier_centers_direct(:, :)`: Positions of Wannier centers, used in the phase factor for the lattice gauge.
-   `Origin_cell%Kua, Origin_cell%Kub, Origin_cell%Kuc`: Reciprocal lattice vectors for converting k-points to Cartesian if needed for output.
-   `pi`, `zi`: Mathematical constants.
-   `stdout`, `outfileindex`: Fortran unit numbers for output.

## Usage Examples

These subroutines are not typically called directly by a user but are invoked by the `main` program in `main.f90` when the `BerryPhase_calc` flag is set to true in the `wt.in` input file. The user specifies the k-path in the `KPATH_BERRY` section of `wt.in`.

**Example `wt.in` snippet:**
```
&CONTROL
  BerryPhase_calc = .TRUE.
  ...
&END

&PARAMETERS
  Nk = 50         ! Number of k-points per segment for Berry phase
  NumOccupied = 10 ! Example: 10 occupied bands
  ...
&END

&KPATH_BERRY
  G  0.0  0.0  0.0
  X  0.5  0.0  0.0
  M  0.5  0.5  0.0
  G  0.0  0.0  0.0  ! Closing the loop
&END
```
When WannierTools is run with such an input, `main.f90` will call either `berryphase` or `berryphase_atomic` (the choice might be implicit or set by another parameter, though typically `berryphase` with lattice gauge is common for standard Berry phase / Z2 calculations from Wannier Hamiltonians).

## Dependencies and Interactions

-   **`module.f90`**: Heavily dependent on the `para` module for input parameters, system definitions, and constants. Also uses `wmpi` for `cpuid`.
-   **`ham_bulk.f90`**: Calls `ham_bulk_latticegauge` or `ham_bulk_atomicgauge` from this file to construct the k-space Hamiltonian.
-   **Linear Algebra**: Relies on a routine `eigensystem_c` (likely a wrapper for a LAPACK routine like `zheevd` or similar) to diagonalize the Hamiltonian.
-   **`main.f90`**: Called from `main.f90` when `BerryPhase_calc = .TRUE.`.

The choice between `berryphase` (lattice gauge) and `berryphase_atomic` (atomic gauge) depends on how the wavefunctions and Hamiltonians are consistently defined and manipulated throughout the calculation sequence. For standard calculations based on `wannier90_hr.dat`, the lattice gauge approach is common for Berry phase calculations.
```

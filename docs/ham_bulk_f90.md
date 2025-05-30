# Documentation for `src/ham_bulk.f90`

## Overview

This file (`src/ham_bulk.f90`) is a crucial part of the WannierTools package. It contains a collection of Fortran subroutines dedicated to constructing the bulk Hamiltonian $H(\mathbf{k})$ for a crystalline system at a given k-vector. Different subroutines cater to various representations (e.g., atomic gauge, lattice gauge), model types (standard tight-binding, k.p models for specific systems), and storage formats (dense, sparse Coordinate Format - COO). Additionally, it provides routines to calculate derivatives of the Hamiltonian with respect to $\mathbf{k}$, which are essential for computing quantities like group velocities.

## Key Components

### Standard Hamiltonian Construction

-   **`ham_bulk_atomicgauge(k, Hamk_bulk)`**:
    *   Calculates the bulk Hamiltonian $H(\mathbf{k})$ using the "atomic gauge."
    *   The phase factor $e^{i\mathbf{k} \cdot (\mathbf{R} + \mathbf{	au}_j - \mathbf{	au}_i)}$ is applied, where $\mathbf{	au}_i$ and $\mathbf{	au}_j$ are the positions of Wannier centers within the unit cell.
    *   `k`: Input k-vector (direct coordinates).
    *   `Hamk_bulk`: Output $N_{wann} 	imes N_{wann}$ Hamiltonian matrix.

-   **`ham_bulk_latticegauge(k, Hamk_bulk)`**:
    *   Calculates the bulk Hamiltonian $H(\mathbf{k})$ using the "lattice gauge."
    *   The phase factor $e^{i\mathbf{k} \cdot \mathbf{R}}$ is applied. This is the standard form derived from `wannier90_hr.dat`.
    *   `k`: Input k-vector (direct coordinates).
    *   `Hamk_bulk`: Output $N_{wann} 	imes N_{wann}$ Hamiltonian matrix.

-   **`valley_k_atomicgauge(k, valley_k)`**:
    *   Performs a Fourier transform of a valley operator (presumably read from a file similar to `wannier90_hr.dat`) using the atomic gauge.
    *   `valley_k`: Output $N_{wann} 	imes N_{wann}$ matrix for the valley operator at $\mathbf{k}$.

### Sparse Hamiltonian Construction (COO Format)

-   **`ham_bulk_coo_densehr(k, nnzmax, nnz, acoo, icoo, jcoo)`**:
    *   Constructs $H(\mathbf{k})$ from a dense `HmnR` (read from `wannier90_hr.dat`) and outputs it in sparse Coordinate (COO) format.
    *   `nnzmax`: Input, maximum number of non-zero elements.
    *   `nnz`: Output, actual number of non-zero elements.
    *   `acoo, icoo, jcoo`: Output arrays for values, row indices, and column indices of the sparse Hamiltonian.

-   **`ham_bulk_coo_sparsehr_latticegauge(k, acoo, icoo, jcoo)`**:
    *   Constructs $H(\mathbf{k})$ in COO sparse format directly from a pre-read sparse `HmnR` (stored in `hicoo`, `hjcoo`, `hacoo`, `hirv` from `para` module) using the lattice gauge.
    *   Assumes `splen` (sparse length) is available from the `para` module.

-   **`ham_bulk_coo_sparsehr(k, acoo, icoo, jcoo)`**:
    *   Similar to `ham_bulk_coo_sparsehr_latticegauge` but uses the atomic gauge.

-   **`overlap_bulk_coo_sparse(k, acoo, icoo, jcoo)`**:
    *   Constructs the overlap matrix $S(\mathbf{k})$ in COO sparse format from a pre-read sparse overlap matrix (stored in `sicoo`, `sjcoo`, `sacoo`, `sirv`) using the atomic gauge. Used for non-orthogonal basis sets.

-   **`valley_k_coo_sparsehr(nnz, k, acoo, icoo, jcoo)`**:
    *   Constructs a valley operator at $\mathbf{k}$ in COO sparse format from a pre-read sparse valley operator, using the atomic gauge.

### k.p Model Hamiltonians

-   **`ham_bulk_kp_abcb_graphene(k, Hamk_bulk)`**:
    *   Constructs a specific k.p model Hamiltonian for ABCB tetralayer graphene around the K point.
    *   The parameters (`l(1)` to `l(19)`) for this model are hardcoded.
    *   Includes terms for electric fields (`Symmetrical_Electric_field_in_eVpA`, `Electric_field_in_eVpA`).
    *   `Num_wann` is expected to be 8 for this model.

-   **`ham_bulk_kp(k, Hamk_bulk)`**:
    *   Constructs a specific k.p model Hamiltonian, seemingly for a WC (Tungsten Carbide) system (based on comments about space group 187, C3h little group).
    *   Parameters (`A1, A2, ..., E1, E2, ...`) are hardcoded.
    *   `Num_wann` is expected to be 4 for this model.

### Specialized Hamiltonians

-   **`ham_bulk_LOTO(k, Hamk_bulk)`**:
    *   Calculates the Hamiltonian for a phonon system including LO-TO (longitudinal optical - transverse optical) splitting/correction.
    *   Uses `Diele_Tensor` (dielectric tensor) and `Born_Charge` (Born effective charges) from the `para` module.

-   **`ham_bulk_tpaw(mdim, k, Hamk_moire)`**:
    *   Appears to be a placeholder or starting point for calculating a Hamiltonian for a twisted system (e.g., moiré materials). The implementation details are sparse in the provided snippet ("for a test, we assume a twist homo-bilayer system").
    *   `mdim`: Leading dimension of the moiré Hamiltonian.

### Hamiltonian Derivatives (Velocities)

-   **`dHdk_atomicgauge(k, velocity_Wannier)`**:
    *   Calculates the first derivative of the Hamiltonian $
abla_{\mathbf{k}} H(\mathbf{k})$ in the Wannier basis using the atomic gauge.
    *   The result, `velocity_Wannier(n,m,dim)`, corresponds to the matrix elements of the velocity operator.

-   **`dHdk_atomicgauge_Ham(k, eigvec, Vmn_Ham)`**:
    *   Calculates the velocity operator matrix elements in the basis of Hamiltonian eigenvectors (Bloch states).
    *   It first calls `dHdk_atomicgauge` and then transforms the result to the Hamiltonian eigenbasis using `eigvec` via `rotation_to_Ham_basis`.

-   **`dHdk_latticegauge_wann(k, velocity_Wannier)`**:
    *   Calculates $
abla_{\mathbf{k}} H(\mathbf{k})$ in the Wannier basis using the lattice gauge.

-   **`dHdk_latticegauge_Ham(k, eigval, eigvec, Vmn_Ham)`**:
    *   Calculates the velocity operator in the Hamiltonian eigenbasis using the lattice gauge. This involves a more complex transformation that includes energy differences between states (`eigval(n) - eigval(m)`).

-   **`d2Hdk2_atomicgauge(k, DHDk2_wann)` / `d2Hdk2_atomicgauge_wann(k, D2HDk2_wann)`**:
    *   Calculates the second derivative of the Hamiltonian $
rac{\partial^2 H(\mathbf{k})}{\partial k_i \partial k_j}$ in the Wannier basis using the atomic gauge. (Note: Two subroutines with very similar names exist, likely one is primary).

-   **`d2Hdk2_atomicgauge_Ham(UU, dHdkdk, D2HDk2_Ham)`**:
    *   Transforms the second derivative matrix from the Wannier basis (`dHdkdk`) to the Hamiltonian eigenbasis (`D2HDk2_Ham`) using the eigenvector matrix `UU`.

### Utility Subroutines

-   **`rotation_to_Ham_basis(UU, mat_wann, mat_ham)`**:
    *   A general utility to transform a matrix (`mat_wann`) from the Wannier basis to the Hamiltonian eigenbasis (`mat_ham`) using the matrix of eigenvectors `UU`.
    *   Calculates `mat_ham = UU_dag * mat_wann * UU`.

## Important Variables/Constants

This file itself does not define many global constants but heavily relies on those defined in `src/module.f90` (via `use para`):
-   `Num_wann`: Number of Wannier functions, defining matrix dimensions.
-   `HmnR`: Array storing $H(\mathbf{R}_{mn})$, the real-space Hamiltonian matrix elements.
-   `irvec`: Integer array of $\mathbf{R}$ vectors.
-   `crvec`: Cartesian coordinates of $\mathbf{R}$ vectors.
-   `ndegen`: Degeneracy factor for each $\mathbf{R}$ vector.
-   `Origin_cell%wannier_centers_direct`, `Origin_cell%lattice`: Used for atomic gauge calculations.
-   `splen`, `hicoo`, `hjcoo`, `hacoo`, `hirv`: For sparse Hamiltonian construction.
-   `zi`, `pi2zi`, `twopi`: Mathematical constants.
-   `Rcut`: Cutoff radius for real-space summations.

## Usage Examples

These subroutines are fundamental building blocks called by higher-level routines in WannierTools, often within loops over k-points.

-   **Typical call to get Hamiltonian at a k-point (lattice gauge):**
    ```fortran
    use para
    ! ... k_vector is defined ...
    complex(dp) :: H_k(Num_wann, Num_wann)
    call ham_bulk_latticegauge(k_vector, H_k)
    ! ... now H_k can be diagonalized ...
    ```

-   **Getting velocity operators in Hamiltonian basis (atomic gauge):**
    ```fortran
    use para
    ! ... k_vector, eigenvectors_at_k are defined ...
    complex(dp) :: vel_op_H_basis(Num_wann, Num_wann, 3)
    call dHdk_atomicgauge_Ham(k_vector, eigenvectors_at_k, vel_op_H_basis)
    ```

## Dependencies and Interactions

-   **`module.f90`**: This file is critically dependent on `module.f90` (specifically the `para` module) for all global parameters, data structures (like `Origin_cell`, `HmnR`, sparse arrays), and mathematical/physical constants.
-   **Called By**:
    *   `main.f90`: Various routines in `main.f90` that set up calculations (e.g., before calling diagonalization routines for band structures).
    *   Energy band calculation routines (e.g., in `ek_bulk.f90`, `ek_slab.f90`).
    *   Routines calculating Berry phase, Berry curvature, Wilson loops, conductivities, etc., as these often require $H(\mathbf{k})$ and its derivatives at multiple k-points.
-   **Mathematical Libraries**: Implicitly relies on standard Fortran intrinsics for complex arithmetic and potentially external linear algebra libraries (like LAPACK/BLAS) if diagonalization is performed subsequently on the constructed Hamiltonians (though diagonalization itself is not in this file).

This file provides the essential machinery for representing the electronic structure of the material in k-space, forming the basis for nearly all subsequent property calculations in WannierTools.
```

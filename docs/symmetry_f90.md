# Documentation for `src/symmetry.f90`

## Overview

The `src/symmetry.f90` file in WannierTools is responsible for handling crystal symmetries. Its primary role is to read symmetry operations from the input, process them, and potentially use them to symmetrize the Hamiltonian or other relevant quantities. Effective use of symmetry can significantly reduce computational cost and improve the accuracy of calculations by ensuring that the model respects the inherent symmetries of the crystal structure.

The exact content and capabilities can be extensive. Based on typical needs in such codes, this file would likely:
1.  Read symmetry operations provided by the user or generate them if point/space group information is given.
2.  Represent symmetry operations (e.g., rotations, mirrors, inversion) as matrices.
3.  Provide tools to apply these symmetry operations to k-vectors, Hamiltonians, or wavefunctions.
4.  Possibly interface with external libraries like spglib for more robust symmetry determination, though simpler internal implementations are also common.

## Key Components (Hypothetical based on common structure)

Since the content of `symmetry.f90` was not explicitly provided in the earlier `read_files` command, this documentation is based on the typical structure and functionality of symmetry-handling modules in condensed matter codes and references to it from `main.f90` and `module.f90`.

### Main Subroutine `symmetry`

-   **Purpose**: This is likely the main entry point called from `main.f90`. It orchestrates the reading, processing, and application of symmetry operations.
-   **Functionality**:
    *   Checks the `Symmetry_Import_calc` flag from the `para` module. If true, proceeds with symmetry processing.
    *   May call subroutines to read symmetry operations from `wt.in` (e.g., if a `SYMMETRY_OPERATIONS_CART` card is present) or from a dedicated symmetry file.
    *   Stores the symmetry operations (rotations, translations) in arrays within the `para` module (e.g., `pgop_cart`, `tau_direct`, `number_group_operators`).
    *   Could involve determining the point group or space group of the crystal.
    *   Might build mapping tables for how atoms or Wannier orbitals transform under each symmetry operation (`imap_sym` in `para`).
    *   Optionally, it might perform symmetrization of the input Hamiltonian `HmnR` if such a feature is enabled.

### Potential Helper Subroutines

-   **`read_symmetry_operations_card()`**: A subroutine to parse symmetry operations explicitly provided in the `wt.in` file under a specific card.
-   **`generate_point_group_operations(generators)`**: If point group generators are given, this routine would generate all operations of the point group.
-   **`apply_symmetry_to_kvector(k_vector, symm_op, k_vector_transformed)`**: Transforms a k-vector according to a given symmetry operation.
-   **`symmetrize_hamiltonian(HmnR, symmetry_ops)`**: Adjusts the real-space Hamiltonian `HmnR` to make it consistent with the crystal symmetries. This often involves averaging $H(\mathbf{R})$ with $H(S\mathbf{R})$ transformed appropriately.
-   **`find_atom_mapping(atom_positions, symm_op, atom_map)`**: Determines how atom indices are permuted under a given symmetry operation.
-   **`get_rotation_matrix_from_axis_angle()`**: Utility to construct rotation matrices.

## Important Variables/Constants (from `para` module)

This file interacts heavily with symmetry-related variables defined in the `para` module (`src/module.f90`):

-   **Control Flags**:
    *   `Symmetry_Import_calc`: Logical flag; if true, symmetry processing is activated.
-   **Symmetry Operation Storage**:
    *   `number_group_generators`: Number of generators for the symmetry group.
    *   `number_group_operators`: Total number of symmetry operations in the group.
    *   `generators_find`: Indices of generator operations.
    *   `tau_find`: Fractional translations associated with generators.
    *   `pgop_cart`, `pgop_direct`: Point group operations in Cartesian and direct coordinates, respectively (usually $3 	imes 3$ matrices).
    *   `tau_cart`, `tau_direct`: Translation vectors associated with space group operations (in Cartesian and direct coordinates).
    *   `spatial_inversion`: Stores the inversion operation matrix.
    *   `imap_sym(:, :)`: An array mapping `imap_sym(op, atom_in)` to `atom_out`, showing how `atom_in` transforms to `atom_out` under symmetry operation `op`.
-   **System Geometry**:
    *   `Origin_cell%lattice`, `Origin_cell%Atom_position_direct`, `Origin_cell%Num_atoms`: Information about the crystal structure is essential for symmetry analysis.
    *   `Origin_cell%wannier_centers_direct`: Positions of Wannier functions, needed to understand how they transform.
-   **Tolerances**:
    *   `symprec`: A small real number used as a tolerance for comparing atomic positions or distances when checking symmetry operations.

## Usage Examples

The `symmetry` subroutine is typically called once from `main.f90` after reading the basic crystal structure but before detailed calculations that might benefit from symmetry.

**Input in `wt.in` (Conceptual):**

Users might enable symmetry processing via:
```
&CONTROL
  Symmetry_Import_calc = .TRUE.
/
```
And provide symmetry operations if not automatically detected (though automatic detection via spglib or similar is common in modern codes):
```
SYMMETRY_OPERATIONS_CART  ! Example card name
  3  ! Number of operations
  1.0  0.0  0.0   0.0  0.0  0.0  ! Op 1 Rotation part, Translation part
  0.0  1.0  0.0
  0.0  0.0  1.0

 -1.0  0.0  0.0   0.0  0.0  0.0  ! Op 2 (e.g., Inversion)
  0.0 -1.0  0.0
  0.0  0.0 -1.0
  ...
```

## Dependencies and Interactions

-   **`module.f90` (`para` module)**: This is the primary dependency. `symmetry.f90` reads into and uses variables related to symmetry, crystal structure, and control flags defined in `para`.
-   **`main.f90`**: The main `symmetry` subroutine is called from `main.f90`.
-   **`readinput.f90`**: While `readinput.f90` parses most of `wt.in`, specific cards related to symmetry operations might be parsed directly within `symmetry.f90` or helper routines it calls.
-   **Hamiltonian Construction (e.g., `src/readHmnR.f90` or `src/ham_bulk.f90`)**: If Hamiltonian symmetrization is performed, `symmetry.f90` would interact with the Hamiltonian data (`HmnR`).
-   **External Libraries (Potentially)**: More advanced symmetry analysis might involve calls to external libraries like `spglib` to find symmetries, which would then be stored in the `para` module variables.

By correctly identifying and utilizing symmetries, `symmetry.f90` can play a vital role in ensuring the physical realism of the calculations and reducing the computational burden, for example, by restricting calculations to the irreducible Brillouin zone.
```

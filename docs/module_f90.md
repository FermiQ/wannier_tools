# Documentation for `src/module.f90`

## Overview

This file is a cornerstone of the WannierTools application. It defines several critical Fortran modules that provide:
- Control over numerical precision.
- MPI (Message Passing Interface) related definitions, communication structures, and helper subroutines for parallel execution.
- A vast collection of global parameters, namelist definitions for input file parsing, and derived data types that model physical systems and control program behavior.
- Specific utility modules for accessing periodic table information and performing calculations related to Angle-Resolved Photoemission Spectroscopy (ARPES) matrix elements.

Due to its central role, many other files in the `src/` directory and elsewhere depend on the modules defined here, particularly for accessing shared parameters and data structures.

## Key Components

### Module `prec`
- **Purpose**: Controls the numerical precision used throughout the application.
- **Key Parameters**:
    - `li`: Defines the kind for long integers (e.g., for large array indexing).
    - `Dp`: Defines the kind for double-precision floating-point numbers, ensuring high accuracy for scientific calculations.

### Module `wmpi`
- **Purpose**: Encapsulates MPI definitions and utilities for parallel processing.
- **Key Structures**:
    - `WTParCSRComm`: Structure for managing communication of data in a Compressed Sparse Row (CSR) format, likely used for sparse matrices. Includes fields for send/receive CPUs, map starts, and map elements.
    - `WTParVecComm`: Structure for managing communication of vector data.
    - `WTCommHandle`: Handle for managing MPI requests and data buffers for non-blocking communication.
- **Key Subroutines**:
    - `WTGeneratePartition(length, nprocs, part)`: Generates a partition of a given length across `nprocs` processors.
    - `WTGenerateLocalPartition(length, nprocs, icpu, first, last)`: Generates the local portion of a distributed array for a specific CPU.
    - `WTCommHandleCreate(SendRecv, SendData, RecvData, CommHandle)`: Initializes a communication handle for sending and receiving data.
    - `WTCommHandleDestroy(CommHandle)`: Finalizes communication and cleans up resources associated with a `WTCommHandle`.
    - `mp_allreduce_z(comm, ndim, vec, vec_mpi)`: A wrapper for MPI_Allreduce for complex arrays.

### Module `para`
- **Purpose**: This is arguably the most critical module, defining a vast array of global parameters, control flags, derived types, and namelists that govern the program's execution and data representation.
- **Namelists (for input file `wt.in`)**:
    - `TB_FILE`: Parameters related to the tight-binding Hamiltonian file (e.g., `Hrfile`, `Is_Sparse_Hr`).
    - `CONTROL`: Boolean flags that enable or disable specific calculations (e.g., `BulkBand_calc`, `SlabSS_calc`, `BerryCurvature_calc`).
    - `SYSTEM`: Parameters defining the physical system (e.g., `Soc` for spin-orbit coupling, `E_fermi`, `Nslab` for slab thickness, `Bx`, `By`, `Bz` for magnetic field).
    - `PARAMETERS`: Numerical parameters for calculations (e.g., `E_arc` for Fermi energy in arc calculations, `OmegaMin`, `OmegaMax` for energy ranges, `Nk1`, `Nk2`, `Nk3` for k-point mesh densities).
- **Derived Types (Data Structures)**:
    - `cell_type`: Represents a crystal cell, storing lattice vectors, atomic positions, projector information, etc. Used for `Origin_cell`, `Folded_cell`, `Magnetic_cell`.
    - `dense_tb_hr`: Describes a tight-binding Hamiltonian in the dense Wannier90 `hr.dat` format.
    - `kcube_type`: Manages k-point meshes for calculations over the Brillouin zone, including MPI distribution.
    - `int_array1D`: A simple type for a 1D integer array.
- **Example Parameters/Flags**:
    - `version`: Character string for the WannierTools version.
    - `stdout`: Integer unit number for standard output (typically `WT.out`).
    - `Hrfile`: Path to the tight-binding Hamiltonian file.
    - `BulkBand_calc`: Logical flag to enable bulk band structure calculation.
    - `E_fermi`: Fermi energy in eV.
    - `Num_wann`: Number of Wannier functions.
    - `Nk1, Nk2, Nk3`: Number of k-points along different directions for BZ sampling.
    - `Origin_cell`: A `cell_type` variable storing the primary unit cell information.
    - `HmnR`: Stores the Hamiltonian matrix elements.

### Module `wcc_module`
- **Purpose**: Provides data structures for Wannier Charge Center (WCC) and Wilson loop calculations.
- **Key Types**:
    - `kline_wcc_type`: Stores properties at a k-point for WCC calculation, including the WCC values, gaps, and convergence status.
    - `kline_integrate_type`: Stores k-point properties for integration steps in Wilson loop calculations, including eigenvectors.

### Module `element_table`
- **Purpose**: Provides data and utility functions related to the periodic table of elements.
- **Key Data**:
    - `element_name`: Array of element symbols.
    - `element_electron_config`: Array of electron configurations.
    - `orb_ang_sign`, `orb_sign`: Arrays for orbital characters (s, p, d, f) and their specific names (e.g., px, dz2).
- **Key Subroutines**:
    - `get_element_index(name, index_)`: Gets the atomic number (index) for a given element name.
    - `get_orbital_index(orbital_name, index_l, index_m)`: Gets quantum numbers l and m for a given orbital name.
    - `get_electron_config(name, configuration)`: Retrieves the full electron configuration for an element.
    - `get_valence_config(name, v_configuration)`: Retrieves the valence electron configuration.
    - `factorial(n, res)`: Calculates factorial.

### Module `me_calculate`
- **Purpose**: Contains subroutines to assist in the calculation of matrix elements, particularly for simulating Angle-Resolved Photoemission Spectroscopy (ARPES).
- **Key Subroutines**:
    - `gaunt(l, m, dl, dm, gaunt_value)`: Calculates Gaunt coefficients for dipole transitions under complex spherical harmonics.
    - `gaunt_real_basis(l, m, dl, dm, realbasis_gaunt_value, k_theta, k_phi)`: Calculates Gaunt coefficients under real spherical harmonics.
    - `polarization_vector(polar_epsilon)`: Calculates the polarization vector of incoming light based on input parameters.
    - `Z_eff_cal(name, n, l, Z_eff)`: Calculates the effective nuclear charge Z_eff using Slater's rules.
    - `slater_Rn(r, n, Z_eff)`: Generates a Slater-type radial wave function.
    - `spherical_bessel(x, l)`: Calculates spherical Bessel functions.
    - `Ylm(theta, phi, l, m)`: Calculates real spherical harmonics.
    - `integrand_r2(r, k_f, name, n, l)`, `integrand_r3(...)`: Define integrands for radial integrals.
    - `integral_simpson(a, b, func, k_f, name, n, l, ans)`: Performs numerical integration using Simpson's rule.
    - `get_matrix_element(ele_name, orb_name, k_cart, atom_position_cart, matrix_element_res)`: The main subroutine to calculate the ARPES matrix element.

## Important Variables/Constants

Many constants are defined in the `para` module for physical units, mathematical constants, and default values:
- `Pi`: Mathematical constant pi.
- `twopi`: 2 * Pi.
- `zi`: Complex number (0, 1).
- `eV2Hartree`: Conversion factor from electron-volts to Hartrees.
- `Angstrom2atomic` (or `Ang2Bohr`): Conversion factor from Angstroms to Bohr radii.
- `stdout`: Fortran unit number for standard output (file `WT.out`).
- Default values for various tolerances (e.g., `eps6`, `eps8`, `eps12`).

## Usage Examples

Modules from this file are used throughout the WannierTools code.

- **Accessing Parameters**: Any Fortran file needing access to global parameters or types will include `use para`.
  ```fortran
  program some_calculation
    use para
    implicit none
    ! Now E_fermi, Num_wann, etc. are available
    if (BulkBand_calc) then
      ! Do bulk band calculation
    endif
  end program
  ```

- **MPI Communication**: MPI functionalities are accessed via `use wmpi`.
  ```fortran
  subroutine parallel_task
    use wmpi
    implicit none
    ! cpuid, num_cpu are available
    if (cpuid == 0) then
      ! Master process tasks
    endif
    ! Call WTGeneratePartition or other MPI helpers
  end subroutine
  ```

- **Namelist Reading**: The `readinput.f90` file reads namelists like `CONTROL` and `SYSTEM` defined in `para` to populate these global variables from the `wt.in` input file.

## Dependencies and Interactions

- **Self-contained Precision**: `prec` is fundamental and used by other modules within this file.
- **MPI Core**: `wmpi` provides core parallel functionalities. It might use `prec`.
- **Central Hub**: `para` is the central hub for parameters and types. It uses `prec` and `wmpi`. Most other calculation files (`*.f90`) in `src/` will `use para`.
- **Specialized Utilities**: `wcc_module`, `element_table`, and `me_calculate` use `para` (and indirectly `prec`, `wmpi`). They provide specific functionalities to other parts of the code, often called from `main.f90` or routines involved in specific calculations.
- **Input System**: `readinput.f90` heavily relies on `para` to define the variables that will be filled from the input file.
- **Main Program**: `main.f90` uses `wmpi` for parallel setup and `para` for almost all control flow and data passing to calculation subroutines.

```

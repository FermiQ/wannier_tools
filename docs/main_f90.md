# Documentation for `src/main.f90`

## Overview

`main.f90` serves as the primary entry point and control program for the WannierTools software package. It orchestrates the various computational tasks based on parameters read from an input file (typically `wt.in`). The program initializes the environment (including MPI for parallel execution), reads and processes inputs, sets up the Hamiltonian, and then calls a sequence of subroutines to perform the requested calculations, such as bulk band structures, surface states, Berry curvature, Wilson loops, and transport properties. It also manages timing for different computational stages and outputs results to `WT.out` and other specific files.

## Key Components

### Program `main`

The `main` program follows a structured execution flow:

1.  **Initialization**:
    *   Sets the WannierTools `version`.
    *   Initializes the MPI environment if compiled with MPI support (`call mpi_init`, `mpi_comm_rank`, `mpi_comm_size`).
    *   Opens the main output file `WT.out` (on CPU 0).
    *   Records initial time using `call now(time_init)`.
    *   Calls `header` to print introductory information.

2.  **Input Processing**:
    *   Calls `readinput` (from `readinput.f90`) to parse the user-provided input file (e.g., `wt.in`). This populates parameters in the `para` module.
    *   Determines `Num_wann` (number of Wannier functions) based on input and handles adjustments for Zeeman fields if `SOC` (spin-orbit coupling) is off.
    *   Sets `Num_wann_BdG` for Bogoliubov-de Gennes calculations.
    *   Sets `Ndim` for surface Green's function calculations.

3.  **Symmetry Setup**:
    *   Calls `symmetry` (from `symmetry.f90`) to process symmetry information, which might involve reading symmetry operators or symmetrizing the Hamiltonian.

4.  **Hamiltonian Loading**:
    *   Checks `Is_HrFile`.
    *   If `Is_Sparse_Hr` is true, calls `readSparseHmnR()` to read a sparse representation of the Hamiltonian. It may also call `readsparse_valley_operator` or `readsparse_overlap` if needed.
    *   Otherwise, calls `readNormalHmnR()` to read the standard `wannier90_hr.dat` file. It may also call `read_valley_operator`.

5.  **Conditional Calculations**:
    *   The program then enters a large series of `if` blocks. Each block checks a boolean flag (e.g., `BulkBand_calc`, `SlabSS_calc`, `Wilsonloop_calc`) from the `para` module (set during input reading).
    *   If a flag is true, the corresponding calculation subroutine(s) are called. For example:
        *   `BulkBand_unfold_line_calc`: calls `unfolding_kpath`.
        *   `BulkBand_calc` or `BulkBand_line_calc`:
            *   If sparse: calls `sparse_ekbulk_valley` or `sparse_ekbulk`.
            *   If dense: calls `ek_bulk_line_valley` or `ek_bulk_line`.
        *   `LandauLevel_B_dos_calc`: calls `LandauLevel_B_dos_Lanczos`.
        *   `FindNodes_calc`: calls `FindNodes`.
        *   `Dos_calc`: calls `dos_sub` or `dos_sparse`.
        *   `SlabBand_calc`: calls `ek_slab_sparseHR` or `ek_slab`.
        *   `BerryCurvature_calc`: calls `Berry_curvature_plane_full`.
        *   `Wilsonloop_calc`: calls `wannier_center3D_plane_adaptive`.
        *   `AHC_calc` (Anomalous Hall Conductivity): calls `sigma_AHC`.
        *   `SlabSS_calc` (Surface States): calls `surfstat`.
        *   Many other calculation types are supported (see the `CONTROL` namelist in `module.f90`).
    *   Each calculation block is typically timed using `call now(time_start)` and `call now(time_end)`, with the duration printed via `call print_time_cost`.

6.  **Finalization**:
    *   Records total execution time.
    *   Calls `footer` to print concluding messages.
    *   Finalizes the MPI environment (`call mpi_finalize`) if used.

## Important Variables/Constants

-   `version` (character): Stores the current version string of WannierTools.
-   `cpuid` (integer): The rank of the current MPI process (0 for the master process).
-   `num_cpu` (integer): The total number of MPI processes being used.
-   `time_start`, `time_end`, `time_init` (real(Dp)): Variables for timing program sections.
-   Numerous logical flags from the `para` module (e.g., `BulkBand_calc`, `SlabSS_calc`, `AHC_calc`) control which computational branches are executed. These are set by `readinput`.

## Usage Examples

`main.f90` is compiled into the main WannierTools executable (e.g., `wt.x`). It is typically run from the command line, requiring an input file (commonly named `wt.in`) that specifies the desired calculations and system parameters.

```bash
# Example execution (actual executable name might vary)
./wt.x
```
The program reads `wt.in` from the current directory and produces `WT.out` and other data files.

## Dependencies and Interactions

-   **`module.f90`**: Critically depends on `module.f90` for:
    *   `wmpi`: For all MPI related functionalities.
    *   `para`: For almost all global parameters, control flags (which drive the conditional execution of calculations), derived types, and namelist definitions.
-   **`readinput.f90`**: Calls the `readinput` subroutine from this file to parse the input configuration.
-   **`symmetry.f90`**: Calls the `symmetry` subroutine to handle symmetry operations.
-   **Hamiltonian Reading (`readHmnR.f90`)**: Calls `readNormalHmnR` or `readSparseHmnR` (likely from `readHmnR.f90` or a similar file) to load the tight-binding Hamiltonian.
-   **Calculation Subroutines**: Calls a multitude of subroutines from various other `.f90` files in the `src/` directory, each responsible for a specific physical calculation. Examples include:
    *   `ek_bulk.f90` (for `ek_bulk_line`, `sparse_ekbulk`, etc.)
    *   `ek_slab.f90` (for `ek_slab`)
    *   `fermisurface.f90` (for `fermisurface_kplane`, `fermisurface3D`)
    *   `dos.f90` (for `dos_sub`, `dos_sparse`)
    *   `berrycurvature.f90` (for `Berry_curvature_plane_full`, `Berry_curvature_cube`)
    *   `wcc.f90` (likely contains Wilson loop calculations like `wannier_center3D_plane_adaptive`)
    *   `sigma.f90` (for `sigma_AHC`, `sigma_SHC`)
    *   `surfstat.f90` (likely contains `surfstat` for surface states)
    *   And many others corresponding to the flags in the `CONTROL` namelist.
-   **Utility Subroutines**: Uses utility subroutines like `now` (likely a system call wrapper for time), `header`, `footer`, and `print_time_cost`.

The `main.f90` file acts as the central nervous system of WannierTools, connecting input processing, core model setup, and specialized computational modules.
```

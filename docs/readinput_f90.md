# Documentation for `src/readinput.f90`

## Overview

The `src/readinput.f90` file plays a critical role in the WannierTools workflow by parsing the main user input file, conventionally named `wt.in`. This input file contains various parameters, control flags, and data blocks that dictate the type of calculations to be performed and the specifics of the physical system being studied. The `readinput` subroutine within this file reads `wt.in`, interprets its contents, and populates the corresponding variables and structures defined in the `para` module (`src/module.f90`).

## Key Components

### Subroutine `readinput`

This is the primary, and likely only, public subroutine in this file. Its main responsibilities include:

1.  **Opening and Reading `wt.in`**: It opens the input file (defaulting to `wt.in` if not specified otherwise, though typically hardcoded as `wt.in`).
2.  **Namelist Parsing**: It reads several Fortran namelists. Namelists provide a structured way to input a group of related variables. Key namelists processed are:
    *   `TB_FILE`: Contains parameters related to the tight-binding Hamiltonian input file, such as `Hrfile` (path to `wannier90_hr.dat`), `Is_Sparse_Hr` (if the Hamiltonian is in sparse format), `Orthogonal_Basis`, and `Overlapfile`.
    *   `CONTROL`: A series of logical (boolean) flags that enable or disable specific calculations (e.g., `BulkBand_calc`, `SlabSS_calc`, `BerryCurvature_calc`, `Wilsonloop_calc`, `AHC_calc`).
    *   `SYSTEM`: Parameters defining the physical system, such as `Soc` (spin-orbit coupling strength/flag), `E_fermi` (Fermi energy), `Nslab` (number of layers for slab calculations), `Bx`, `By`, `Bz` (magnetic field components), `Numoccupied`, `Ntotch` (total charge).
    *   `PARAMETERS`: Numerical parameters controlling the details of calculations, like `E_arc` (Fermi energy for Fermi arc plots), `OmegaMin`, `OmegaMax` (energy window for DOS or spectral functions), `Nk1`, `Nk2`, `Nk3` (k-point mesh densities for various dimensions), `Rcut` (cutoff radius for real-space hoppings), `Beta` (inverse temperature).
3.  **Card/Keyword Parsing**: Beyond namelists, `readinput` parses specific "cards" or keywords that introduce blocks of data. This is done by reading lines and checking for specific strings. Examples of cards include:
    *   `LATTICE`: Defines the primitive lattice vectors of the crystal.
    *   `ATOM_POSITIONS`: Specifies the types and positions of atoms in the unit cell.
    *   `PROJECTORS` (or `WANNIER_CENTERS_CART` / `WANNIER_CENTERS_DIRECT`): Defines Wannier function centers and their orbital character.
    *   `SURFACE`: Defines the surface orientation for slab calculations.
    *   `KPATH_BULK`, `KPATH_SLAB`, `KPATH_WIRE`, `KPATH_BERRY`: Define paths in k-space for band structure, Berry phase calculations, etc.
    *   `CENTER_ATOM_ELECTRIC_FIELD`: Specifies atoms for electric field calculations.
    *   `WEYL_NODE_KPOINTS`, `NODAL_LOOP_PARAMETERS`: Input for specific topological feature calculations.
    *   `SYMMETRY_OPERATIONS_CART`: For providing custom symmetry operations.
    *   `SELECTED_ATOMS_GROUP`, `SELECTED_WANNIER_ORBITALS_GROUP`: For selecting specific atoms or orbitals for projection or analysis.
    *   `SUPERCELL_MATRIX`: Defines the supercell transformation matrix for band unfolding.
4.  **Data Storage**: All data read is stored in variables and derived types within the `para` module. For instance, lattice vectors go into `Origin_cell%lattice`, atomic positions into `Origin_cell%Atom_position_direct`, control flags like `BulkBand_calc` are set directly, etc.
5.  **Error Handling and Defaults**: The subroutine may include basic error checking (e.g., if essential parameters are missing) and set default values for some parameters if they are not provided in `wt.in`.
6.  **Unit Conversion**: It might handle conversions, for example, from Angstroms to Bohr radii if needed internally, although many parameters are expected in specific units (e.g., eV for energies).

### Internal Helper Subroutines (Conceptual)

While not explicitly shown in a brief overview, `readinput.f90` might contain internal helper subroutines to:
-   Parse specific data blocks (e.g., a routine to read a 3x3 matrix for `LATTICE`).
-   Convert strings to numbers.
-   Trim whitespace and handle comments in the input file.

## Important Variables/Constants

This file's primary role is to *populate* variables in the `para` module. It doesn't define its own global constants for external use but rather reads values into those defined in `para`. The structure of `wt.in` directly mirrors the namelists and expected card structures tied to variables in `para`.

## Usage Examples

The `readinput` subroutine is called once near the beginning of the `main` program in `main.f90`.

**Structure of `wt.in`:**

```
&TB_FILE
  Hrfile = 'wannier90_hr.dat'
/

&CONTROL
  BulkBand_calc = .TRUE.
  Dos_calc = .TRUE.
/

&SYSTEM
  E_fermi = 0.0
  Soc = 0
/

&PARAMETERS
  Nk1 = 50
  Nk2 = 50
  Nk3 = 50
  OmegaMin = -5.0
  OmegaMax = 5.0
  OmegaNum = 500
/

LATTICE ANGBOHR  ! AngOrBohr parameter can be set here
  1.75  0.00  0.00
  0.875 1.515 0.00
  0.00  0.00  10.0

ATOM_POSITIONS DIRECT  ! DirectOrCart parameter
  C  0.333333  0.666667  0.5
  C  0.666667  0.333333  0.5

KPATH_BULK  ! For bulk band structure
  G  0.0  0.0  0.0
  M  0.5  0.0  0.0
  K  0.333333  0.333333  0.0
  G  0.0  0.0  0.0
```

The `readinput` subroutine parses this file line by line or using Fortran's namelist read capabilities.

## Dependencies and Interactions

-   **`module.f90` (`para` module)**: This is the most critical dependency. `readinput.f90` exists almost solely to populate the variables and derived types (like `Origin_cell`, `KPath_Bulk_kpoints`, various logical flags, and numerical parameters) defined within the `para` module.
-   **`main.f90`**: The `readinput` subroutine is called by the `main` program. The subsequent behavior of `main.f90` (which calculations are run, with what parameters) is entirely determined by the variables set by `readinput`.
-   **Standard Fortran I/O**: Uses standard Fortran `OPEN`, `READ`, `CLOSE`, `REWIND`, `INQUIRE` statements.

`readinput.f90` acts as the bridge between the user's intentions (expressed in `wt.in`) and the internal state of the WannierTools program, enabling flexible control over complex condensed matter calculations.
```

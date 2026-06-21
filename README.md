# IRENE

A [FEniCS](https://fenicsproject.org/)-based finite-element library for numerically solving the steady state and dynamics of viscous fluid layers on curved, evolving two-dimensional surfaces, that can be represented, through the Monge gauge, as the graph of a function. It expresses the governing equations in fully covariant form (curvilinear coordinates, Christoffel symbols, second fundamental form).

---

## What it does

- **Differential geometry on manifolds** — metric and curvature tensors, covariant derivatives of scalars/vectors/forms, Laplace–Beltrami operator, rate-of-deformation tensors for surface flows, and frame maps between the tangent bundle and the embedding 3D space.
- **Fluid mechanics** — surface Navier–Stokes / Stokes flow, including, and forces exerted on domain boundaries.
- **Mesh generation** — parametric `gmsh`/`pygmsh` meshes in 1D, 2D, and 3D (lines, disks, rings, squares with embedded circles/ellipses/polygons, half-circles, balls, boxes, submeshes), each with a matching tag-checking utility.
- **Steady-state and dynamic solvers** — steady flow / no-flow problems and time-dependent dynamics on curved manifolds, with post-processing scripts for fields, forces, errors, and timing.

---

## Repository layout

```
irene/
├── modules/                     # Shared Python package (imported by all solvers)
│   ├── differential_geometry/   # Differential-geometry module for manifold and manifold boundary
│   ├── physics/                 # Module containin physics submodules
│   ├── mesh/                    # load / read / check_tags / utils for each mesh type
│   ├── parameters/              # CSV parameter readers (mesh / solution / analysis)
│   └── calculus.py              # Parametric curves, curve/surface integrals, transforms
│   
├── generate_mesh/               # Mesh-generation pipeline
│   ├── 1d/  2d/  3d/            # One directory per geometry; each holds:
│   │                            #   generate_mesh.py, check_mesh.py,
│   │                            #   mesh_parameters.csv, runtime_arguments.py
│
├── steady_state/                # Steady-state solvers
│   ├── flow/                    # with flows
│   └── no_flow/                 # without flows
│
├── dynamics/                    # Time-dependent solvers (moving manifold)
│   └── channel_with_cylinder_curved_cn/   # Fixed manifold
│
└── animations/                  # Example output animations (.mp4)
```

Each solver directory follows the same convention:

| File | Role |
|------|------|
| `solve.py` | Entry point — assembles and solves the variational problem |
| `switch_problem.py` | Maps a problem name (e.g. `square_a`) to its mesh reader, variational problem, and print-out modules |
| `variational_problem_*.py` | UFL weak forms for each boundary-condition case |
| `function_spaces.py` | Finite-element function spaces |
| `solution_paths.py` / `files.py` | Output path and file handling |
| `parameters_bc_*.csv` | Physical and numerical parameters for each case |
| `print_out_*.py` | Print out of solution (fields, boundary forces, errors, timing) |

---

## Installation

See installation_instructions.pdf. 

---

## Workflow

The typical pipeline has three stages: **generate a mesh → solve → post-process**.

### 1. Generate a mesh

Example: 

To generate a mesh with a disk geometry

```bash
cd /home/fenics/shared/generate_mesh/2d/disk/
PARAMETERS_PATH="/home/fenics/shared/generate_mesh/2d/disk"
SOLUTION_PATH="$PARAMETERS_PATH/solution"
rm -rf "$SOLUTION_PATH"; mkdir "$SOLUTION_PATH"
python3 generate_mesh.py "$PARAMETERS_PATH" "$SOLUTION_PATH"
```

Mesh geometry and resolution are read from the local `mesh_parameters.csv` (e.g. radius `r`, center `c_r`, `resolution`).



[Optional] verify mesh tags:

```bash
CHECK_PATH="$PARAMETERS_PATH/check"
rm -rf "$CHECK_PATH"; mkdir "$CHECK_PATH"
python3 check_mesh.py "$SOLUTION_PATH" "$CHECK_PATH"
```

Tagged mesh regions (volumes, surfaces, vertices) are tested by integrating over them a function, and the result is compared with the finite-element integration. The maximal, relative discrepancy is printed out as ```Maximum relative error of mesh integrals```. 

### 2. Solve


Enter a solver directory (`steady_state/flow`, `steady_state/no_flow`, or `dynamics`) and solve:

Example:

```bash
cd /home/fenics/shared/steady_state/no_flow
MESH_PATH="/home/fenics/shared/generate_mesh/2d/ring/solution"
SOLUTION_PATH="/home/fenics/shared/steady_state/no_flow/solution"
rm -rf "$SOLUTION_PATH"
python3 solve.py ring "$MESH_PATH" "$SOLUTION_PATH"
```
`solve.py` takes three positional arguments: the **problem name** (resolved by `switch_problem.py`), the **mesh input directory**, and the **solution output directory**. Physical/numerical parameters come from the case's `parameters_bc_*.csv`. Each solver ships `print_out_*.py` scripts that read the solution and write out fields, boundary forces, errors, or timings for plotting and analysis.

## License

MIT license (MIT). 

## Authors

Dennis Wörthmüller (1,2), Gaetano Ferraro (1,2,3), Pierre Sen (1,2), and Michele Castellana* (1,2)

1 Institut Curie, PSL Research University, Paris, France

2 CNRS UMR168, 11 rue Pierre et Marie Curie, 75005, Paris,France

2 Polytechnic University of Turin, Corso Castelfidardo 39, 10129 Turin, Italy

* Corresponding author: michele.castellana@curie.fr
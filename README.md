# Analysis-of-Water-Hammer-using-OpenFOAM
Simulation of water hammer in OpenFOAM

## Compatibility
This repository is compatible with **OpenFOAM 11** (Foundation version).

## Quick Start with Docker

For detailed instructions on running this simulation using Docker, see **[DOCKER_INSTRUCTIONS.md](DOCKER_INSTRUCTIONS.md)**.

**Having issues?** Check **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)** for solutions to common errors.

**Quick command:**
```bash
docker pull openfoam/openfoam11-paraview510
docker run -it --rm -v $(pwd):/workspace -w /workspace/waterHammer openfoam/openfoam11-paraview510 /bin/bash
source /opt/openfoam11/etc/bashrc
cd vane && ./meshUnv && cd ../domain && ./mesh && ./run
```

## Overview
This case simulates the water hammer phenomenon using OpenFOAM's overset mesh capability with dynamic mesh motion. The simulation includes:
- Dynamic overset mesh (domain and vane cases)
- Oscillating vane motion causing pressure waves
- Laminar flow simulation
- Time-dependent pressure and velocity fields

## Simulation Workflow

1. **Generate vane mesh:** Import from UNV file and transform
2. **Generate domain mesh:** Create background mesh and merge with vane
3. **Run simulation:** Execute `overPimpleDyMFoam` solver
4. **Post-process:** Visualize results in ParaView

## Key Parameters

- **Solver:** overPimpleDyMFoam
- **End time:** 1.5 s
- **Time step:** 0.00025 s (adjustable)
- **Vane motion:** Oscillating, amplitude 0.1 m, frequency 2.0 rad/s
- **Fluid:** Water (ν = 1e-06 m²/s, laminar)

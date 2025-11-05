# Running Water Hammer Simulation in OpenFOAM 11 with Docker

This guide provides step-by-step instructions for running the water hammer simulation using OpenFOAM 11 in a Docker container.

## Prerequisites

- Docker installed on your system
- At least 4GB of free RAM
- Git (to clone the repository)

## Quick Start

### 1. Pull the OpenFOAM 11 Docker Image

```bash
docker pull openfoam/openfoam11-paraview510
```

### 2. Clone the Repository

```bash
git clone https://github.com/gabejohnsnn/Analysis-of-Water-Hammer-using-OpenFOAM.git
cd Analysis-of-Water-Hammer-using-OpenFOAM
```

### 3. Start the OpenFOAM Docker Container

```bash
docker run -it --rm \
  -v $(pwd):/workspace \
  -w /workspace/waterHammer \
  openfoam/openfoam11-paraview510 \
  /bin/bash
```

**Windows PowerShell users:**
```powershell
docker run -it --rm -v ${PWD}:/workspace -w /workspace/waterHammer openfoam/openfoam11-paraview510 /bin/bash
```

### 4. Source OpenFOAM Environment (Inside Container)

```bash
source /opt/openfoam11/etc/bashrc
```

## Running the Simulation

### Step 1: Generate the Vane Mesh

```bash
cd vane
./meshUnv
cd ..
```

This will:
- Import the vane geometry from `vane.unv` file
- Apply boundary condition changes
- Transform the mesh to the correct position

### Step 2: Generate and Merge the Domain Mesh

```bash
cd domain
./mesh
```

This will:
- Create the background mesh using `blockMesh`
- Apply boundary condition changes
- Merge the vane mesh with the domain mesh
- Set up the overset zones
- Initialize the zoneID field

### Step 3: Run the Simulation

**Option A: Serial Run (for testing)**
```bash
overPimpleDyMFoam | tee log.run
```

**Option B: Parallel Run (recommended, 3 processors)**
```bash
./run
```

This will:
- Decompose the case for 3 processors
- Run the simulation in parallel with `mpirun`
- Log output to `log.run`

### Step 4: Post-Processing

**Inside the container:**

```bash
# Reconstruct parallel case (if you ran in parallel)
reconstructPar

# Start ParaView
paraview
```

**Visualization tips:**
1. Open `domain.foam` file in ParaView
2. Apply the case
3. Select fields to visualize (p, U, zoneID)
4. Use "Slice" or "Clip" filters to see internal flow
5. Use "Glyph" filter to visualize velocity vectors
6. Animate through time steps to see pressure wave propagation

## Advanced Docker Usage

### Run with GUI Support (Linux with X11)

```bash
xhost +local:docker

docker run -it --rm \
  -v $(pwd):/workspace \
  -w /workspace/waterHammer \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  openfoam/openfoam11-paraview510 \
  /bin/bash
```

### Save Container State

If you want to preserve your work and installed packages:

```bash
# Create a named container
docker run -it \
  --name waterhammer_sim \
  -v $(pwd):/workspace \
  -w /workspace/waterHammer \
  openfoam/openfoam11-paraview510 \
  /bin/bash

# Exit and restart later
exit
docker start -ai waterhammer_sim
```

### Clean the Case

```bash
cd domain
./clean
cd ../vane
./clean
cd ..
```

## Simulation Parameters

### Key Settings

**Solver:** `overPimpleDyMFoam` (dynamic overset mesh PIMPLE solver)

**Time Control:**
- End time: 1.5 seconds
- Time step: 0.00025 seconds (adjustable)
- Max Courant number: 1

**Vane Motion:**
- Type: Oscillating linear motion
- Amplitude: 0.1 m (in y-direction)
- Frequency: 2.0 rad/s

**Fluid Properties:**
- Kinematic viscosity: 1e-06 m²/s (water)
- Flow: Laminar

**Mesh:**
- Domain: ~12,500 cells (500 x 25 x 1)
- Vane: Imported from UNV file
- Total (merged): ~13,000-15,000 cells

## Troubleshooting

### Issue: "Command not found" errors

**Solution:** Make sure you've sourced the OpenFOAM environment:
```bash
source /opt/openfoam11/etc/bashrc
```

### Issue: Mesh merge fails

**Solution:** Ensure the vane mesh is generated first:
```bash
cd vane
./meshUnv
cd ../domain
./mesh
```

### Issue: Permission denied on scripts

**Solution:** Make scripts executable:
```bash
chmod +x waterHammer/domain/mesh
chmod +x waterHammer/domain/run
chmod +x waterHammer/domain/clean
chmod +x waterHammer/vane/meshUnv
chmod +x waterHammer/vane/clean
```

### Issue: Simulation diverges

**Solution:**
- Check initial conditions in `domain/0.orig/`
- Reduce time step in `domain/system/controlDict`
- Check mesh quality: `checkMesh -allTopology -allGeometry`

### Issue: Docker container runs out of memory

**Solution:** Increase Docker memory limit:
- Docker Desktop: Settings → Resources → Memory (set to 4GB+)
- Linux: Check `docker stats` and adjust accordingly

## Expected Output

The simulation will generate:
- Time directories with results (0, 0.005, 0.01, 0.015, ...)
- Pressure wave propagation through the domain
- Velocity fields showing the effect of vane oscillation
- Log files for debugging

**Typical run time:**
- Serial: ~30-60 minutes (1500 time steps)
- Parallel (3 cores): ~15-30 minutes

## Visualizing Results

### Key Fields to Examine

1. **Pressure (p)**: Shows water hammer pressure waves
2. **Velocity (U)**: Flow field around oscillating vane
3. **zoneID**: Visualization of overset zones (0=domain, 1=vane)

### Animation

To create an animation of the pressure field evolution:
1. Load case in ParaView
2. Set color by pressure field
3. Adjust color scale range
4. File → Save Animation → Set parameters → OK

## References

- [OpenFOAM 11 Documentation](https://www.openfoam.org/version/11/)
- [OpenFOAM Docker Guide](https://openfoam.org/download/windows-docker/)
- [overPimpleDyMFoam Solver](https://www.openfoam.org/news/overset-mesh/)

## Additional Resources

- **Case Files Location:** `waterHammer/domain/` and `waterHammer/vane/`
- **CAD Files:** `waterHammer/cad/`
- **Results Videos:** `waterHammer/waterHammerPressure.mp4`

## Support

For issues specific to this case, please open an issue on the GitHub repository.

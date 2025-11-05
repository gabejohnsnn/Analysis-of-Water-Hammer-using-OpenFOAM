# Troubleshooting Guide - Water Hammer Simulation

This guide addresses common errors encountered when running the water hammer simulation in OpenFOAM 11.

## Fixed Issues (Latest Update)

### ✅ FIXED: "wrong token type - expected string" Error

**Error Message:**
```
--> FOAM FATAL IO ERROR:
wrong token type - expected string, found on line 17 the word 'overset'
```

**Solution:**
This has been fixed in the latest version. The library loading syntax in `controlDict` now correctly uses quotes:
```cpp
libs            ("overset");  // Correct
// NOT: libs (overset);       // Wrong - causes error
```

**If you cloned an older version:** Pull the latest changes from the repository.

### ✅ FIXED: topoSetDict Syntax Errors

**Error Message:**
```
keyword insidePoints is undefined in dictionary
```

**Solution:**
The `regionToCell` source in OpenFOAM 11 requires a `sourceInfo` subdictionary. This has been fixed in both domain and vane cases.

**Correct syntax:**
```cpp
source  regionToCell;
sourceInfo
{
    insidePoints ((0.001 0.001 0.001));
}
```

## Expected Warnings (Not Errors)

### ⚠️ changeDictionary Deprecation Warning

**Warning Message:**
```
--> FOAM Warning :
changeDictionary has been superseded by foamDictionary and is now deprecated.
```

**Status:** **EXPECTED - NOT AN ERROR**

**Explanation:**
- This is an informational warning only
- `changeDictionary` still works in OpenFOAM 11
- The simulation will run normally despite this warning
- You can safely ignore this warning

**Why not update?**
While `foamDictionary` is the newer tool, `changeDictionary` is more convenient for batch boundary condition updates. We may update the scripts in a future release.

## Common Runtime Errors

### Error: "Command not found" for OpenFOAM commands

**Symptoms:**
```
bash: blockMesh: command not found
bash: overPimpleDyMFoam: command not found
```

**Solution:**
You forgot to source the OpenFOAM environment:
```bash
source /opt/openfoam11/etc/bashrc
```

**To verify it's loaded:**
```bash
echo $WM_PROJECT_VERSION
# Should output: 11
```

### Error: "Cannot find file vane.unv"

**Symptoms:**
```
--> FOAM FATAL ERROR:
cannot open file vane.unv
```

**Solution:**
Make sure you're in the correct directory:
```bash
cd waterHammer/vane
ls vane.unv  # Should exist
./meshUnv
```

### Error: Mesh merge fails

**Symptoms:**
```
cannot find source case ../vane
```

**Solution:**
Run the vane mesh generation first:
```bash
# From waterHammer directory:
cd vane
./meshUnv
cd ../domain
./mesh
```

### Error: Permission denied on scripts

**Symptoms:**
```
bash: ./mesh: Permission denied
```

**Solution:**
Make scripts executable:
```bash
chmod +x waterHammer/domain/mesh
chmod +x waterHammer/domain/run
chmod +x waterHammer/domain/clean
chmod +x waterHammer/vane/meshUnv
chmod +x waterHammer/vane/clean
```

### Error: Simulation diverges (floating point exception)

**Symptoms:**
```
#0  Foam::error::printStack(Foam::Ostream&)
#1  Foam::sigFpe::sigHandler(int)
Floating point exception
```

**Possible Causes:**
1. Bad mesh quality
2. Time step too large
3. Incorrect boundary conditions

**Solutions:**

**Check mesh quality:**
```bash
checkMesh -allTopology -allGeometry | tee log.checkMesh
```

Look for:
- Non-orthogonality < 70
- Skewness < 4
- No negative volumes

**Reduce time step:**
Edit `domain/system/controlDict`:
```cpp
deltaT          0.0001;  // Instead of 0.00025
maxCo           0.5;     // Instead of 1
```

**Check initial conditions:**
Verify fields in `domain/0.orig/` have reasonable values.

### Error: Out of memory in Docker

**Symptoms:**
```
Killed
```

**Solution:**
Increase Docker memory allocation:
- **Docker Desktop:** Settings → Resources → Memory → Set to 4GB or more
- **Linux:** Check available memory with `docker stats`

## Mesh Generation Issues

### Warning: Low quality mesh elements

**Warning:**
```
***High aspect ratio cells found, Max aspect ratio: 100
```

**Action:**
- This is expected for 2D simulations (with 1 cell in z-direction)
- Check aspect ratio is reasonable in x-y plane
- If aspect ratio > 1000 in x-y plane, consider refining mesh

### Error: topoSet fails to find cells

**Symptoms:**
```
Created cellSet c0
    Number of cells = 0
```

**Solution:**
Check the `insidePoints` coordinate is actually inside the mesh:
```bash
# Check mesh bounds:
checkMesh | grep -A 5 "Bounding box"

# Update topoSetDict with point inside bounds
```

## Parallel Running Issues

### Error: MPI not available

**Symptoms:**
```
mpirun: command not found
```

**Solution (in Docker):**
The official OpenFOAM Docker images include MPI. Make sure you're using:
```bash
docker pull openfoam/openfoam11-paraview510
```

### Error: Decomposition fails

**Symptoms:**
```
Number of processor domains = 3
Maximum number of cells = 5000
```

**Solution:**
The mesh might be too small for 3 processors. Edit `domain/system/decomposeParDict`:
```cpp
numberOfSubdomains  2;  // Instead of 3

coeffs
{
    n           (2 1 1);  // Instead of (3 1 1)
}
```

Or run in serial instead:
```bash
overPimpleDyMFoam | tee log.run
```

## Visualization Issues

### ParaView doesn't start in Docker

**Linux with X11:**
```bash
# Enable X11 forwarding
xhost +local:docker

# Run container with display
docker run -it --rm \
  -v $(pwd):/workspace \
  -w /workspace/waterHammer \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  openfoam/openfoam11-paraview510 \
  /bin/bash
```

**macOS:**
Use XQuartz and set DISPLAY variable, or export results and visualize locally.

**Windows:**
Use VcXsrv or export results and use ParaView on Windows.

### Cannot open .foam file

**Solution:**
Create a dummy file for ParaView:
```bash
cd domain
touch case.foam
# Open case.foam in ParaView
```

## Getting More Information

### Enable detailed logging

Add to the beginning of your run:
```bash
set -x  # Enable bash debugging
```

### Check OpenFOAM version

```bash
foam
# Should show: OpenFOAM-11
```

### Verify case setup

```bash
foamListTimes        # List available time directories
foamInfo             # Show case information
```

### Check boundary conditions

```bash
# After running mesh:
foamDictionary -entry boundaryField -value 0/U
foamDictionary -entry boundaryField -value 0/p
```

## Still Having Issues?

1. **Check log files:**
   - `domain/log.blockMesh`
   - `domain/log.changeDictionary`
   - `domain/log.mergeMeshes`
   - `domain/log.setFields`
   - `domain/log.decomposePar`
   - `domain/log.run`

2. **Verify file structure:**
   ```bash
   tree waterHammer -L 2
   ```

3. **Check you have the latest fixes:**
   ```bash
   git pull
   ```

4. **Clean and restart:**
   ```bash
   cd domain
   ./clean
   cd ../vane
   ./clean
   cd ..
   # Then start over with ./meshUnv
   ```

5. **Report the issue:**
   Include the full error message and relevant log files when reporting issues on GitHub.

## Quick Reference: Correct Workflow

```bash
# 1. Start Docker container
docker run -it --rm -v $(pwd):/workspace -w /workspace/waterHammer \
  openfoam/openfoam11-paraview510 /bin/bash

# 2. Source OpenFOAM
source /opt/openfoam11/etc/bashrc

# 3. Generate vane mesh
cd vane
./meshUnv
cd ..

# 4. Generate domain mesh and merge
cd domain
./mesh

# 5. Run simulation
./run

# 6. Post-process
reconstructPar
paraview
```

## Summary of Key Fixes

✅ Library loading: Use `libs ("overset");` with quotes
✅ topoSetDict: Use `sourceInfo` subdictionary
⚠️ changeDictionary warning: Expected and harmless
✅ All syntax updated for OpenFOAM 11 compatibility

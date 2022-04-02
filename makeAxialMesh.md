makeAxialMesh
=============
Reference:
1. [cfd-china thread](https://www.cfd-china.com/topic/787/%E9%97%B2%E6%9D%A5%E6%97%A0%E4%BA%8B-%E5%81%9A%E4%B8%80%E4%B8%AAaxial-mesh%E7%9A%84%E6%95%99%E7%A8%8B%E5%90%A7)
2. [A Japanese blog](http://penguinitis.g1.xrea.com/study/OpenFOAM/axisymmetric/axisymmetric.html)
3. [OpenFOAM wiki](http://penguinitis.g1.xrea.com/study/OpenFOAM/axisymmetric/axisymmetric.html)
4. [Jianshu blog](https://www.jianshu.com/p/0306c8830570)

## Source code
Download `makeAxialMesh` for OpenFOAM `2.x` versions from [here](https://openfoamwiki.net/images/5/5e/MakeAxialMesh_2.x.tar.gz).  
Compile:
```sh
./Allwmake
```

## How to use
### 1. Create a 2D axisymmetric mesh using ICEM
Make sure the axis is set in the boundary conditions.
### 2. Convert ICEM 2d mesh to OpenFOAM
Use `fluentMeshToFoam`:
```sh
fluentMeshToFoam fluent.msh
```
Note: Don't use scale option! Use mm to prevent roundOff error.  
### 3. Convet the mesh type
Use `makeAxialMesh`:
```sh
makeAxialMesh -axis AXIS -wedge frontAndBackPlanes -offset 1e-6 -overwrite
# Use flattenMesh to prevent checkMesh error
flattenMesh
```
Please note! The config `-offset 1e-6` is used to fix this error report:
```
Changing patch types 
 Changing AXIS to symmetryPlane
 Changing frontAndBackPlanes_pos to wedge
 Changing frontAndBackPlanes_neg to wedge



--> FOAM FATAL ERROR: 
Symmetry plane 'AXIS' is not planar.
At local face at (-0.04525201889605017 0 0) the normal (0 -1 0) differs from the average normal (0 -0.4972222222222222 0) by 0.2527854938271605
Either split the patch into planar parts or use the symmetry patch type

    From function symmetryPlanePolyPatch::n()
    in file meshes/polyMesh/polyPatches/constraint/symmetryPlane/symmetryPlanePolyPatch.C at line 64.

FOAM exiting
```
### 4. Collapse mesh
Create `collapseDict` file in `system` folder:
```c++
FoamFile
{
    version         2.0;
    format          ascii;
    class           dictionary;
    location        "system";
    object          collapseDict;
}

collapseEdgesCoeffs
{
    minimumEdgeLength   1e-6;

    maximumMergeAngle   179.5;
}
```
Run this command:
```sh
collapseEdges -overwrite
# Scale mesh from mm to m
transformPoints -scale "(1e-3 1e-3 1e-3)"
```
See Ref[2] for more information.

### 5. Check
Check mesh and boundary patch names in `constant/polyMesh/boundary`.  
```sh
checkMesh 
```


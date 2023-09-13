Merge two different mesh files
================================
This tutorial shows how to merge two mesh files genreated by two different softwares.

```plain
--------- 
|       |  
|       ----   ---------------
|    A     |   |      B      |
|          |   |             |
-----------    ---------------
           ^   ^
         interface
```

For example, a body can be divided to part A and part B. They are connected through the interface: inter-A and inter-B.

Use Fluent meshing to generate poly-hexcore mesh for part A. Use ICEM to generate struct hexal mesh for part B. The interface of A and B have different mesh. (Version 22R1)

## 1. Combine two mesh sets
- `Fluent <solution mode>` -> `Read mesh A`
- `Domain` -> `Zones` -> `Append (append case file)` -> `Select part B mesh file` -> `Click ok` -> `ignore the warnings`
- `Domain` -> `Zone` -> `Combine (merge)` -> `fluid (select two fluid zones)` -> `Merge` 
- `File` -> `Write` -> `Case`, for example `test.cas`
## 2. Get mesh from the case file
- `Fluent <meshing mode>`
- `File` -> `Read (case)` -> `select test.cas`
- Use TUI to export mesh file
```sh
Meshing> file
Meshing/file> file-format
Write binary files? [yes] no
Meshing/file> write-mesh test.msh
```
## 3. Convert the merged mesh to OpenFOAM

```sh
fluen3DMeshToFoam test.msh -scale 0.001
```
Edit the `constant/boundary` file, change `inter-A` and `inter-B` settings to:
```cpp
    inter-A
    {
        nFaces          xxxxxx;
        startFace       xxxxxx;

        type            cyclicAMI;
        inGroups        1(cyclicAMI);
        matchTolerance  0.0001;
        neighbourPatch  inter-B;
        transform       noOrdering;

    }
    inter-B
    {
        nFaces          xxxxxx;
        startFace       xxxxxx;

        type            cyclicAMI;
        inGroups        1(cyclicAMI);
        matchTolerance  0.0001;
        neighbourPatch  inter-B;
        transform       noOrdering;
    }
```
Boundary conditions for any variables:
```cpp
    inter-A
    {
        type            cyclicAMI;
    }
    inter-B
    {
        type            cyclicAMI;
    }
```

postProcessing
==============
1. Sampling cut-plane's average values during simulation

```cpp
    Z15mm
    {
        type            faceSource;
        functionObjectLibs ("libfieldFunctionObjects.so");

        enabled         true;
        outputControl   timeStep;
        outputInterval  1;
        
        // Show result in terminal
        log             true;

        // Will write surface values in vtk/raw if true
        valueOutput     false; 
        surfaceFormat   vtk;

        writeFields     no;

        source          sampledSurface;

        // Surface definition (same as surfaceSampling)
        sampledSurfaceDict
        {
            type            plane;
            basePoint       (0 0 0.062);
            normalVector    (0 0 1);
        }
        
        // Operation of the surface values
        operation       areaAverage;

        // Weighted average (only valid for patches), here is mass-weighted average
        //operation       weightedAverage;
        //weightField     phi;

        fields ( p U T);
    }
```

2. Surface sammling during simulation

```cpp
    surfaceSampling
    {
        type surfaces;
        functionObjectLibs ("libsampling.so");
        enabled         true;
        outputControl   timeStep;
        outputInterval	500;

        interpolationScheme cell;
        surfaceFormat vtk;
        // Fields to be sampled
        fields
        (
            Z T C U OH
        );

        surfaces
        (
             middleCut
             {
                type            plane;
                interpolate	true;
                triangulate	false;
                normalVector	(1 0 0);
                basePoint	(0.0001 0 0);
             }
        );
    }
```

myFoamLearning
==============
Maybe some useful learning material for OpenFOAM.

## 1. codedFixedValue   
For some cases there may need an inlet velocity profile. Luckily OF provides a codedFixedValue inlet condition for this situation.  
Check [this blog](http://sourceflux.de/blog/the-codedfixedvalue-boundary-condition/) in [sourceflux](http://sourceflux.de) for more information.  
Here is a simple example for Sandia D jet inlet velocity profile. And the code is modified from the blog mentioned above.  
```C++
    JET
    {
        //type            fixedValue;
        //value           uniform (49.6 0 0 );
        type            codedFixedValue;
        value           uniform (0 0 0);
        redirectType    velocityProfile;

        code
        #{
            const fvPatch& boundaryPatch = patch(); 
            const vectorField& Cf = boundaryPatch.Cf(); 

            vectorField& field = *this; 
            scalar min = 0.501;  
            scalar max = 0.751; 
            double r_d[11] = {0.0,  0.0694,  0.1388,  0.2083,  0.2777,  0.3472,  0.4166,  0.4861,  0.5000, 0.5555, 0.6250};
            double u[11] = {62.95,  62.54,  61.36,  59.21,  56.73,  53.34,  48.80,  41.99,  0.0, 3.45, 11.46};

            forAll(Cf, faceI)
            {
               if (
                    (Cf[faceI].z() > min) &&
                    (Cf[faceI].z() < max) &&
                    (Cf[faceI].y() > min) &&
                    (Cf[faceI].y() < max) 
                  )
                {
                    field[faceI] = vector(0.1, 0, 0);
                }
                scalar r = sqrt(sqr(Cf[faceI].y())+sqr(Cf[faceI].z()));
                for(int i=0; i<11; i++)
                {
                    if (r<r_d[i+1]*0.0072 && r >= r_d[i]*0.0072)
                    {
                        // interpolate
                        scalar utemp = (u[i+1]-u[i])/(r_d[i+1]-r_d[i])*(r/0.0072-r_d[i])+u[i];
                        field[faceI] = vector(utemp,0,0);
                        //Info<<"This is test io:"<<'\t'<< r <<'\t'<<r/0.0072<<'\t'<<i<<'\t'<<r_d[i]<<'\t'<<utemp<<endl;
                        break;
                    }
                }

            }
        #};
    }
```

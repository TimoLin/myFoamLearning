Lagrangian Library
==================

## Atomization model
### 1. LISA model

Ref1:   
```
    P.K. Senecal, D.P. Schmidt, I. Nouar, C.J. Rutland, R.D. Reitz, M. Corradini
    "Modeling high-speed viscous liquid sheet atomization"
    International Journal of Multiphase Flow 25 (1999) pags. 1073-1097
```
Code: 
```cpp
    if (volFlowRate < SMALL)
    {
        return;
    }

    scalar tau = 0.0;
    scalar dL = 0.0;
    scalar k = 0.0;

    // update atomization characteristic time
    tc += dt;
    
    /// Gas weber number
    scalar We = 0.5*rhoAv*sqr(Urel)*d/sigma;
    scalar nu = mu/rho;

    scalar Q = rhoAv/rho;

    vector diff = pos - injectionPos;
    scalar pWalk = mag(diff);
    scalar traveledTime = pWalk/Urel;

    scalar h = diff & injectorDirection_;
    scalar delta = sqrt(sqr(pWalk) - sqr(h));
    
    /// Sheet thickness Eqn(47) (not exactly the same)
    scalar hSheet = volFlowRate/(constant::mathematical::pi*delta*Urel);

    // update drop diameter
    d = min(d, hSheet);

    if (We > 27.0/16.0)
    {
        /// Short wave mode

        scalar kPos = 0.0;
        /// Initial guess of k
        scalar kNeg = Q*sqr(Urel)*rho/sigma;

        scalar derivPos = sqrt(Q*sqr(Urel));

        /// Solution difference of the firt order derivative of Eqn(33)
        scalar derivNeg =
            (
                8.0*sqr(nu)*pow3(kNeg)
              + Q*sqr(Urel)*kNeg
              - 3.0*sigma/2.0/rho*sqr(kNeg)
            )
          / sqrt
            (
                4.0*sqr(nu)*pow4(kNeg)
              + Q*sqr(Urel)*sqr(kNeg)
              - sigma*pow3(kNeg)/rho
            )
          - 4.0*nu*kNeg;

        scalar kOld = 0.0;

        /// Solve the first order derivation of Eqn(33) iteratively 
        for (label i=0; i<40; i++)
        {
            k = kPos - (derivPos/((derivNeg - derivPos)/(kNeg - kPos)));

            scalar derivk =
                (
                    8.0*sqr(nu)*pow3(k)
                  + Q*sqr(Urel)*k
                  - 3.0*sigma/2.0/rho*sqr(k)
                )
              / sqrt
                (
                    4.0*sqr(nu)*pow4(k)
                  + Q*sqr(Urel)*sqr(k)
                  - sigma*pow3(k)/rho
                )
              - 4.0*nu*k;

            if (derivk > 0)
            {
                derivPos = derivk;
                kPos = k;
            }
            else
            {
                derivNeg = derivk;
                kNeg = k;
            }

            if (mag(k - kOld)/k < 1e-4)
            {
                break;
            }

            kOld = k;
        }
        
        /// calculate OmegaS (most unstable wave growth rate)
        scalar omegaS =
          - 2.0*nu*sqr(k)
          + sqrt
            (
                4.0*sqr(nu)*pow4(k)
              + Q*sqr(Urel)*sqr(k)
              - sigma*pow3(k)/rho
            );
        
        /// breakup time
        tau = cTau_/omegaS;
        
        /// breakup ligament diameter
        dL = sqrt(8.0*d/k);
    }
    else
    {
        /// Long wave mode

        /// Eqn(38) derived from inviscid condition
        k = rhoAv*sqr(Urel)/(2.0*sigma);
        
        /// No source paper. Need a reference!
        scalar J = 0.5*traveledTime*hSheet;
        /// Breakup time
        tau = pow(3.0*cTau_, 2.0/3.0)*cbrt(J*sigma/(sqr(Q)*pow4(Urel)*rho));

        /// breakup ligament diameter
        dL = sqrt(4.0*d/k);
    }

    /// Most unstable wavenumber of ligament
    scalar kL = 1.0/(dL*sqrt(0.5 + 1.5 * mu/sqrt(rho*sigma*dL)));

    /// Breakup droplets diameter
    scalar dD = cbrt(3.0*constant::mathematical::pi*sqr(dL)/kL);

    scalar atmPressure = 1.0e+5;

    scalar pRatio = pAmbient/atmPressure;

    dD = dD*pow(pRatio, lisaExp_);

    scalar pExp = 0.135;

    //  modifing dD to take account of flash boiling
    dD = dD*(1.0 - chi*pow(pRatio, -pExp));
    /// Breakup length
    scalar lBU = Cl_ * mag(Urel)*tau;

    if (pWalk > lBU)
    {
        /// Ligament breakup to droplets!
        scalar x = 0;

        switch (SMDMethod_)
        {
            case method1:
            {
                #include "LISASMDCalcMethod1.H"
                break;
            }
            case method2:
            {
                #include "LISASMDCalcMethod2.H"
                break;
            }
        }

        //  New droplet properties
        liquidCore = 0.0;
        /// New diameter
        d = x;
        tc = 0.0;
    }
```

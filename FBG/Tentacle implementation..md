
FBG:
E = 7GPa glass (from FBGS Website)
E = 2GPa (ORMOCER Coating). From [FBGS Website](https://fbgs.com/faq/what-is-the-youngs-modulus-of-the-dtg-sensor/).

EcoFlex 0030:
E circa 100MPa



# Composite Properties

Note that I (second moment of area) is not the same as I (inertia). Henceforth in this document I refers to the second moment of area.

For a solid circle:

$I = \frac{\pi d^4 }{64}$ ​

For an annulus:

$I = \frac{\pi (d^4_{outer} - d^4_{inner}) }{64}$ 


The Cross Section of the tentacle is given:
![[TentacleCS.png]]
Outer: Circa 100MPa, mid circa 2GPa, inner circa 70GPa

Outer: 3mm diameter, Mid, 0.2mm diameter, Inner, 0.13mm diameter

```
close all;

clear;

clc;

% computeCompositeProperties calculates the composite Young's modulus and

% second moment of area for a tentacle with an embedded FBG.

% Constants

E_silicone = 100e6; % Young's modulus of silicone (Pa)

E_outer = 2e9; % Young's modulus of FBG outer core (Pa)

E_inner = 70e9; % Young's modulus of FBG inner core (Pa)

D_FBG_outer = 0.2; % Diameter of FBG outer core (mm)

D_FBG_inner = 0.13; % Diameter of FBG inner core (mm)

D_OverallTentacle = 3; % Diameter of complete tentacle.

% Calculate second moments of area (I) for each component

I_total = (pi/64) * D_OverallTentacle^4; % Total I for the tentacle

I_silicone = (pi/64) * (D_OverallTentacle^4 - D_FBG_outer^4); % I for silicone part

I_outer = (pi/64) * (D_FBG_outer^4 - D_FBG_inner^4); % I for FBG outer core

I_inner = (pi/64) * D_FBG_inner^4; % I for FBG inner core

% Calculate EI (E multiplied by I) for each component

EI_silicone = E_silicone * I_silicone;

EI_outer = E_outer * I_outer;

EI_inner = E_inner * I_inner;

% Sum up EI values to get the total EI

EI_total = EI_silicone + EI_outer + EI_inner

% Calculate the composite Young's modulus

E_composite = EI_total / I_total
```
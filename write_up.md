# The C++ Project Write Up #

####Project Requirement:
1. Cloned the repository and gotten familiar with the C++ environment as outlined in C++ Setup.    
  
2. Complete each of the scenarios outlined in the C++ project readme. This will involve implementing and tuning   controllers incrementally:
 - Body rate and roll/pitch control (scenario 2)
 - Position/velocity and yaw angle control (scenario 3)
 - Non-idealities and robustness (scenario 4)   
 
3. Tune controller and make sure it works to successfully meet each of the evaluations in each scenario.    

The architecture of the controller consists of altitude controller, position controller, and attitude controller like folowing:
![](./images/3d_control.png)    

#### Parameter Tuning (scenario 1)
In scenario 1, I modified the quad mass in `QuadControlParams.txt` line [12](./cpp/QuadControlParams.txt#L12) 

[![Youtube Demo:](https://img.youtube.com/vi/oNuX0w8yZDE/0.jpg)](https://www.youtube.com/watch?v=oNuX0w8yZDE)   

```
PASS: ABS(Quad.PosFollowErr) was less than 0.500000 for at least 0.800000 seconds
Program ended with exit code: 0
```
### Body rate and roll/pitch control (scenario 2) ###   

The attitude controller breaks down into smaller controllers responsible for roll-pitch, yaw, and body rate like below:
![](./images/attitude_control.png)

 In scenario 2, I see a quad above the origin. It is created with a small initial rotation speed about its roll axis. Controller will need to stabilize the rotational motion and bring the vehicle back to level attitude.
 
1. Implement body rate control

 - implemented the code in the function `GenerateMotorCommands().`The code is in `QuadControl.cpp` line [56 to 96](/src/QuadControl.cpp#L56-L96) 
 - implemented the code in the function `BodyRateControl().`The code is in `QuadControl.cpp` line [98 to 122](/src/QuadControl.cpp#L98-L122) 
 - Tune `kpPQR` in `QuadControlParams.txt` to kpPQR = `30, 30, 5` to get the vehicle to stop spinning quickly but not overshoot.

 The rotation of the vehicle about roll (omega.x) get controlled to 0 while other rates remain zero. Note that the vehicle will keep flying off quite quickly, since the angle is not yet being controlled back to 0.  Also note that some overshoot will happen due to motor dynamics!

If you come back to this step after the next step, you can try tuning just the body rate omega (without the outside angle controller) by setting `QuadControlParams.kpBank = 0`.

2. Implement roll / pitch control
We won't be worrying about yaw just yet.

 - implemented the code in the function `RollPitchControl()`. The code is in `QuadControl.cpp` line [125 to 166](/src/QuadControl.cpp#L125-L166)  
 - Tune `kpBank` in `QuadControlParams.txt` to minimize settling time but avoid too much overshoot.

[![Youtube Demo:](https://img.youtube.com/vi/ZAcTQpNt_sg/0.jpg)](https://www.youtube.com/watch?v=ZAcTQpNt_sg)

```
Simulation #11 (../config/2_AttitudeControl.txt)
PASS: ABS(Quad.Roll) was less than 0.025000 for at least 0.750000 seconds
PASS: ABS(Quad.Omega.X) was less than 2.500000 for at least 0.750000 seconds
```
### Position/velocity and yaw angle control (scenario 3) ###
    
  - implemented the code in the function `LateralPositionControl()`. The code is in `QuadControl.cpp` line [207 to 251](/src/QuadControl.cpp#L207-L251).  
  - implemented the code in the function `AltitudeControl()`. The code is in `QuadControl.cpp` line [168 to 204](/src/QuadControl.cpp#L168-L204).  
  - tune parameters kpPosZ and kpPosZ
  - tune parameters kpVelXY and kpVelZ
  
After tuning angle rate gains `kpPQR = 50, 50, 10` successful pass scenario 3 & 4

[![Youtube Demo:](https://img.youtube.com/vi/RRuN89Ynjrk/0.jpg)](https://www.youtube.com/watch?v=RRuN89Ynjrk)

```
Simulation #18 (../config/3_PositionControl.txt)
PASS: ABS(Quad1.Pos.X) was less than 0.100000 for at least 1.250000 seconds
PASS: ABS(Quad2.Pos.X) was less than 0.100000 for at least 1.250000 seconds
PASS: ABS(Quad2.Yaw) was less than 0.100000 for at least 1.000000 seconds
```
### Non-idealities and robustness (scenario 4) ###

 - The green quad has its center of mass shifted back
 - The orange vehicle is an ideal quad
 - The red vehicle is heavier than usual
 - Edit `AltitudeControl()` to add basic integral control to help with the different-mass vehicle.The code is in `QuadControl.cpp` line [168 to 204](/src/QuadControl.cpp#L168-L204)

After tuning angle rate gains `kpPQR = 50, 50, 10` successful pass scenario 3 & 4

[![Youtube Demo:](https://img.youtube.com/vi/7zBqXAWNi4E/0.jpg)](https://www.youtube.com/watch?v=7zBqXAWNi4E)   

```
Simulation #4 (../config/4_Nonidealities.txt)
PASS: ABS(Quad1.PosFollowErr) was less than 0.100000 for at least 1.500000 seconds
PASS: ABS(Quad2.PosFollowErr) was less than 0.100000 for at least 1.500000 seconds
PASS: ABS(Quad3.PosFollowErr) was less than 0.100000 for at least 1.500000 seconds
```
### Tracking trajectories ###

[![Youtube Demo:](https://img.youtube.com/vi/eetnNu0gJfg/0.jpg)](https://www.youtube.com/watch?v=eetnNu0gJfg)

```
Simulation #3 (../config/5_TrajectoryFollow.txt)
PASS: ABS(Quad2.PosFollowErr) was less than 0.250000 for at least 3.000000 seconds
```

**PC Login**
User = stormlab
Password = stormlab


**Coil Information** 
MAX demands = 18 mT in x,  22 mT in y, 18 mT in z
Max step demand = 10 mT


**Running Manual Control**

1. Open terminal. Navigate to the ros workspace
```
cd ros_ws/
```

2. Navigate to the source folder

```
cd src/
```

3. Navigate somewhere else?
```
cd ../../
```

4. Launch the ros coils for the manual control
```
roslaunch ros_coils field_manual_op.launch
```


5. Enter x mT, press enter. Enter y mT, press enter. Enter z mT, press enter.

6. When finished
```
shutdown
```

7. Ctrl c to close process.
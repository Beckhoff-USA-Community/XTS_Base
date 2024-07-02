
# Basic Diagnostics

Project Diagnostics in this documentation focuses on troubleshooting during active project development, and does not provide functionality for Alarm Handling on a production system. Those functionalities must be added to this project on an application-specific basis.

Several tools are made available in this project file to make development as quick and easy as possible, such as:

- Object status information
- Function block error codes
- Mover activity Event Log
- User Log Events

As a starting point, always be sure to investigate the object status outputs in the variable viewer inside TwinCAT. Here we can immediately see a Mover which is:

- Currently destined for Station #0
- Most recently commanded via Position Trigger #2
- Has no active errors

> For more advanced troubleshooting steps, see the additional categories to the left.

![Mover Status](../../Images/MoverStatus.png)

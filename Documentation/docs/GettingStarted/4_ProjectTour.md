
# Project Tour

## Basics

This project is roughly organized into *Provided Objects* and *Implementation Placeholders*.

### Provided Objects

A suite of ready-to-use function blocks / objects are made available by this project and located in the "XTS" directory within the PLC project. These objects are:

- **Mover**
- **MoverList**
- **PositionTrigger**
- **Station**
- **Track**
- **Zone**

Additionally there are three "administrative" object definitions which handle some of the background tasks & quality-of-life enhancements. These include:

- **Mediator**
- **ErrorMover**
- **Objective**

Finally, we provide a single function block / object which houses the rest:

- **FB_XTS**

Detailed documentation for all of these objects is available in the *Code Reference* tab, above.


!!! tip "Intent"
    All of these objects are intended to be instantiated and used *as-is* without modifying their internal implementations- simply by calling the public methods and properties. However, for advanced solutions these objects are all provided **completely open source** for you to customize and extend as needed.

### Implementation Placeholders

A Main State Machine is provided in the **MAIN (PRG)**. Underneath this POU are ACTIONS (*Initializing*, *Recoverying*, *RecoverOneshot*, and *StationLogic*), which can be places to add your application-specific logic. Some basic motion code is already included which makes this project ready-to-run, but the intent is for you to overwrite these sections according to your needs.

The calling architecture itself is also an example, but can be manipulated to meet your organization's preferred coding methodology.
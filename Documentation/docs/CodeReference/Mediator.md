
# Mediator

## Broad Concept

Most mover actions can be represented as a combination of a Station, Position Trigger or Zone and a mover. However there are some system-level concepts that require knowledge of all of the stations, movers, zones, etc. The Mediator fulfills this role.

By using the system level Mediator functionality such as next and previous movers can be realized, and tasks that need to be called once for the entire system can be managed such as locating Mover 1 at startup.

Finally, the Mediator handles calling the necessary .Cyclic() methods on all objects within the system, simplifying code needed to be managed by the end user.

## Configuration

Code already exists to make the mediator aware of all [Movers](Mover.md), [MoverLists](MoverList.md), [PositionTriggers](PositionTrigger.md), [Stations](Station.md), [Zones](Zone.md) and [Tracks](Track.md). It is executed at the appropriate states in the MAIN program. However, the mediator must be made aware of the number of each object. This is accomplished in the global variable list GVL.

![Global Variable List](../Images/GlobalVarsMediator.png)

Update the count of each object according to the needs of the system in this list. It's recommended to oversize these arrays so that during runtime debugging additional stations, zones or position triggers can be added without the need to stop and recompile the PLC program.

## Methods

### Cyclic

*Mediator.Cyclic()*

> Required to be called once per scan. The Mediator will then call all necessary .Cyclic() methods for the objects that are registered to it.

### Other Methods

The following methods are used internally by the Mediator and are not intended for direct access.

- AddMover
- AddMoverList
- AddPositionTrigger
- AddStation
- AddTrack
- AddZone

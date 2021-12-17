
# Objectives

## Overview

*Objective* is an umbrella term used by the project, that includes the following objects:

- Stations
- Zones
- SpeedTriggers
- PositionTriggers
- MoverLists

Each *Objective* defines a set criteria that a Mover can either fulfill or not at any given point. As an example, a Mover fulfills the criteria of a *Station* when it is parked at the Station's configured track position.

When a Mover satisfies the requirements of the *Objective*, the *Objective* provides a Reference to the Mover through which new Mover commands can be issued.

## Common Methods

The objects listed above all share some common methods, which are implemented in the base parent *Objective* object.

### RegisterMover

*RegisterMover( NewMover : Mover )*

> Adds a Mover to the list of Tracked Movers that the objective is currently monitoring. If the input Mover has already been added to the Tracked Movers list, the method call is ignored.

```javascript
Mover[1].MoveToPosition( Station[1].TrackPosition );
Station[1].RegisterMover( Mover[1] );
```

The code above is similar in functionality to:

```javascript
Mover[1].MoveToStation( Station[1] );
```

Stations include some unique features regarding mover registration. See [Station Object](Station.md) for more details.

---

### UnregisterMover

*RegisterMover( ExistingMover : Mover )*

> Removes a Mover from the list of Tracked Movers that the objective is currently monitoring. If the input Mover is not already tracked by the objective, the method call is ignored.

```javascript
Mover[1].MoveToStation( Station[1] );
Station[1].UnregisterMover( Mover[1] );
```

Because the Mover would not be registered with the Station when it arrives, the Station would not report that the Mover is InPosition. The code above is functionally identical to:

```javascript
Mover[1].MoveToPosition( Station[1].TrackPosition );
```

---

### UnregisterCurrent

*UnregisterCurrent( )*

> Automatically unregisters whatever mover is listed as CurrentMover for the Objective

Here the MoverInPosition status output changes. The mover is still physically located at the Station position, but since it is no longer registered, it cannot report InPosition.

```javascript
bCheck	:= Station[1].MoverInPosition;	// returns TRUE
Station[1].UnregisterCurrent();
bCheck	:= Station[1].MoverInPosition;	// returns FALSE
```

```javascript
IF PositionTrigger[1].MoverPassedPosition THEN
	PositionTrigger[1].CurrentMover.SetVelocity( 200 );
	PositionTrigger[1].UnregisterCurrent();
END_IF
```

---

### UnregisterAll

*UnregisterAll()*

> Automatically unregisters every mover from the Tracked Movers list

```javascript
IF xClearTrigger THEN
	PositionTrigger[2].UnregisterAll();
	xClearTrigger	:= FALSE;
END_IF
```

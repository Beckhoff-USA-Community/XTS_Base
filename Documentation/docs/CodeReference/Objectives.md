
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

---

## Invalid References

Objectives that return References to MoverLists or Movers have the potential to return *invalid* references. For example, a Station with no mover present can't possibly return a CurrentMover, because by definition no CurrentMover exists! To prevent this, ensure that all *CurrentMover* and *CurrentMoverList* output properties are only evaluated when a *MoverInPosition*, *MoverPassedPosition*, *MoverInVelocity*, etc. flag is true for the relevant Objective.

```javascript
// There is no CurrentMover at the Station, so this code will not operate correctly!!
IF Station[1].MoverInPosition = FALSE THEN
	Station[1].CurrentMover.MoveToPosition( 200 );
END_IF
```

Typically, calling methods on these *invalid references* would result in a Pagefault and halt the XAR. However this outcome can be frustrating and slows development. Instead, an imposter **ErrorMover** object is returned in these circumstances as a quality-of-life improvement. ErrorMovers replace all the method functionality of standard Movers and will instead generate Errors in TwinCAT. For example:

![ErrorMover](../Images/PagefaultPrevention.png)

---

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

Here the MoverInPosition status output changes. The mover is still physically located at the Station position, but since it is no longer registered, it cannot report MoverInPosition.

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

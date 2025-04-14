# Objectives

*Objective* is an ABSTRACT base class used by the project, whose child objects include the following:

- [Stations](Station.md)
- [Zones](Zone.md)
- [PositionTriggers](PositionTrigger.md)
- [MoverLists](MoverList.md)
- [Tracks](Track.md)

Each *Objective* defines a set criteria that a Mover can either fulfill or not at any given point. As an example, a Mover fulfills the criteria of a *Station* when it is parked at the Station's configured track position.

When a Mover satisfies the requirements of the *Objective*, the *Objective* provides a Reference to the Mover through which new Mover commands can be issued.

## Common Methods

The objects listed above all share some common methods, which are implemented in the parent *Objective* base class.

### RegisterMover

*RegisterMover( NewMover : Mover )*

> Adds a Mover to the list of Tracked Movers that the objective is currently monitoring. If the input Mover has already been added to the Tracked Movers list, the method call is ignored.

```javascript
Mover[1].MoveToPosition( Station[1].Position );
Station[1].RegisterMover( Mover[1] );
```

The code above is similar in functionality to:

```javascript
Mover[1].MoveToStation( Station[1] );
```

Stations include some unique features regarding mover registration. See [Station Object](Station.md) for more details.


### RegisterMoverList

*RegisterMoverList( NewMoverList : MoverList )*

> Provides the same functionality as [RegisterMover](#registermover), but for a group of movers represented as a [MoverList](MoverList.md).

```javascript
MoverListA.RegisterMoverList(MoverListB);
```

### UnregisterMover

*RegisterMover( ExistingMover : Mover )*

> Removes a Mover from the list of Tracked Movers that the objective is currently monitoring. If the input Mover is not already tracked by the objective, the method call is ignored.

```javascript
Mover[1].MoveToStation( Station[1] );
Station[1].UnregisterMover( Mover[1] );
```

Because the Mover would not be registered with the Station when it arrives, the Station would not report that the Mover is InPosition. The code above is functionally identical to:

```javascript
Mover[1].MoveToPosition( Station[1].Position );
```

### UnregisterCurrent

*UnregisterCurrent( )*

> Automatically unregisters whatever mover is listed as CurrentMover for the Objective

Here the MoverInPosition status output changes. The mover is still physically located at the Station position, but since it is no longer registered, it cannot report MoverInPosition.

```javascript
bCheck	:= Station[1].MoverInPosition;	// returns TRUE
Station[1].UnregisterCurrent();
bCheck	:= Station[1].MoverInPosition;	// returns FALSE
```
Position triggers use `UnregisterCurrent` to indicate to the trigger that any processing that needs to be done with this mover has been completed.

```javascript
IF PositionTrigger[1].MoverPassedPosition THEN
	PositionTrigger[1].CurrentMover.SetVelocity( 200 );
	PositionTrigger[1].UnregisterCurrent();
END_IF
```

### UnregisterAll

*UnregisterAll()*

> Automatically unregisters every mover from the Tracked Movers list

```javascript
IF xClearTrigger THEN
	PositionTrigger[2].UnregisterAll();
	xClearTrigger	:= FALSE;
END_IF
```

## Properties

### CurrentMover

*REFERENCE TO Mover*

Current mover is the Reference output through which Movers can be addressed contextually via an Objective. Let's look at a simple example:

```javascript
Station[3].CurrentMover;	// is equivalent to...
Mover[4];					// as long as Mover #4 is parked at Station#3
```

And as a Reference, this *CurrentMover* output accepts any Method call instructions that a base Mover object could. E.g.:

```javascript
Station[3].CurrentMover.SetVelocity( 2000 );	// is equivalent to...
Mover[4].SetVelocity( 2000 );	// so long as the the Mover is parked as above
```

### TrackedMoverCount
*USINT*
> Returns the number of movers being tracked by this objective.

### TrackedMovers
*ARRAY OF POINTER TO Mover*
>Returns an array of pointers to all the movers tracked by this object.

Note that this returns a pointer. To access properties or methods of a mover the pointer must be dereferenced with the `^` (caret) operator such as `Objective.TrackedMovers[0]^.TrackInfo.TrackPosition`.


# Project Overview

## Broad Concept

The core concept driving the project structure is that it is useful to address movers **contextually** as opposed to **directly**.

To illustrate, consider a scenario of recovering from an E-Stop where a robot is positioned directly in the path of the movers. In this instance, a first step might be to command any movers *downstream* of the robot to move further downstream, and movers *upstream* of the robot should be commanded further upstream in order to create some space for the arm to reset.

As a programmer, one wouldn't seek to specifically command Mover #4 and Mover #5 to carry out those instructions because during the next occurrence it may be an entirely different pair of movers in this situation. Instead, the commands are issued to *whichever movers are closest to the robot right now*.

In this sense, we take a list of all movers on the track system, and apply *filters* to help select only the movers we care to command. These filters are applied via a family of objects called Objectives.


## Objectives

*Objective* is an umbrella term used by the project, that includes the following objects:

- Stations
- Zones
- PositionTriggers
- MoverLists

Each *Objective* defines a set criteria that a Mover can either fulfill or not at any given point. As an example, a Mover fulfills the criteria of a *Station* when it is parked at the Station's configured track position.

When a Mover satisfies the requirements of the *Objective*, the *Objective* provides a Reference to the Mover through which new Mover commands can be issued.

## .CurrentMover

*CurrentMover* is the Reference output through which Movers can be addressed contextually via an Objective. Let's look at a simple example:

```javascript
Station[3].CurrentMover;	// is equivalent to...
Mover[4];					// as long as Mover #4 is parked at Station#3
```

And as a Reference, this *CurrentMover* output accepts any Method call instructions that a base Mover object could. E.g.:

```javascript
Station[3].CurrentMover.SetVelocity( 2000 );	// is equivalent to...
Mover[4].SetVelocity( 2000 );	// so long as the the Mover is parked as above
```


---
<br>
<br>

## Common Methods

The objects listed above all share some common methods, which are implemented in the parent *Objective* base class.

<br>
<br>

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

Stations include some unique features regarding mover registration. See [Station Object](Objects/Station.md) for more details.

---
<br>
<br>

### RegisterMoverList

*RegisterMoverList( NewMoverList : MoverList )*

> Provides the same functionality as [RegisterMover](#registermover), but for a group of movers represented as a [MoverList](Objects/MoverList.md).

```javascript
MoverListA.RegisterMoverList(MoverListB);
```
---
<br>
<br>

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

---
<br>
<br>

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

---6
<br>
<br>

### UnregisterAll

*UnregisterAll()*

> Automatically unregisters every mover from the Tracked Movers list

```javascript
IF xClearTrigger THEN
	PositionTrigger[2].UnregisterAll();
	xClearTrigger	:= FALSE;
END_IF
```

---
<br>
<br>

## Method Chaining

Most methods in the project return *the current object itself*, allowing for additional method calls on the same line of code. This approach is often referred to as a *fluent interface*. For example:

```javascript
// All these method calls apply to Mover 0 in sequence
Mover[0].SetVelocity( 2000 ).SetAcceleration( 20000).MoveToPosition( 1000 );
```

In some cases, the *order* of the methods calls must be carefully considered. In the following example, the *CurrentMover* of *Station#0* is commanded to move to *Station#1*. At this point, it is no longer considered the *CurrentMover* of *Station#0*. As a result, the *SetVelocity* method does not apply to the intended Mover object.

```javascript
IF Station[0].MoverInPosition THEN
	// This line will return an ErrorMover and will not function as intended
	Station[0].CurrentMover.MoveToStation(Station[1]).SetVelocity(1000);
END_IF;
```

However, the following example will work as intended:

```javascript
IF Station[0].MoverInPosition THEN
	// This line of code will function correctly
	Station[0].CurrentMover.SetVelocity(1000).MoveToStation(Station[1]);
END_IF;
```

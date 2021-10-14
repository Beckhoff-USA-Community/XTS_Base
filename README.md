
# XTS Base Code

The goal of this repository is to provide a minimal Starter Project for [Beckhoff XTS](https://www.beckhoff.com/en-en/products/motion/xts-linear-product-transport/?pk_campaign=AdWords-AdWordsSearch-XTS_EN&pk_kwd=xts&gclid=Cj0KCQjwqp-LBhDQARIsAO0a6aJsNGgetwo4ur4_Xtr1ndGf5e2lnI9EKZHnQzBE3XHpt7mB6rVKqOIaAnGLEALw_wcB) applications.

## Project Organization

1. Repo structure should largely follow Vincent Driessen's [Gitflow](https://nvie.com/posts/a-successful-git-branching-model/)
2. Bugs & conventions are managed in the Issues tab within Github

---
### Mover

The Mover function block is the heart of the solution. It contains essential functionality, including:
- Basic administrative commands like enabling & resetting
- Easily commanding movements to absolute & relative positions
- Reading out current move status information like position, velocity, etc.
- Updating motion parameters during operation

Below are simple examples of different operations utilizing the Mover object:

```javascript
// Mover : ARRAY [0..gNUM_MOVERS] OF Mover

Mover[1].SetVelocity( 2500 );
Mover[1].MoveToPosition( 1200 );
```


```javascript
// ParameterSet : MoverParameters_typ;

ParameterSet.Velocity		:= 2200;	// mm/s
ParameterSet.Acceleration	:= 15000;	// mm/s2
ParameterSet.Deceleratoin	:= 10000;	// mm/s2
ParameterSet.Jerk			:= mcDEFAULT;
ParameterSet.Direction		:= mcDirectionPositive;

Mover[1].MotionParameters	:= ParameterSet;
```


```javascript
// adjacentPosition : REAL;

adjacentPosition			:= Mover[2].AxisReference.NcToPlc.ActPos - 200;

Mover[1].MoveToPosition( adjacentPosition );
```

---

### Station

A Station is a defined track position for the Mover to stop and wait until it is released. Stations are objects that actively monitor all of the *Movers* that are **currently destined for that particular Station**. When any of those monitored Movers are stopped at the Station, the Station reports that it has a *Mover In Position* and provides a Pointer to that mover.

This approach makes it easy to send new commands to whatever *Mover* is currently docked with a Station.

```javascript
// Station : ARRAY [0..gNUM_STATIONS] OF Station

IF Station[1].MoverInPosition AND xCommandRelease THEN
	Station[1].CurrentMover^.SetVelocity( myVelocityParameter );
	Station[1].CurrentMover^.ReleaseToStation( Station[2] );
END_IF
```

> Note: *Station[x].CurrentMover* is a pointer, is only valid when a Mover is present at the Station. Be careful dereferencing.

Stations need to know which Movers are *incoming* in order to actively monitor them. Stations include methods to *register* new incoming movers to be added to a tracked list.

If a Mover that was never *registered* with a Station arrives and stops at the Station, it will not be reported as *In Position* and no Pointer will be provided.

Registration and deregistration can be done manually if needed:
```javascript
// Add Mover #2 to the Tracked Movers list
Station[1].RegisterMover( Mover[2] );

// Remove it again
Station[1].UnregisterMover( Mover[2] );
```
Or, in the case of the Mover's MoveToStation method, is handled automatically:
```javascript
Mover[2].MoveToStation( Station[1] );
// Station[1].RegisterMover(Mover[2]) called implicitly
```
---
### Abstracting the Concept

In a sense, a Station represents a *goal* or *objective* for the Mover to achieve: The mover receives a command to try an accomplish the goal, and the Station object monitors until the Mover is *successful* (i.e. In Positionn).

In the objects defined below, we extend this idea to include other types of objectives, like reaching or exceeding a specified Velocity.

---

### Speed Trigger

Like Station objects, Movers must be *registered* with a Speed Trigger for the trigger object to actively monitor it. Multiple movers can be registered with the same Speed Trigger.

```javascript
// Add Mover #3 to a list of Movers tracked by the trigger object
SpeedTrigger.RegisterMover( Mover[3] );
```

Now, the Speed Trigger will report when any of the watched Movers cross over a set Threshold Velocity so new commands can be issued:

```javascript
SpeedTrigger.ThresholdVelocity	:= 1500;

IF SpeedTrigger.MoverExceedsVelocity THEN
	SpeedTrigger.CurrentMover^.Stop()
	SpeedTrigger.UnregisterCurrent();
END_IF
```

---
### Position Trigger

Position Triggers become active if a Mover has crossed over the Threshold position *since being registered*. They are operated in a nearly identical fashion to Speed Triggers:

```javascript
// Register the Mover
PositionTrigger.ThresholdPosition	:= 450;
PositionTrigger.RegisterMover( Mover[3] );

// Update mover velocity at position
IF PositionTrigger.MoverPastPosition THEN
	PositionTrigger.CurrentMover^.SetVelocity( UpdatedVelocity );
	PositionTrigger.UnregisterCurrent();
END_IF
```
---

### Force Trigger

> Under construction

---

### Zone Triggers

> Under construction

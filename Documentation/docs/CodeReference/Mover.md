
# Mover Object

> The Mover function block is the heart of the solution. It contains essential functionality, including:

> - Basic administrative commands like enabling & resetting
- Easily commanding movements to absolute positions or stations
- Reading out current move status information like position, velocity, etc.
- Updating motion parameters during operation

---

## Setup & Execution

It is recommended, but not required, to declare Movers as an array.
```javascript
// Instance declaration
Mover			: ARRAY [0..GVL.NUM_MOVERS-1] OF Mover;

ParameterSet	: MoverParameters_typ;
```

Movers contain a Cyclic() method that must be called every cycle. This cyclic method must also be given a Collision Avoidance Group reference as an argument.

```javascript
// Call this method cyclically
FOR i := 0 TO GVL.NUM_MOVERS-1] DO
	Mover[i].Cyclic( GroupRef );
END_FOR
```

---

## Methods


### Enable

*Enable()*

> Carries out steps to prepare the Mover for motion commands.

1. An internal MC_Reset function block is called to reset any errors that may exist on the Axis object.
2. The mover will read M1 Detection Settings configured in the XtsProcessingUnit. If M1 Detection is configured, the Mover will activate this process to find Mover 1. If this is not configured, this step is skipped.
3. An internal MC_Power function block is called to energize the axis object.
4. The mover will automatically add itself to a Collision Avoidance group, and if necessary will also automatically enable this group.
5. The mover will set its Ready output True.

```javascript
Mover[1].Enable();
```

---

### Disable

*Disable()*

> Disables the mover. Holding Torque will be lost. Method will automatically remove the mover from its Collision Avoidance group.

```javascript
Mover[1].Disable();
```

---

### MoveToPosition

*MoveToPosition( Destination : LREAL )*

> Executes an Absolute Move (with Collision Avoidance) to the target location.

```javascript
IF xMoveCommand THEN
	Mover[1].MoveToPosition( 1200 );
	xMoveCommand	:= FALSE;
END_IF
```

---

### MoveToStation

*MoveToStation( Destination : Station )*

> Executes an Absolute Move (with Collision Avoidance) to the location of the target Station, and implicitly calls that Station's [RegisterMover](../Objectives/Station#registermover) method.

```javascript
IF xMoveToHome THEN
	Mover[1].MoveToStation( Station[1] );
	xMoveToHome		:= FALSE;
END_IF
```

---

### MoveVelocity

*MoveVelocity( DesiredVelocity : LREAL )*

> Executes a Velocity movement (with Collision Avoidance) at the specified Velocity. This command also implicitly updates the Mover's internal Velocity Motion Parameter.

```javascript
IF xMoveVelocity THEN
	Mover[1].MoveVelocity( 2000 );
	xMoveVelocity	:= FALSE;
END_IF
```

---

### ReissueCommand

*ReissueCommand()*

> Executes the latest command that was issued to the mover, e.g. a MoveToPosition command. The move is executed with the *current* internal Motion Parameters of the Mover, so the dynamics of the mover may not exactly match what was initially commanded. However, in cases of MoveToPosition or MoveToStation, the destination will be the same.

To determine what type of movement ReissueCommand() will repeat, see Mover properties CurrentMoveType, CurrentObjective, and CurrentDestinationPosition.

```javascript
// Issue a primary command
IF xInitialCommand THEN
	Mover[1].SetVelocity( 500 );
	Mover[1].MoveToPosition( 1000 );
	xInitialCommand		:= FALSE;

// Reissue the primary command, but with a new velocity parameter
ELSIF xUpdateCommand THEN
	Mover[1].SetVelocity( 750 );
	Mover[1].ReissueCommand();
	xUpdateCommand		:= FALSE;
END_IF
```

---

### SetAcceleration

*SetAcceleration( DesiredAccel : LREAL )*

> Updates the Mover's internal Motion Parameter for Acceleration [mm/s2] and immediately reissues any currently executing motion blocks so that the parameter update takes effect immediately

```javascript
IF xCommandHighAccel THEN
	Mover[1].SetAcceleration( 2e4 );	// mm/s2
	xCommandHighAccel		:= FALSE;
END_IF
```

---

### SetDeceleration

*SetDeceleration( DesiredDecel : LREAL )*

> Updates the Mover's internal Motion Parameter for Deceleration [mm/s2] and immediately reissues any currently executing motion blocks so that the parameter update takes effect immediately

```javascript
IF xCommandLowDecel THEN
	Mover[1].SetDeceleration( 1e3 );	// mm/s2
	xCommandLowDecel	:= FALSE;
END_IF
```

---

### SetDirection

*SetDirection( Direction : Tc3_Mc3Definitions.MC_Direction );*

> Updates the Mover's internal Motion Parameter for Direction and immediately reissues any currently executing motion blocks so that the parameter update takes effect immediately

This method should only be used with care, and an understanding that a mover enroute will immediately reverse course and execute a motion command in the opposite direction when this method is called.

```javascript
// For supported directions, see Infosys

IF xReverseDirection THEN
	Mover[1].SetDirection( mcDirectionPositive );
	xReverseDirection	:= FALSE;
END_IF
```

---

### SetJerk

*SetJerk( Jerk : LREAL )*

> Updates the Mover's internal Motion Parameter for Jerk [mm/s3] and immediately reissues any currently executing motion blocks so that the parameter update takes effect immediately

```javascript
IF xUpdateJerk THEN
	Mover[1].UpdateJerk( 1e5 );
	xUpdateJerk		:= FALSE;
END_IF
```

---

### SetVelocity

*SetVelocity( DesiredVelocity : LREAL )*

> Updates the Mover's internal Motion Parameter for Velocity [mm/s] and immediately reissues any currently executing motion blocks so that the parameter update takes effect immediately

```javascript
IF xCommandSlowMode THEN
	Mover[1].SetVelocity( 500 );	// mm/s
	xCommandSlowMode 	:= FALSE;
END_IF
```

## Properties

#### .CurrentMoveType

*MoverCommandType_enum*

```
MOVETYPE_NONE			// this mover was just enabled and has no movement history
MOVETYPE_POSITION		// this mover was most recently issued a MoveToPosition command
MOVETYPE_STATION		// this mover was most recently issued a MoveToStation command
MOVETYPE_VELOCITY		// this mover was most recently issued a MoveVelocity command
```

> Provides the current (last executed) type of movement command issued to the Mover

---

#### .CurrentDestinationPosition

*LREAL*

> Provides the current destination position for the last movement command issued to the Mover. For Station commands, this will be the TrackPosition of the Station. For Velocity commands with no real destination position, the value is set to +/-1E300.

---

#### .MotionParameters

*MotionParameters_typ*

> Defines a structure containing the dynamics settings for the Mover. Any new motion commands issued will utilize these values.

*Note: despite listing this value as a Property here in the documentation, MotionParameters are actually defined as a regular Input to the Mover object. This allows component access to the members of the STRUCT, which is not possible for Properties.*

---

#### .CurrentObjective

*STRING*

> Provides the current Objective destination for the Mover. Right now this is only valid when the Mover is destined for a Station objective, and provides a string name for that station.

---

#### .Payload

> !!! Under Construction !!! At the moment, this property is a placeholder for application specific information regarding the current status of products onboard the mover, and can be modified as needed for your application.

---

## Extra Examples

Below are simple examples of different operations utilizing the Mover object:

```javascript
// Mover: ARRAY [0..GVL.NUM_MOVERS] OF Mover
Mover[1].SetVelocity( 2500 );
Mover[1].MoveToPosition( 1200 );
```

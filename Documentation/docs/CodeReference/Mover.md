
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

Movers contain a Cyclic() method that must be called every cycle. This cyclic method must also be given a Collision Avoidance Group reference as an argument. If track management is used, the CyclicTrack() method must also be called each scan.

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
<br>
<br>

### Disable

*Disable()*

> Disables the mover. Holding Torque will be lost. Method will automatically remove the mover from its Collision Avoidance group.

```javascript
Mover[1].Disable();
```

---

### GroupStop

*GroupStop()*

> Immediately stops all enabled movers on the system with the deceleration parameters of the mover on which the method was called.

Since the method acts upon the collision avoidance group, all enabled movers on the system are immediately stopped and any active movements are cancelled. New motion commands to each mover are ignored until the Stop has completed.

```javascript
IF xStopAllMovers THEN
	Mover[0].GroupStop();
	xStopAllMovers		:= FALSE;
END_IF
```

---
<br>
<br>

### Halt

*Halt()*

> Immediately stops the commanded mover with the deceleration parameters stored in the Mover object

Halt does not alter the mover's internal axis state, so new movements can be executed at any point to send the mover onwards.

```javascript
IF xStopSingleMover THEN
	Mover[0].Halt();
	xStopSingleMover	:= FALSE;
END_IF
```

---
<br>
<br>

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
<br>
<br>

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
<br>
<br>

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
<br>
<br>

### SyncToAxis

*SyncToAxis( MasterAxis : AXIS_REFERENCE, MasterSyncPos : LREAL, SlaveSyncPos : LREAL, SyncStrategy : MC_SYNC_STRATEGY )*

> Pairs the current mover with an external axis (real or virtual) with specified Master & Slave Sync Positions. The current mover will synchronize according to the parameter SyncStrategy and a 1:1 gear ratio.
The synchronization can be ended by executing a call on the slave for any other motion command, e.g. MoveToPosition.

**MasterAxis** is a reference to the NC axis object that the mover should synchronize with, real or virtual.

**MasterSyncPos** is the position of the Master at which point the mover will be *InSync* and will have the correct gear ratio (1:1).

**SlaveSyncPos** is the position of the mover at which point it is *InSync*

**SyncStrategy** defines the type of calculation used to blend the movements and establish the synchronization. Simply put, the mover should synchronize *immediately & aggressively*, *as late as possible and aggressively*, or *as gently as possible*.

> For more information on these inputs, see InfoSys documentation for the underlying [MC_GearInPosCA](https://infosys.beckhoff.com/english.php?content=../content/1033/tf5410_tc3_collision_avoidance/1534969995.html&id=) Function Block


> See also:
> [Mover.IsSyncedToAxis](Mover.md#issyncedtoaxis)

```javascript
IF xCmdSyncToAxis THEN
	xCmdSyncToAxis	:= FALSE;
	MasterSyncPos	:= ConveyorAxis.NcToPlc.ActPos + 400;
	Mover[0].SyncToAxis( ConveyorAxis, MasterSyncPos, 1500, mcSyncStrategyEarly );
END_IF;
```

---
<br>
<br>

### SyncToMover

*SyncToMover( MasterMover : Mover, Gap : LREAL )*

> Pairs the current mover with a desired mover at a specified gap distance. The current mover will immediately move (with CA) to a specified distance from the Master Mover and then mimic all motion from the master. The synchronization can be ended by executing a call on the slave for any other motion command, e.g. MoveToPosition.


**Gap** specifies a center-to-center following distance between master and slave movers. This gap cannot be achieved if it is below the minimum collision avoidance distance established in the CA group. Positive Gap values will result in a following position *behind* the master and negative values will result in a slave that *precedes* the master.


Additional calls to this method can be used to update the gap between paired movers.

> See also:
> [Mover.IsSyncedToMover](Mover.md#issyncedtomover) and [Mover.MasterMover](Mover.md#mastermover)


```javascript

IF xCmdSyncToLeader THEN
	Mover[0].SyncToMover( Mover[1], 100 );	// move 100mm away from Mover 1

	IF Mover[0].IsSyncedToMover THEN
		Mover[1].MoveToPosition( 2000 );	// Master moves to 2000 and Mover 0 will follow
	END_IF;
END_IF;
```

```javascript
IF xBuildTrain THEN
	Mover[3].SyncToMover( Mover[4], 100 );
	Mover[2].SyncToMover( Mover[4], 200 );
	Mover[1].SyncToMover( Mover[4], 300 );
	Mover[0].SyncToMover( Mover[4], 400 );

	IF Mover[0].IsSyncedToMover THEN
		// Commands Mover 4 to move ahead at 300 mm/s
		Mover[0].MasterMover.MoveVelocity( 300 );
	END_IF;
END_IF;		
```

---
<br>
<br>

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
<br>
<br>

### LogUserEvent

*LogUserEvent( Text1 : STRING, Text2 : STRING, Text3 : STRING )*

> Submits custom log events to the Event Logger which can be viewed in TwinCAT

```javascript
IF PositionTrigger[1].MoverPassedPosition THEN
	PositionTrigger[1].CurrentMover.LogUserEvent('Mover Passed PT#1','','');
END_IF;
```

---
<br>
<br>

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
<br>
<br>

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
<br>
<br>

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
<br>
<br>

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
<br>
<br>

### SetVelocity

*SetVelocity( DesiredVelocity : LREAL )*

> Updates the Mover's internal Motion Parameter for Velocity [mm/s] and immediately reissues any currently executing motion blocks so that the parameter update takes effect immediately

```javascript
IF xCommandSlowMode THEN
	Mover[1].SetVelocity( 500 );	// mm/s
	xCommandSlowMode 	:= FALSE;
END_IF
```

### ActivateTrack

*ActivateTrack( DesiredTrack : Track )*

> Updates the mover's logical track.

This method is used with track management to change the track that the mover is assigned to. Two conditions should be considered when changing tracks.
- A mover will stop immediately when ActivateTrack is called. It's recommended to only change tracks when the mover is in a stopped position.
- A mover's position can change when calling ActivateTrack. This will happen if the zero point for the current track and new track are different.

A track change takes several PLC and NC scans. Issuing a motion command while this change is in progress will cause the mover to throw an error. To check if the track change is complete monitor the Mover.IsTrackReady property.

```javascript
// state machine
100:
	Mover[0].ActivateTrack(Track1);
	IF (Mover[0].IsTrackReady) THEN
		iState := 200;
	END_IF;
200:
	// mover is ready for additional commands
```

---
<br>
<br>

### SetGap

*SetGap( Gap : LREAL )*

> Sets the collision avoidance gap for the specified mover in mm. This takes effect immediately.

Caution: This method can cause unexpected motion. If setting a larger gap for a mover that currently has movers within the new gap distance (on either side of the mover). Each affected mover will immediately attempt to adjust for the new gap which may result in movers moving forwards or backwards. When increasing the gap between movers it's recommended to do this one mover at a time only when the space in front of the mover is clear for at least the new gap distance. This can be accomplished with either a PositionTrigger or Zone.

```javascript
Mover[1].SetGap( 65.0 );
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
<br>
<br>

#### .CurrentDestinationPosition

*LREAL*

> Provides the current destination position for the last movement command issued to the Mover. For Station commands, this will be the Position of the Station. For Velocity commands with no real destination position, the value is set to +/-1E300.

---
<br>
<br>

#### .MotionParameters

*MotionParameters_typ*

> Defines a structure containing the dynamics settings for the Mover. Any new motion commands issued will utilize these values.

*Note: despite listing this value as a Property here in the documentation, MotionParameters are actually defined as a regular Input to the Mover object. This allows component access to the members of the STRUCT, which is not possible for Properties.*

---
<br>
<br>

#### .CurrentObjective

*STRING*

> Provides the current Objective destination for the Mover. Right now this is only valid when the Mover is destined for a Station objective, and provides a string name for that station.

---
<br>
<br>

#### .CurrentTrack

*POINTER TO Track*

> Returns a pointer to the track that this mover is currently assigned to.

Use the ^ operator to dereference the pointer and query track properties

```javascript
currentTrackId := Mover[1].CurrentTrack^.Id
```

---
<br>
<br>

#### .IsSyncedToMover

*BOOL*

> Returns true if the mover is slaved to another mover and has successfully reached the following position specified by the Gap.

---
<br>
<br>

#### .IsSyncedToAxis

*BOOL*

> Returns true if the mover is slaved to an external axis and has successfully reached the following position specified by the Master & Slave Sync Positions

---
<br>
<br>

#### .IsTrackReady

*BOOL*

> Returns true when the ActivateTrack() method has completed successfully. When using track management motion commands should not be issued unless this value is true or the mover will throw an error.

---
<br>
<br>

#### .MasterMover

*REFERENCE To Mover*

> Reference variable that refers to the Mover this Mover is currently slaved to

When this mover is not slaved to another mover, .MasterMover is an invalid reference. There is therefore an ErrorMover object which will return information on what action was requested by the object while .MasterMover was invalid. This will show up in TwinCAT's Event Logs. See [Diagnostics / ErrorMovers](../CodeReference/ErrorMovers.md) for more information.  

It is recommended that all evaluations are nested inside IF checks for .IsSyncedToMover OR by calling __ISVALIDREF

---
<br>
<br>

#### .Payload

> !!! Under Construction !!! At the moment, this property is a placeholder for application specific information regarding the current status of products onboard the mover, and can be modified as needed for your application.

---
<br>
<br>

#### .TrackInfo

> Returns information provided by MC_ReadTrackPositions for use with track management.

| Member | Type | Description |
|---|---|---|
| TrackId | OTCID | The hardware ID of the track the mover is assigned to |
| PartId | OTCID | The hardware ID of the part the mover is physically present on |
| TrackPosition | LREAL | Position of the mover measured from the zero point of the track. This may differ from Mover.AxisReference.NcToPLC.ActPos in certain track configurations |
| PartPosition | LREAL | Position of the mover measured from the zero point of the part the mover is physically on|

---
<br>
<br>

## Extra Examples

Below are simple examples of different operations utilizing the Mover object:

```javascript
// Mover: ARRAY [0..GVL.NUM_MOVERS] OF Mover
Mover[1].SetVelocity( 2500 );
Mover[1].MoveToPosition( 1200 );
```

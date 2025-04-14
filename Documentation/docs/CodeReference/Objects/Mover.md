
# Mover Object

> The Mover function block is the heart of the solution. It contains essential functionality, including:

> - Basic administrative commands like enabling & disabling
- Easily commanding movements to absolute positions or stations
- Reading out current move status information like position, velocity, etc.
- Updating motion parameters during operation

Many of the motion related commands are chainable as noted below. This allows a concise set of motion commands to be written on one line.

```javascript
// chained motion command that sets velocity, acceleration and a destination in one line
Mover[1].SetVelocity(1000).SetAcceleration(1e3).MoveToStation(Station[2]);
```

## Setup & Execution

It is recommended, but not required, to declare Movers as an array.
```javascript
// Instance declaration
Mover			: ARRAY [1..GVL.NUM_MOVERS] OF Mover;
```

Movers must also be added to the Mediator object. By default, this is handled automatically.

```javascript
// Example implementation
Mediator.AddMover( Mover[1] );
```


## Methods

### ActivateTrack

*ActivateTrack( DesiredTrack : Track )*

> Updates the mover's logical track.

This method is used with track management to change the track that the mover is assigned to. Two conditions should be considered when changing tracks:

- A mover will stop immediately when ActivateTrack is called. It's recommended to only change tracks when the mover is in standstill
- A mover's position can change when calling ActivateTrack. This will happen if the zero point for the current track and new track are different.

A track change takes several PLC and NC scans. Issuing a motion command while this change is in progress will cause the mover to throw an error. To check if the track change is complete monitor the `Mover.IsTrackReady` property.

```javascript
// state machine
100:
	Mover[1].ActivateTrack(Track1);
	IF (Mover[1].IsTrackReady) THEN
		iState := 200;
	END_IF;
200:
	// mover is ready for additional commands
```

### Cyclic
*Cyclic()*

> Handles background tasks for the mover such as populating status bits and handling completion of tasks that take more than 1 scan.

!!! important
	Cyclic must be called every cycle. By default this is handled automatically by the [Mediator](./Mediator.md).

### Disable

*Disable()*

*Chainable*

> Disables the mover. Holding Torque will be lost. Method will automatically remove the mover from its Collision Avoidance group.

```javascript
Mover[1].Disable();
```

### Enable

*Enable()*

*Chainable*

> Carries out steps to prepare the Mover for motion commands.

1. An internal MC_Reset function block is called to reset any errors that may exist on the Axis object.
2. The mover, via the [Mediator](./Mediator.md), will read M1 Detection Settings configured in the XtsProcessingUnit. If M1 Detection is configured, it will activate this process to find Mover 1. If this is not configured, this step is skipped.
3. An internal MC_Power function block is called to energize the axis object.
4. The mover will automatically add itself to a Collision Avoidance group, and if necessary will also automatically enable this group.
5. The mover will set its `Ready` output True.

```javascript
Mover[1].Enable();
```
### GroupStop

*GroupStop()*

*Chainable*

> Immediately stops all enabled movers on the system with the deceleration parameters of the mover on which the method was called.

Since the method acts upon the collision avoidance group, all enabled movers on the system are immediately stopped and any active movements are cancelled. New motion commands to each mover are ignored until the Stop has completed.

```javascript
IF xStopAllMovers THEN
	Mover[1].GroupStop();
	xStopAllMovers		:= FALSE;
END_IF
```
### Halt

*Halt()*

*Chainable*

> Immediately stops the commanded mover with the deceleration parameters stored in the Mover object

Halt does not alter the mover's internal axis state, so new movements can be executed at any point to send the mover onwards.

```javascript
IF xStopSingleMover THEN
	Mover[1].Halt();
	xStopSingleMover	:= FALSE;
END_IF
```
### LogUserEvent

*LogUserEvent( Text1 : STRING, Text2 : STRING, Text3 : STRING )*

*Chainable*

> Submits custom log events to the Event Logger which can be viewed in TwinCAT

```javascript
IF PositionTrigger[1].MoverPassedPosition THEN
	PositionTrigger[1].CurrentMover.LogUserEvent('Mover Passed PT#1','','');
END_IF;
```

### MoveToPosition

*MoveToPosition( Destination : LREAL )*

*Chainable*

> Executes an Absolute Move (with Collision Avoidance) to the target location.

```javascript
IF xMoveCommand THEN
	Mover[1].MoveToPosition( 1200 );
	xMoveCommand	:= FALSE;
END_IF
```


### MoveToStation

*MoveToStation( Destination : Station )*

*Chainable*

> Executes an Absolute Move (with Collision Avoidance) to the location of the target Station, and implicitly calls that Station's [RegisterMover](../Objectives/Station#registermover) method.

```javascript
IF xMoveToHome THEN
	Mover[1].MoveToStation( Station[1] );
	xMoveToHome		:= FALSE;
END_IF
```
### MoveVelocity

*MoveVelocity( DesiredVelocity : LREAL )*

*Chainable*

> Executes a Velocity movement (with Collision Avoidance) at the specified Velocity. This command also implicitly updates the Mover's internal Velocity Motion Parameter.

```javascript
IF xMoveVelocity THEN
	Mover[1].MoveVelocity( 2000 );
	xMoveVelocity	:= FALSE;
END_IF
```
### ReissueCommand

*ReissueCommand()*

*Chainable*

> Executes the latest command that was issued to the mover, e.g. a MoveToPosition command. The move is executed with the *current* internal Motion Parameters of the Mover, so the dynamics of the mover may not exactly match what was initially commanded. However, in cases of MoveToPosition or MoveToStation, the destination will be the same.

To determine what type of movement ReissueCommand() will repeat, see Mover properties `CurrentMoveType`, `CurrentObjective`, and `CurrentDestinationPosition`.

```javascript
// Issue a primary command
IF xInitialCommand THEN
	Mover[1].MotionParameters.Velocity	:= 500;	// mm/s
	Mover[1].MoveToPosition( 1000 );
	xInitialCommand		:= FALSE;

// Reissue the primary command, but with a new velocity parameter
ELSIF xUpdateCommand THEN
	Mover[1].MotionParemeters.Velocity	:= 750; // mm/s
	Mover[1].ReissueCommand();
	xUpdateCommand		:= FALSE;
END_IF
```

### Reset

*Reset()*

*Chainable*

> Calls a reset on the underlying axis immediately. The state machine unaffected and will, for example, keep the mover in the enabled state if .Reset() is called while is in the enable state.

The mover's error properties will be cleared, and if and Axis error continues to exist after the reset, the error properties will continue to reflect this.


### SetAcceleration

*SetAcceleration( DesiredAccel : LREAL )*

*Chainable*

> Updates the Mover's internal Motion Parameter for Acceleration (mm/s²) and immediately reissues any currently executing motion blocks so that the parameter update takes effect immediately

```javascript
IF xCommandHighAccel THEN
	Mover[1].SetAcceleration( 2e4 );	// mm/s²
	xCommandHighAccel		:= FALSE;
END_IF
```

!!! note "Notice"
	This method implicitly calls .ReissueCommand() in the background in order for the updated dynamics to take effect immediately. If this is not the intent, consider modifying the *Mover[x].MotionParameters* property instead.


### SetDeceleration

*SetDeceleration( DesiredDecel : LREAL )*

*Chainable*

> Updates the Mover's internal Motion Parameter for Deceleration (mm/s²) and immediately reissues any currently executing motion blocks so that the parameter update takes effect immediately

```javascript
IF xCommandLowDecel THEN
	Mover[1].SetDeceleration( 1e3 );	// mm/s²
	xCommandLowDecel	:= FALSE;
END_IF
```
!!! note "Notice"
	This method implicitly calls .ReissueCommand() in the background in order for the updated dynamics to take effect immediately. If this is not the intent, consider modifying the *Mover[x].MotionParameters* property instead.

### SetDirection

*SetDirection( Direction : [Tc3_Mc3Definitions.MC_Direction](https://infosys.beckhoff.com/english.php?content=../content/1033/tf5410_tc3_collision_avoidance/8996812427.html&id=) );*

*Chainable*

> Updates the Mover's internal Motion Parameter for Direction and immediately reissues any currently executing motion blocks so that the parameter update takes effect immediately

!!! warning "Caution"
	This method implicitly calls .ReissueCommand() in the background in order for the updated dynamics to take effect immediately. If this is not the intent, consider modifying the *Mover[x].MotionParameters* property instead. This method should only be used with care, and an understanding that a mover enroute can immediately reverse course and execute a motion command in the opposite direction when this method is called.

```javascript
// For supported directions, see Infosys

IF xReverseDirection THEN
	Mover[1].SetDirection( mcDirectionPositive );
	xReverseDirection	:= FALSE;
END_IF
```
### SetGap

*SetGap( Gap : LREAL )*

*Chainable*

> Sets the collision avoidance gap for the specified mover in mm. This takes effect immediately.

!!! warning "Caution"
	This method can cause unexpected motion. If setting a larger gap for a mover that currently has movers within the new gap distance (on either side of the mover). Each affected mover will immediately attempt to adjust for the new gap which may result in movers moving forwards or backwards.

When increasing the gap between movers it's recommended to do this one mover at a time only when the space in front of the mover is clear for at least the new gap distance. This can be accomplished with either a PositionTrigger or Zone.

```javascript
Mover[1].SetGap( 65.0 );
```


### SetGapMode

*SetGapMode( Mode : [Tc3_Mc3Definitions.MC_GAP_CONTROL_MODE](https://infosys.beckhoff.com/english.php?content=../content/1033/tf5410_tc3_collision_avoidance/2783316363.html&id=) )*

*Chainable*

> Sets the collision avoidance gap control mode for the specified mover. This takes effect immediately.

Default defers to the mode set in the NC's Collision Avoidance object.

!!! Note
	For modes other than Standard the NC's Collision Avoidance object must be set to `mcGapCtrlDirectionPositive`. This paramater can only be set in configuration and not during runtime.

| Mode | Description |
|--|--|
| mcGapControlModeGroupDefault | This value indicates that the GapControlMode set in the group parameters should be used for this motion command. |
| mcGapControlModeStandard | "Caterpillar" The axes fan out characteristically during the acceleration phase, such that a collision during the motion command is prevented. |
| mcGapControlModeFast | "Lock-step" All Axes move at the same time and with the full dynamics. |
| mcGapControlModeNone | This value indicates that the Gap Control is not active in the command. After the command, the Standby Gap Control takes effect again with the mode, which is set in the group and the gap size of the last valid command. |

Additional documentation of gap control mode is available on [Infosys](https://infosys.beckhoff.com/content/1033/tf5410_tc3_collision_avoidance/1539510027.html?id=7324917585245879036) which includes graphs comparing the options.

```javascript
Mover[1].SetGapMode( mcGapControlModeStandard );
```

### SetJerk

*SetJerk( Jerk : LREAL )*

*Chainable*

> Updates the Mover's internal Motion Parameter for Jerk [mm/s3] and immediately reissues any currently executing motion blocks so that the parameter update takes effect immediately

```javascript
IF xUpdateJerk THEN
	Mover[1].SetJerk( 1e5 );
	xUpdateJerk		:= FALSE;
END_IF
```
!!! note "Notice"
	This method implicitly calls .ReissueCommand() in the background in order for the updated dynamics to take effect immediately. If this is not the intent, consider modifying the *Mover[x].MotionParameters* property instead.


### SetVelocity

*SetVelocity( DesiredVelocity : LREAL )*

*Chainable*

> Updates the Mover's internal Motion Parameter for Velocity [mm/s] and immediately reissues any currently executing motion blocks so that the parameter update takes effect immediately

```javascript
IF xCommandSlowMode THEN
	Mover[1].SetVelocity( 500 );	// mm/s
	xCommandSlowMode 	:= FALSE;
END_IF
```
!!! note "Notice"
	This method implicitly calls .ReissueCommand() in the background in order for the updated dynamics to take effect immediately. If this is not the intent, consider modifying the *Mover[x].MotionParameters* property instead.


### SyncToAxis

*SyncToAxis( MasterAxis : AXIS_REFERENCE, MasterSyncPos : LREAL, SlaveSyncPos : LREAL, SyncStrategy : MC_SYNC_STRATEGY )*

*Chainable*

> Pairs the current mover with an external axis (real or virtual) with specified Master & Slave Sync Positions. The current mover will synchronize according to the parameter SyncStrategy and a 1:1 gear ratio.
The synchronization can be ended by executing a call on the slave for any other motion command, e.g. `MoveToPosition` or `Halt`.

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
	Mover[1].SyncToAxis( ConveyorAxis, MasterSyncPos, 1500, mcSyncStrategyEarly );
END_IF;
```
### SyncToMover

*SyncToMover( MasterMover : Mover, Gap : LREAL )*

*Chainable*

> Pairs the current mover with a desired mover at a specified gap distance. The current mover will immediately move (with CA) to a specified distance from the Master Mover and then mimic all motion from the master. The synchronization can be ended by executing a call on the slave for any other motion command, e.g. `MoveToPosition` or `Halt`.


**Gap** specifies a center-to-center following distance between master and slave movers. This gap cannot be achieved if it is below the minimum collision avoidance distance established in the CA group. Positive Gap values will result in a following position *behind* the master and negative values will result in a slave that *precedes* the master.


Additional calls to this method can be used to update the gap between paired movers.

> See also:
> [Mover.IsSyncedToMover](Mover.md#issyncedtomover) and [Mover.MasterMover](Mover.md#mastermover)


```javascript

IF xCmdSyncToLeader THEN
	Mover[1].SyncToMover( Mover[1], 100 );	// move 100mm away from Mover 1

	IF Mover[1].IsSyncedToMover THEN
		Mover[2].MoveToPosition( 2000 );	// Master moves to 2000 and Mover 0 will follow
	END_IF;
END_IF;
```

```javascript
IF xBuildTrain THEN
	Mover[4].SyncToMover( Mover[4], 100 );
	Mover[3].SyncToMover( Mover[4], 200 );
	Mover[2].SyncToMover( Mover[4], 300 );
	Mover[1].SyncToMover( Mover[4], 400 );

	IF Mover[1].IsSyncedToMover THEN
		// Commands Mover 4 to move ahead at 300 mm/s
		Mover[1].MasterMover.MoveVelocity( 300 );
	END_IF;
END_IF;		
```

### UnregisterFromAll
*UnregisterFromAll()*

> Unregisters a mover from all Mover Lists, Position Triggers, Station and Zones.



### ValidateTrack

*ValidateTrack()*

> Confirms that a valid track is selected and returns true. Otherwise logs an error and returns false to be handled by higher-level code.

This method is typically used internally by other motion methods of movers to log an error message when a command is issued to a mover who is not on a valid track. The property .IsTrackReady is intended for use in user code.


## Properties

### .CurrentDestinationPosition

*LREAL*

> Provides the current destination position for the last movement command issued to the Mover. For Station commands, this will be the Position of the Station. For Velocity commands with no real destination position, the value is set to +/-1E300.


### .CurrentDestinationTrack

*DINT*

> Provides the current destination track for the last movement command issued to the Mover. When track management is used, it should be queried along with the property .CurrentDestinationPosition to validate destinations.

### .CurrentMoveType

*MoverCommandType_enum*

```
MOVETYPE_NONE			// this mover was just enabled and has no movement history
MOVETYPE_POSITION		// this mover was most recently issued a MoveToPosition command
MOVETYPE_STATION		// this mover was most recently issued a MoveToStation command
MOVETYPE_VELOCITY		// this mover was most recently issued a MoveVelocity command
```

> Provides the current (last executed) type of movement command issued to the Mover

### .CurrentObjective

*STRING*

> Provides the current Objective destination for the Mover. Right now this is only valid when the Mover is destined for a Station objective, and provides a string name for that station.

### .CurrentTrack

*POINTER TO Track*

> Returns a pointer to the track that this mover is currently assigned to.

Use the ^ operator to dereference the pointer and query track properties

```javascript
currentTrackId := Mover[1].CurrentTrack^.Id
```

### .IsCollisionAvoidanceActive

*BOOL*

> Returns true if collision avoidance is affecting the desired motion profile of this mover

```javascript
IF Mover[1].IsCollisionAvoidanceActive THEN
	// additional commands
END_IF;
```

### .IsErrorMover

*BOOL*

> Returns TRUE if the current Mover reference is an Error Mover. An Error mover is automatically returned when an invalid command is given to an interface capable of returning a Mover. See [Diagnostics / ErrorMovers](../Diagnostics/ErrorMovers.md) for more information.  

```javascript
//Returns an ErrorMover as the GetMover function works on an 1 based Index

MoverReference REF= Zone[0].GetMover(0,MC_Direction.MC_Positive_Direction);
IF NOT MoverReference.IsErrorMover THEN
	// Will not execute as the MoverReference is an error mover.
END_IF;
```

### .IsSyncedToAxis

*BOOL*

> Returns true if the mover is slaved to an external axis and has successfully reached the following position specified by the Master & Slave Sync Positions

### .IsSyncedToMover

*BOOL*

> Returns true if the mover is slaved to another mover and has successfully reached the following position specified by the Gap.

### .IsTrackReady

*BOOL*

> Returns true when the ActivateTrack() method has completed successfully. When using track management motion commands should not be issued unless this value is true or the mover will throw an error.

When using track management it can take several PLC scans for the .ActivateTrack method to complete. It is advised to wait for this to return after issuing an .ActivateTrack() command to avoid motion errors.


### .MasterMover

*REFERENCE To Mover*

> Reference variable that refers to the Mover this Mover is currently slaved to

When this mover is not slaved to another mover, .MasterMover is an invalid reference. There is therefore an ErrorMover object which will return information on what action was requested by the object while .MasterMover was invalid. This will show up in TwinCAT's Event Logs. See [Diagnostics / ErrorMovers](../Diagnostics/ErrorMovers.md) for more information.  

It is recommended that all evaluations are nested inside IF checks for .IsSyncedToMover OR by calling __ISVALIDREF

### .MotionParameters

*MotionParameters_typ*

> Defines a structure containing the dynamics settings for the Mover. Any new motion commands issued will utilize these values.

!!! Note
	Despite listing this value as a Property here in the documentation, MotionParameters are actually defined as a regular Input to the Mover object. This allows component access to the members of the STRUCT, which is not possible for Properties.

```javascript
STRUCT
	Jerk         : LREAL;                           // mm/s³
	Acceleration : LREAL;                           // mm/s²
	Deceleration : LREAL;                           // mm/s²
	Velocity     : LREAL;                           // mm/s
	Direction    : Tc3_Mc3Definitions.MC_Direction; // MC_Direction
	Gap			 : LREAL;							// mm
	GapMode		 : MC_GAP_CONTROL_MODE := MC_GAP_CONTROL_MODE.mcGapControlModeGroupDefault; // default to mode defined in collision avoidance object
END_STRUCT
```

Modifying a mover's Motion Parameters will not affect the current dynamics of a mover enroute. Updated dynamics properties will only take effect when the next motion command is issued.

It is also possible to alter motion paramaters with mover methods such as `.SetVelocity()`, `.SetAcceleration()` and others. These methods implicitly call .ReissueCommand() and therefore take effect immediately.

### .NextMover

> Returns the next mover in line with respect to this mover. Specifically, it returns the closest mover with a greater Position value than the current mover, regardless of actual direction of travel.


### .Payload

> Data associated with the product on the mover.

The code uses a structure with some example data that may be relevant to an in-process part on the mover. The structure is located in the PLC Project > DUTs > MoverPayload_typ.

```javascript
TYPE MoverPayload_typ :
STRUCT
	// data related to the in-process part on the mover
	// can be modifited to suit the application
	RecipeId		: DINT;
	FailureReason	: DINT;
	SerialNumber	: STRING;
END_STRUCT
END_TYPE
```

This data travels with the mover and when a mover arrives at a station it can be queried or modified based on the station's process. This structure can be modified as needed to suit the application.

Note that because .Payload is a property that returns a structure an intermediate variable must be used for access as shown in the examples below.

```javascript
// query the recipe ID and react accordingly
VAR
	MoverPayload : MoverPayload_typ;
END_VAR

IF (Station[1].MoverInPosition) THEN
	// get the payload
	MoverPayload := Station[1].CurrentMover.Payload;
	// query the payload
	IF MoverPayload.RecipeID = 1 THEN
		// add recipe 1 specific code here
	ELSE
		// add code specific to other recipes here
	END_IF;
END_IF;
```

```javascript
// Store a serial number for the part on the mover from a scanned barcode
VAR
	MoverPayload : MoverPayload_typ;
END_VAR

IF (ScanComplete) THEN
	// get the payload
	MoverPayload := Station[1].CurrentMover.Payload;
	// store data to the payload, then the mover
	MoverPayload.SerialNumber := ScannedBarcode;
	Station[1].CurrentMover.Payload := MoverPayload;
	// send mover to next station
	Station[1].CurrentMover.MoveToStation(Station[2]);
END_IF;
```
### .PreviousMover

> Returns the previous mover in line with respect to this mover. Specifically, it returns the closest mover with a lesser Position value than the current mover, regardless of actual direction of travel.


### .TrackInfo

> Returns information provided by MC_ReadTrackPositions for use with track management.

!!! Note
	Despite listing this value as a Property here in the documentation, TrackInfo is actually defined as a regular Output from the Mover object. This allows component access to the members of the STRUCT, which is not possible for Properties.

| Member | Type | Description |
|---|---|---|
| TrackId | OTCID | The hardware ID of the track the mover is assigned to |
| PartId | OTCID | The hardware ID of the part the mover is physically present on |
| TrackPosition | LREAL | Position of the mover measured from the zero point of the track. This may differ from Mover.AxisReference.NcToPLC.ActPos in certain track configurations |
| PartPosition | LREAL | Position of the mover measured from the zero point of the part the mover is physically on|



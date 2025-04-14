
# Mover List Object

> The Mover List object provides a way to group Movers together and issue commands to every Mover in the list. Alternatively, commands can be sent to individual movers within the list based on their geographic proximity to a track position.

Many of the methods and properties below are group commands based on corresponding methods available and documented in the singular [Mover](./Mover.md) object.

## Setup & Execution

```javascript
// Declaration
MoverListA		: MoverList;
```

```javascript
// Usage
MoverListA.RegisterMover( Mover[1] );
MoverListA.RegisterMover( Mover[2] );
MoverListA.RegisterMover( Mover[3] );

MoverListA.SetAllVelocity( 1000 );
MoverListA.MoveAllToStation( Station[4] );

MoverListA.UnregisterAll();
```

MoverLists must also be added to the Mediator object. By default, this is handled already in the MAIN.Initialize ACTION.

```javascript
// Example implementation
Mediator.AddMoverList( MoverListA );
```

## Chainability and related objects

The [`Track`](./Track.md) and [`Zone`](./Zone.md) objects keep internal mover lists that can be accessed with the `.CurrentMoverList` property. And many of the methods below return mover lists.

Additionally a mover list is extended from [`Objective`](./Objective.md) which provides two additional and very helpful properties `.TrackedMoverCount` and `.TrackedMovers`.

The combination of these functionalities can allow for powerful and concise handling of movers throughout the system. Some examples follow.

```javascript
// find all movers on track 2 with a destination station of 3 and redirect them to station 4
Track[2].CurrentMoverList.FilterDestinationStation(Station[3]).MoveAllToStation(Station[4]);
```
```javascript
// count the number of movers in Zone 2 and Zone 3
MoverCount := Zone[2].CurrentMoverList.LogicalIntersect(Zone[3].CurrentMoverList).TrackedMoverCount;
```

## Methods

### ActivateAllTrack

*ActivateAllTrack( Track: Track)*

> Issues individual ActivateTrack commands to every mover registered with the list.

The property IsAllTrackReady can be used to query for the completion of this method. Note that ActivateAllTrack and IsAllTrackReady may need to operate on different lists when working with Zone and Track lists as both Zone and Track are track-aware and only return movers that are both assigned to the track that the track or zone is also assigned to.

See [Mover.ActivateTrack](Mover.md#activatetrack) and for additional notes about track management.

```javascript
// state machine
100:
	Track[1].CurrentMoverList.ActivateAllTrack(Track[2]);
	// check track 2 for command completeness as the movers are being reassigned to this track
	IF (Track[2].CurrentMoverList.IsAllTrackReady) THEN
		iState := 200;
	END_IF;

200:
	// movers on track 2 ready for commands
```
### ApplyAllParameterSet
*ApplyAllParameterSet( ParameterSet : MotionParameters_typ )*

> Applies a set of motion paramaters to then entire mover list.

This only sets the motion parameters that will be used for the next motion command. Any in process motion will not be affected by this method. To immediately change motion paramaters use methods such as `.SetAllVelocity` or `.SetAllAcceleration`.

### Contains

*Contains( Mover: Mover )*

> Returns TRUE if the MoverList already contains the input Mover

```javascript
IF MoverListA.Contains( Mover[4] ) THEN
	// Yes, Mover 4 must exist in MoverListA
END_IF
```

### FilterDestinationStation

*FilterDestinationStation( Destination : REFERENCE TO Station ) : MoverList*

> Filters the mover list to only include movers that have a destination that matches the parameter Destination.

Filtering a mover list by station can be helpful when workings with several movers in a zone. There are two typical use cases. The first is to avoid re-triggering a move to station command after code such as `Zone[1].CurrentMoverList.MoveAllToStation(Station[2])` has executed, but movers are still accelerating out of this zone. The second is to ignore movers that may have a non-typical destination such as a rejected part in an otherwise normal flow of good parts.

### DisableAll
*DisableAll()*

> Calls the Disable() method for all movers in the list.

### EnableAll

> Calls the Enable() method for all movers in the list.

### GetMoverByLocation

*GetMoverByLocation( Index : USINT, Position : LREAL, Direction : Tc2_MC2.MC_Direction ) : REFERENCE TO Mover*

> Returns a reference to a singular mover from the Mover List, based on it's geographic location relative to a fixed track position.

**Index** specifies the number of movers that should lie between the selection and the Position input. Therefore Index = 1 would be the closest mover to the input position (in a given direction), Index = 2 would be the second closest, Index = 3 would be the third closest, etc.

**Position** specifies the target around which mover proximity should be considered.

**Direction** specifies the direction around the track *from which* the movers are indexed. Therefore MC_Positive_Direction will begin returning movers with the *most positive* absolute position values that are still *less than* the position input. In linear, non-closed tracks MC_Shortest_Way can be used to find movers in both directions using the absolute value of the difference between the fixed Position and the mover position.

```javascript
// Select a mover in the MoverList
// Which is the first closest mover to position 900
// And which has a position less than 900
// Then sends it to a station
MoverListA.GetMoverByLocation( 1, 900, MC_Positive_Direction ).MoveToStation( Station[3] );

// Select a mover in the MoverList
// Which is the third closest mover to position 2000
// And which has a position greater than 3000
// Then sets its acceleration
MoverListA.GetMoverByLocation( 3, 3000, MC_Negative_Direction ).SetAcceleration( 1E4 );

```
### Halt All

*HaltAll()*

> Calls the Halt() method for all movers in the list.

### LogicalComplement

*LogicalComplement()*

> Returns the logical complement of the current MoverList. Thus the returned list will list all movers not in the current MoverList.

```javascript
// System contains: M1, M2, M3, M4, M5
// MoverListA =     M1, M2, M3
MoverListA.LogicalCompliment(); // returns: M4, M5
```


### LogicalDifference

*LogicalDifference( MoverList : MoverList )*

> Returns the logical difference of the current MoverList and the provided MoverList. Thus the returned list will list all movers in the current MoverList (MoverListA) and not in the provided MoverList (MoverListB).

```javascript
// MoverListA = M1, M2, M3
// MoverListB =         M3, M4, M5
MoverListA.LogicalDifference( MoverListB ); // returns: M1, M2
```


### LogicalIntersect

*LogicalIntersect( MoverList : MoverList )*

> Returns the logical intersection of the current MoverList and the provided MoverList. Thus the returned list will list all movers in the current MoverList (MoverListA) that are also in the provided MoverList (MoverListB).

```javascript
// MoverListA = M1, M2, M3
// MoverListB =         M3, M4, M5
MoverListA.LogicalIntersect( MoverListB ); // returns: M3
```


### LogicalUnion

*LogicalUnion( MoverList : MoverList )*

> Returns the logical union of the current MoverList and the provided MoverList. Thus the returned list will list all movers in the current MoverList (MoverListA) and in the provided MoverList (MoverListB) deduplicated.

```javascript
// MoverListA = M1, M2, M3
// MoverListB =         M3, M5
MoverListA.LogicalUnion( MoverListB ); // returns: M1, M2, M3, M5
```


### MoveAllToPosition

*MoveAllToPosition( DestinationPosition : LREAL )*

> Issues individual MoveToPosition commands to every mover registered with the list.

```javascript
MoverListA.MoveAllToPosition( 1200 );
```


### MoveAllToStation

*MoveAllToStation( DestinationStation: Station )*

> Issues individual MoveToStation commands to every mover registered with the list.

```javascript
MoverListA.MoveAllToStation( Station[3] );
```


### MoveAllVelocity

*MoveAllVelocity( DesiredVelocity : LREAL )*

> Issues individual MoveVelocity commands to every mover registered with the list.

```javascript
MoverListA.MoveAllVelocity( 300 );
```


### SetAllAcceleration

*SetAllAcceleration( DesiredAcceleration : LREAL )*

> Sets the acceleration motion parameter equal to the input argument for every mover in the list

```javascript
MoverListA.SetAllAcceleration( 1E3 );
```

!!! note
	This method causes new motion commands to be issued for the dynamics to take effect immediately- potentially triggering a large number of motion commands to execute on the same scan. See the individual Mover command and consider the ramifications for your application.


### SetAllDeceleration

*SetAllDeceleration( DesiredDeceleration : LREAL )*

> Sets the deceleration motion parameter equal to the input argument for every mover in the list

```javascript
MoverListA.SetAllDeceleration( 15000 );
```

!!! note
	This method causes new motion commands to be issued for the dynamics to take effect immediately- potentially triggering a large number of motion commands to execute on the same scan. See the individual Mover command and consider the ramifications for your application.


### SetAllDirection

*SetAllDirection( Direction : Tc3_Mc3Definitions.MC_DIRECTION )*

> Sets the direction motion parameter equal to the input argument for every mover in the list

```javascript
MoverListA.SetAllDirection( mcDirectionPositive );
```

!!! note
	This method causes new motion commands to be issued for the dynamics to take effect immediately- potentially triggering a large number of motion commands to execute on the same scan. See the individual Mover command and consider the ramifications for your application.

### SetAllGap

*SetAllGap( Gap : LREAL )*

> Sets the gap for every mover in the list

```javascript
MoverListA.Gap( 65.0 );
```

!!! note
	This method causes new motion commands to be issued for the dynamics to take effect immediately- potentially triggering a large number of motion commands to execute on the same scan. See the individual Mover command and consider the ramifications for your application.

### SetAllGapMode

*SetAllGapMode( Mode : Tc3_Mc3Definitions.MC_GAP_CONTROL_MODE )*

> Sets the gap mode for every mover in the list

```javascript
MoverListA.SetAllGapMode( mcGapControlModeStandard );
```

!!! note
	This method causes new motion commands to be issued for the dynamics to take effect immediately- potentially triggering a large number of motion commands to execute on the same scan. See the individual Mover command and consider the ramifications for your application.

### SetAllJerk

*SetAllJerk( DesiredJerk : LREAL )*

> Sets the jerk motion parameter equal to the input argument for every mover in the list

```javascipt
MoverListA.SetAllJerk( 1e5 );
```

!!! note
	This method causes new motion commands to be issued for the dynamics to take effect immediately- potentially triggering a large number of motion commands to execute on the same scan. See the individual Mover command and consider the ramifications for your application.


### SetAllVelocity

*SetAllVelocity( DesiredVelocity : LREAL )*

> Sets the velocity motion parameter equal to the input argument for every mover in the list

```javascript
MoverListA.SetAllVelocity( 2000 );
```

!!! note
	This method causes new motion commands to be issued for the dynamics to take effect immediately- potentially triggering a large number of motion commands to execute on the same scan. See the individual Mover command and consider the ramifications for your application.

### UnregisterAll

*UnregisterAll()*

> Unregisters every mover from the list immediately

```javascript
MoverListA.UnregisterAll();
```

## Properties

### .IsAllMoversDisabled

*BOOL*

> Queries all movers registered with the mover list for the their `.Ready` property and returns true if all movers are not ready.

```javascript
// state machine
	100:
		// disable movers
		MoverList[1].DisableAll();
		state := 200;
	200:
		// wait for all movers to be disabled
		IF (MoverList[1].AllMoversDisabled) THEN
			state := 300;
		END_IF;
	300:
		// movers are disabled
```

### .IsAllMoversHalted

*BOOL*

> Queries all movers registered with the mover list for their `.Moving` property and returns true if all movers are not moving

```javascript
	// state machine
		100:
			// halt all movers in zone 1
			Zone[1].CurrentMoverList.HaltAll();
			state := 200;
		200:
			// wait for all movers to be stopped
			IF (Zone[1].CurrentMoverList.AllMoversHalted) THEN
				state := 300;
			END_IF;
		300:
			// movers in zone 1 are stopped
```

### .IsAllMoversReady

*BOOL*

> Queries all movers registered with the mover list for their `.Ready` property and returns true if all mover are ready.

```javascript
// state machine
	100:
		// enable movers
		MoverList[1].EnableAll();
		state := 200;
	200:
		// wait for all movers to be ready
		IF (MoverList[1].AllMoversReady) THEN
			state := 300;
		END_IF;
	300:
		// movers are enable and ready for motion commands
```

### .IsAllTrackReady

*BOOL*

> Queries all movers registered with the mover list for their IsTrackReady property and returns true when all movers have this property set true.

Note: This routine intentionally returns false if no movers are on the track. This is to handle the several scan delay between activating a track and the mover reporting the track is active. In a typical use case .ActivateAllTrack and .IsAllTrackReady are called back to back. Without this exception code folling .IsAllTrackReady would falsely assume the track switch is complete if the track was empty causing mover motion commands to throw errors.

See ActivateAllTrack() for an example.

### .TrackedMoverCount

*USINT*

!!! Note
	This property is part of [Objective](./Objective.md#trackedmovercount) but is frequently used with mover lists and can be accessed as part of the MoverList object.

### .TrackedMovers

*ARRAY OF POINTER TO Mover*

!!! Note
	This property is part of [Objective](./Objective.md#trackedmovers) but is frequently used with mover lists and can be accessed as part of the mover list.

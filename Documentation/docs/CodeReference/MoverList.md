
# Mover List Object

> The Mover List object provides a way to group Movers together and issue commands to every Mover in the list. Alternatively, commands can be sent to individual movers within the list based on their geographic proximity to a track position

---
<br>
<br>

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

---
<br>
<br>

## Methods

### ActivateAllTrack

*ActivateAllTrack( Track: Track)*

> Issues individual ActivateTrack commands to every mover registered with the list.

The property IsAllTrackReady can be used to query for the completion of this method. Note that ActivateAllTrack and IsAllTrackReady may need to operate on different lists when working with Zone and Track lists as both Zone and Track are track-aware and only return movers that are both assigned to the track that the track or zone is also assigned to.

See [Mover](Mover.md).ActivateTrack and for additional notes about track management.

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

---
<br>
<br>

### Contains

*Contains( Mover: Mover )*

> Returns TRUE if the MoverList already contains the input Mover

```javascript
IF MoverListA.Contains( Mover[4] ) THEN
	// Yes, Mover 4 must exist in MoverListA
END_IF
```

---
<br>
<br>
<br>

### GetMoverByLocation

*GetMoverByLocation( Index : USINT, Position : LREAL, Direction : Tc2_MC2.MC_Direction ) : REFERENCE TO Mover*

> Returns a reference to a singular mover from the Mover List, based on it's geographic location relative to a fixed track position.

**Index** specifies the number of movers that should lie between the selection and the Position input. Therefore Index = 0 would be the closest mover to the input position (in a given direction), Index = 1 would be the second closest, Index = 2 would be the third closest, etc.

**Position** specifies the target around which mover proximity should be considered.

**Direction** specifies the direction around the track *from which* the movers are indexed. Therefore MC_Positive_Direction will begin returning movers with the *most positive* absolute position values that are still *less than* the position input. In linear, non-closed tracks MC_Shortest_Way can be used to find movers in both directions using the absolute value of the difference between the fixed Position and the mover position.

```javascript
// Select a mover in the MoverList
// Which is the first closest mover to position 900
// And which has a position less than 900
// Then sends it to a station
MoverListA.GetMoverByLocation( 0, 900, MC_Positive_Direction ).MoveToStation( Station[3] );

// Select a mover in the MoverList
// Which is the third closest mover to position 2000
// And which has a position greater than 3000
// Then sets its acceleration
MoverListA.GetMoverByLocation( 2, 3000, MC_Negative_Direction ).SetAcceleration( 1E4 );

```

---
<br>
<br>

### MoveAllToPosition

*MoveAllToPosition( DestinationPosition : LREAL )*

> Issues individual MoveToPosition commands to every mover registered with the list.

```javascript
MoverListA.MoveAllToPosition( 1200 );
```

---
<br>
<br>

### MoveAllToStation

*MoveAllToStation( DestinationStation: Station )*

> Issues individual MoveToStation commands to every mover registered with the list.

```javascript
MoverListA.MoveAllToStation( Station[3] );
```

---
<br>
<br>

### MoveAllVelocity

*MoveAllVelocity( DesiredVelocity : LREAL )*

> Issues individual MoveVelocity commands to every mover registered with the list.

```javascript
MoverListA.MoveAllVelocity( 300 );
```

---
<br>
<br>

### SetAllAcceleration

*SetAllAcceleration( DesiredAcceleration : LREAL )*

> Sets the acceleration motion parameter equal to the input argument for every mover in the list

```javascript
MoverListA.SetAllAcceleration( 1E3 );
```

---
<br>
<br>

### SetAllDeceleration

*SetAllDeceleration( DesiredDeceleration : LREAL )*

> Sets the deceleration motion parameter equal to the input argument for every mover in the list

```javascript
MoverListA.SetAllDeceleration( 15000 );
```

---
<br>
<br>

### SetAllDirection

*SetAllDirection( Direction : Tc3_Mc3Definitions.MC_DIRECTION )*

> Sets the direction motion parameter equal to the input argument for every mover in the list

```javascript
MoverListA.SetAllDirection( mcDirectionPositive );
```

---
<br>
<br>

### SetAllJerk

*SetAllJerk( DesiredJerk : LREAL )*

> Sets the jerk motion parameter equal to the input argument for every mover in the list

```javascipt
MoverListA.SetAllJerk( 1e5 );
```

---
<br>
<br>

### SetAllVelocity

*SetAllVelocity( DesiredVelocity : LREAL )*

> Sets the velocity motion parameter equal to the input argument for every mover in the list

```javascript
MoverListA.SetAllVelocity( 2000 );
```

---
<br>
<br>

### StopAll

*StopAll()*

> Stops all movers in the list by calling their individual Stop methods

```javascript
MoverListA.StopAll();
```

---
<br>
<br>

### UnregisterAll

*UnregisterAll()*

> Unregisters every mover from the list immediately

```javascript
MoverListA.UnregisterAll();
```

## Properties

#### .IsAllTrackReady

*BOOL*

> Queries all movers registered with the mover list for their IsTrackReady property and returns true when all movers have this property set true.

Note: This routine intentionally returns false if no movers are on the track. This is to handle the several scan delay between activating a track and the mover reporting the track is active. In a typical use case .ActivateAllTrack and .IsAllTrackReady are called back to back. Without this exception code folling .IsAllTrackReady would falsely assume the track switch is complete if the track was empty causing mover motion commands to throw errors.

See ActivateAllTrack() for an example.
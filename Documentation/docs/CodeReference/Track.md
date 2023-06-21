# Track Object

> The track object contains functionality used for track management (switching movers between different tracks). In includes functionality for:

- Attaching a track hardware ID number to this object
- Assigning a user-friendly numerical TrackID for use in code
- Finding and addressing all movers on a track

## Use

The track object is only necessary when using track management. For most applications that utilize a single, closed loop track the Track object does not need to be addressed directly. Movers, zones, stations and position triggers all reference a track internally but are set with defaults of TrackID 0 which is a special case that eliminates the need for the Track object in these situations.

---

## Setup

It is recommended, but not required, to declare Tracks as an array. Starting at the zero index is also recommended.
```javascript
// Instance declaration
Track			: ARRAY [0..GVL.NUM_TRACKS] OF Track;

```
Tracks then need to have an ID and OTCID assigned to them. Index 0 should use 0 for both values as this is a special case for the absolute hardware position that may be useful during startup or recovers of some track configurations.

OTCID is found in System > TcCOM Objects > XTS Processing Unit 1 > Track # and is labeled Object ID on the Object tab. Note that the value displayed uses the "0x" hex notation, but needs to be entered using "16#" hex notation in the code

For simplicity ID should match the array index. This application already includes code that assigns the ID to match the array index of the track.


```javascript
		// If a single logical track is used (typically aa closed loop) then tracks to not need to be initialized
		Track[0].OTCID := 0;				// special "absolute reference" case for a single closed loop
		Track[1].OTCID := 16#010100B0;
		Track[2].OTCID := 16#010100D0;
		// additional tracks

```
## Use

Tracks are used in three ways:

- Parameters to other objects
- Managing movers on a track
- Properties of a mover

### Tracks as parameters

When configuring a Station, Zone or Position trigger a TrackID can be attached to each. If you are not using track management, a default TrackID is assigned to these objects that will allow them to work without explicitly assigning a TrackID.

Stations, Zones and Position Triggers will only return or act on movers that are assigned to the same track as the object's TrackId

```javascript
// position trigger at 150.0 mm on track 2
PositionTrigger[1].Position := 150.0;
PositionTrigger[1].TrackId  := 2;

// Zone between 200.0 and 300.0 mm on track 3
Zone[1].StartPosition := 200.0;
Zone[1].EndPosition   := 200.0;
Zone[1].TrackId       := 3;

// Station at 250.0 mm on track 3
Station[1].Position := 250.0;
Station[1].TrackId  := 3;
```
---

### Tracks managing movers

Tracks maintain a MoverList that can be used to track and query the movers on the track. See the [MoverList](MoverList.md) object for complete details. Some common use cases are shown below

```javascript
// move all movers on track to a station
// movers will queue at the station by using built-in collision avoidance
Track[1].CurrentMoverList.MoveAllToStation(Station[1]);

// find the mover closest to a fixed position on this track and set the velocity
Track[1].CurrentMoverList.GetMoverByLocation(0, 150.0, MC_Positive_Direction).SetVelocity(500.0);

// get the count of movers on this track
iMoverCount := Track[1].CurrentMoverCount;
```

### Properties of a mover

Every mover is assigned to a track. Movers can only move to stations and positions on the track they are currently assigned to.

See the property [Mover](Mover.md) for additional information.

---
<br>
<br>

## Methods

The methods listed below are used internally within the base code and should not be called directly by the user. They are shown for completeness.


### RegisterMover

*RegisterMover( NewMover : REFERENCE TO Mover )*

> This is used internally by Mover.TrackCyclic()

---
<br>
<br>


### UnregisterAll

*UnregisterAll( )*

> This is used internally by the Track Object

---
<br>
<br>


### UnregisterMover

*unRegisterMover( RemovingMover : REFERENCE TO Mover)*

> This is used internally by Mover.TrackCyclic()

---
<br>
<br>

## Properties

### CurrentMoverList

*MoverList (Ready Only)*

> Provides a MoverList object reference, containing all Movers that are currently assigned to this track. As a MoverList, methods are provided to command all movers as a group. See [MoverList](MoverList.md) for more information

### CurrentMoverCount

*DINT (Read Only)*

> Returns the count of movers currently assigned to this track

### Id

*DINT*

> Stores the user-friendly numeric track identifier.

This property is read/write, but should only be written once during initialization and should not change during execution. It is provided for querying and comparing TrackIds between objects.

### OTCID

*OTCID*

> Stores the hardware id of the track object found in XTSProcessingUnit 1

This property is read/write, but should only be written once during initialization and should not change during execution. It is used internally by Mover.ActivateTrack()

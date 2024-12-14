# Track Object

> The track object contains functionality used for track management (switching movers between different tracks). In includes functionality for:

- Assigning a user-friendly numerical TrackID for use in code
- Finding and addressing all movers on a track
- Checking the status of the track initialization

## Description

Tracks are a software defined group of motor modules that can be grouped together independently of the physical machine layout. Tracks allow you to switch movers between different groups of otherwise disconnected motor modules. A common use case is an oval system layout with a reject spur. Movers typically travel around the oval during normal production, but stop at a track "switch" which moves between the oval and a separate reject spur.  

The track object is only necessary when using track management. For most applications that utilize a single, closed loop track the Track object does not need to be addressed directly. Movers, zones, stations and position triggers all reference a track internally but are set with defaults of TrackID 1 which is generated automatically by the configuration tool.

## Setup

Tracks are declared as an array and should be numbered starting at 1 to match the numbering assigned by the XTS configurator. The provided code does this automatically and will find the correct hardware address for each track in the array during the initialization phase.

Tracks must also be added to the Mediator object. By default, this is handled in the MAIN.Initialize ACTION

```javascript
// Example implementation
Mediator.AddTrack( Track );
```

!!! Important
	Tracks must receive their hardware ID from the XTS Processing Unit before they can be used. This is handled automatically by the `.Cyclic()` method but may take several PLC scans at startup before the track can be utilized in code. The property `.isInitialized` is provided to check that this process has been completed before using the track in code.

## Use

Tracks are used in three ways in software:

- Parameters to other objects
- Managing movers on a track
- Properties of a mover

### Tracks as parameters

When configuring a Station, Zone or Position trigger a TrackID can be attached to each. If you are not using track management, a default TrackID is assigned to these objects that will allow them to work without explicitly assigning a TrackID.

Stations, Zones and Position Triggers will only return or act on movers that are assigned to the same track as the object's TrackId

```javascript
// position trigger at 150.0 mm on track 2
PositionTrigger[1].Position := 150.0;
PositionTrigger[1].SetTrack(Track[2]);

// Zone between 200.0 and 300.0 mm on track 3
Zone[1].StartPosition := 200.0;
Zone[1].EndPosition   := 200.0;
Zone[1].SetTrack(Track[3]);

// Station at 250.0 mm on track 3
Station[1].Position := 250.0;
Station[1].SetTrack(Track[3]);
```

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

### Mover Direction Considerations

When using tracks where some are closed loops (ovals) and others are not (spurs), it is important to pay attention to the direction mode of the mover with `Mover.SetDirection()`. By default this is set to `mcDirectionPositive` which works well for ovals and will automatically wrap positions (modulo) as the mover reaches the rollover point.

When a non-closed section of track is used (such as a rework spur), the mover commands `Mover.MoveToPosition` and `Mover.MoveToStation` as well as the related commands in a `MoverList` do not work with the default direction. When a mover is switched to a non-closed track, `Mover.SetDirection(mcDirectionNonModulo)` should also be called to set the mover's positioning to absolute along this portion of the track. Then when transitioning back to a closed section of track `Mover.SetDirection(mcDirectionPositive)` should be called to revert to the default modulo operation.


## Methods

The methods listed below are used internally within the base code and should not be called directly by the user. They are shown for completeness.

### Cyclic

Handles the initialization of the Track object by finding its hardware OTCID.

### RegisterMover

*RegisterMover( NewMover : REFERENCE TO Mover )*

> This is used internally by Mover.Cyclic() to manage the list of movers associated with this track.

### UnregisterAll

*UnregisterAll( )*

> This is used internally by the Track Object



### UnregisterMover

*unRegisterMover( RemovingMover : REFERENCE TO Mover)*

> This is used internally by Mover.Cyclic() to manage the list of movers associated with this track.


## Properties

### CurrentMoverList

*MoverList (Read Only)*

> Provides a MoverList object reference, containing all Movers that are currently assigned to this track. As a MoverList, methods are provided to command all movers as a group. See [MoverList](MoverList.md) for more information

### CurrentMoverCount

*DINT (Read Only)*

> Returns the count of movers currently assigned to this track

### Id

*DINT*

> Stores the user-friendly numeric track identifier.

This property is read/write, but should only be written once during initialization (handled by `MAIN.Registering()`) and should not change during execution. It is provided for querying and comparing TrackIds between objects.

### OTCID

*OTCID*

> Stores the hardware id of the track object found in XTSProcessingUnit 1

This property is read/write, but should only be written once during initialization (handled by `.Cyclic()`) and should not change during execution. It is used internally by `Mover.ActivateTrack()`.

By default during the initialization step all track OTCIDs are assigned automatically.
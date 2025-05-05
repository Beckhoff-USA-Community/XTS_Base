
# Station Object

> The Station object provides a fixed location for a Mover to stop at until it is ready to be released. A Station is a type of Objective, and provides a handle to the Mover object that is currently docked with the Station, allowing further motion commands to be issued to the Mover.


## Setup & Execution

```javascript
// Declaration
Station		: ARRAY [0..GVL.NUM_STATIONS-1] OF Station;
```

```javascript
// Initialization
Station[1].Position		:= 250;
Station[2].Position		:= 500;
Station[3].Position		:= 750;
```
Stations must also be added to the Mediator object. By default, this is handled automatically.

```javascript
// Example implementation
Mediator.AddStation( Station[1] );
```


## Mover Registration

Unlike other types of Objectives, Stations manage mover registration automatically. Any mover that is commanded with a [MoveToStation](Mover.md#movetostation) is also automatically registered with that station. As an example, this implementation would be redundant:

```javascript
Mover[2].MoveToStation( Station[3] );
Station[3].RegisterMover( Mover[2] );		// unnecessary
```

Stations also automatically *unregister* movers that have been redirected with another move command, even if that command's destination is the same location as the Station.

```javascript
Mover[2].MoveToStation( Station[3] );
Mover[2].MoveToPosition( Station[3].Position );
```

Here, the Station will not report *MoverInPosition* when it arrives because the original command is interrupted by one where the *CurrentMoveType* is not `MOVETYPE_STATION`.


## Methods

### Cyclic

*Cyclic()*

> Stations require a cyclic call in the Main program. This allows the station to automatically unregister movers from its Tracked list who have been redirected and are no longer destined for this Station.

The call to Cyclic() is handled automatically after the station has been registered with the Mediator.

### Reset Statistics

*ResetStatistics()*

> Resets the Statistics for this station. See the [Statistics](#statistics) property.

### Set track

*SetTrack(<Track>)*


```javascript
Station[1].Position := 1000.0;
Station[1].SetTrack(Track[2]);
```

## Properties

### .Blocked

*BOOL*

> A station is considered blocked if a mover has left the station but that mover has not yet moved further than the mover's `.Gap`.

This value is used internally to calculate some values in the `.Statistics` property by detecting when a station does not have a mover, but also can not receive a mover because of the gap maintained by collision avoidance.

### .CurrentMover

*REFERENCE TO Mover*

> Reference variable that refers to the Mover that is docked with the Station

When no mover is present in the Station, .CurrentMover is an invalid reference. In this case, an ErrorMover will replace the invalid reference. The user will receive notifications when this happens in the TwinCAT Event Logs.

It is recommended that all evaluations are nested inside IF checks for [.MoverInPosition](#moverinposition).

### .Description

*STRING*

> A text description of the station that is passed to the visualization tools. It's also visible within within the TwinCAT debugging tools for easier identification of stations.

This value can be changed at runtime and will update on the visualization. This offers the opportunity to put dynamic data on the XTS visualization such as a station part count.

### .GroupSize

*DINT*

> When multiple stations perform the same operation they are considered "grouped". This value represents the number of stations in the group.

This value is only used to help calculate throughput `.Statistics` on the mover and is used in combination with the `.Blocked` property.


### .MoverInPosition

*BOOL*

> Status boolean indicating that a mover is currently docked with the Station


### .TrackedMoverCount

*USINT*

> Simply reports the number of Movers that are currently registered with the Station. Because Stations automatically deregister movers that are not currently destined for this Station, this value also represents the current number of incoming movers.

!!! Note
	This property is part of [Objective](./Objective.md#trackedmovercount) but is frequently used with stations and can be accessed as part of the Station object.

### .Position

*LREAL*

> Current placement of the Station along the track

This value is also used to place a marker on the visualization tools representing the station. See [Visualization](../../GettingStarted/Visualization.md).


### .Statistics

> A set of timers and counts that track the throughput and utilization of the station.

| Member | Type | Description |
|--|--|--|
| TimeOccupied				| TIME |  Current amount of time that the station has had a MoverInPosition |
| TimeEmpty | TIME |  Current amount of time where MoverInPosition is FALSE |
| TimeBlocked | TIME | Current amount of time where station cannot receive a new mover because the previous mover has not moved far enough away from the station |
| OccupiedTimeHistory | ARRAY[0..63] OF TIME | Ring buffer of recent TimeOccupied values |
| EmptyTimeHistory | ARRAY[0..63] OF TIME |  Ring buffer of recent TimeEmpty values |
| BlockedTimeHistory | ARRAY[0..63] OF TIME | Ring buffer of recent TimeBlocked values | 
| AverageTimeOccupied			| TIME |  Average of all nonzero TimeOccupied values in the ring buffer|
| AverageTimeEmpty			| TIME |  Average of all nonzero TimeEmpty values in the ring buffer |
| AverageTimeBlocked		| TIME |  Average of all nonzero TimeBlocked values in the ring buffer |
| AverageTimeBetweenMovers	| TIME |  Sum of average TimeOccupied + AverageTimeEmpty |
| AverageUtilizationFactor	| LREAL |  Percentage value equal to AverageTimeOccupied / AverageTimeBetweenMovers|
| AverageMoversPerMinute		| LREAL |  MPM Throughput value, equal to 60 / AverageTimeBetweenMovers [s]	|
| ThrottledThreshold		| LREAL |  A high throttled threshold (Blocked to Empty ratio) indicates a potential bottleneck station	|
| ProcessedMoverCount		| UDINT |  Total number of movers processed by this station |

### .Track

*REFERENCE TO Track*

> Track that the station is assigned to when using track management. See the [Track](Track.md) object.

### .TrackId

*DINT*

> The numeric id of the track that this station is assigned to.

This value is read-only. Use the method `.SetTrack()` to set the station's assigned track.




## Extra Examples

```javascript
// Basic Station implementation with a 1s dwell time
IF Station[0].MoverInPosition THEN
	StationTimer( IN := TRUE, PT := T#1000MS );	// TON

	IF StationTimer.Q THEN
		Station[0].CurrentMover.MoveToStation( Station[1] );
	END_IF
ELSE
	StationTimer( IN := FALSE );
END_IF
```

```javascript
// Immediately redirect all incoming shuttles to Station 1 instead of Station 0
FOR i := 0 TO Station[0].TrackedMoverCount-1 DO
	targetMover := Station[0].TrackedMovers[i];
	targetMover^.MoveToStation( Station[1] );
END_FOR
```
### Station Labeling In Viewing Tools

See [Visualization](../../GettingStarted/Visualization.md)
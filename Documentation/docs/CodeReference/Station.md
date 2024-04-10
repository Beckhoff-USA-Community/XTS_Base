
# Station Object

> The Station object provides a fixed location for a Mover to stop at until it is ready to be released. A Station is a type of Objective, and provides a handle to the Mover object that is currently docked with the Station, allowing further motion commands to be issued to the Mover.

---
<br>
<br>

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

```javascript
// Call this method cyclically
FOR i := 0 TO GVL.NUM_STATIONS - 1 DO
	Station[i].Cyclic();
END_FOR
```

Stations must also be added to the Mediator object. By default, this is handled in the MAIN.Initialize ACTION.

```javascript
// Example implementation
Mediator.AddStation( Station[1] );
```

---
<br>
<br>

## Mover Registration

Unlike other types of Objectives, Stations manage mover registration automatically. Any mover that is commanded with a [MoveToStation](../CodeReference/Mover.md#movetostation) is also automatically registered with that station. As an example, this implementation would be redundant:

```javascript
Mover[2].MoveToStation( Station[3] );
Station[3].RegisterMover( Mover[2] );		// unnecessary
```

Stations also automatically *unregister* movers that have been redirected with another move command, even if that command's destination is the same as the Station.

```javascript
Mover[2].MoveToStation( Station[3] );
Mover[2].MoveToPosition( Station[3].Position );
```

Here, the Station will not report *MoverInPosition*.

---
<br>
<br>

## Methods

### Cyclic

*Cyclic()*

> Stations require a cyclic call in the Main program. This allows the station to automatically unregister movers from its Tracked list who have been redirected and are no longer destined for this Station.

```javascript
FOR i := 0 TO GVL.NUM_STATIONS - 1 DO
	Station[i].Cyclic()
END_FOR
```
---
<br>
<br>

### Reset Statistics

*ResetStatistics()*

> Resets the Statistics for this station. See the [Statistics](#statistics) property.


---
<br>
<br>

## Properties

### .MoverInPosition

> Status boolean indicating that a mover is currently docked with the Station

---
<br>
<br>

### .CurrentMover

> Reference variable that refers to the Mover that is docked with the Station

When no mover is present in the Station, .CurrentMover is an invalid reference. In this case, an ErrorMover will replace the invalid reference. The user will receive notifications when this happens in the TwinCAT Event Logs.

It is recommended that all evaluations are nested inside IF checks for [.MoverInPosition](#moverinposition).

---
<br>
<br>

### .TrackedMoverCount

> Simply reports the number of Movers that are currently registered with the Station. Because Stations automatically deregister movers that are not currently destined for this Station, this value also represents the current number of incoming movers.

---
<br>
<br>

### .Position

> Current placement of the Station along the track

---
<br>
<br>

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


---
<br>
<br>

#### .TrackId
*DINT*
> Track that the station is assigned to when using track management. See the [Track](Track.md) object.
---
<br>
<br>


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

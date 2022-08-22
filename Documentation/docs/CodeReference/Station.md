
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
Station[1].TrackPosition		:= 250;
Station[2].TrackPosition		:= 500;
Station[3].TrackPosition		:= 750;
```

```javascript
// Call this method cyclically
FOR i := 0 TO GVL.NUM_STATIONS - 1 DO
	Station[i].Cyclic();
END_FOR
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
Mover[2].MoveToPosition( Station[3].TrackPosition );
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

## Properties

#### .MoverInPosition

> Status boolean indicating that a mover is currently docked with the Station

---
<br>
<br>

#### .CurrentMover

> Reference variable that refers to the Mover that is docked with the Station

When no mover is present in the Station, .CurrentMover is an invalid reference. In this case, an ErrorMover will replace the invalid reference. The user will receive notifications when this happens in the TwinCAT Event Logs.

It is recommended that all evaluations are nested inside IF checks for .MoverInPosition.

---
<br>
<br>

#### .TrackedMoverCount

> Simply reports the number of Movers that are currently registered with the Station. Because Stations automatically deregister movers that are not currently destined for this Station, this value also represents the current number of incoming movers

---
<br>
<br>

#### .TrackPosition

> Current placement of the Station along the track

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
	Station[0].TrackedMovers[i]^.MoveToStation( Station[1] );
END_FOR
```

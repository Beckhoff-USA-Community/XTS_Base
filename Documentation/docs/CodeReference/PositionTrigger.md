
# Position Trigger Object

> Position Trigger objects monitor movers until they have crossed over a specified location on track, even if they do not stop there. Position Triggers will always point to the most recent registered mover which crossed over the Position Trigger's track position

---
<br>
<br>

## Setup & Execution

```javascript
// Declaration
PositionTriggerA		: PositionTrigger;
```

```javascript
// Initialization
PositionTriggerA.TrackPosition		:= 2000;
PositionTriggerA.TriggerDirection	:= mcDirectionPositive;
```

---
<br>
<br>

## Methods

### Cyclic

*Cyclic()*

> Position Triggers require a cyclic call in the Main Program because they need to constantly monitor the positions of registered movers.

---
<br>
<br>

### MuteCurrent

*MuteCurrent()*

> Clears a Mover reference from the CurrentMover property, and resets MoverPassedPosition to be false. This can be done without the need to unregister a mover from a position trigger

Since Position Triggers uniquely *latch* their CurrentMover output, it is necessary to *clear* this value manually in order to prepare to process the next mover event.

```javascript
IF PositionTrigger[0].MoverPassedPosition THEN
	PositionTrigger[0].CurrentMover.MoveVelocity( 200 );
	PositionTrigger[0].MuteCurrent();
END_IF
```

In this way, it is possible to *reset* the position trigger without having to completely unregister and re-register a mover from the trigger.


---
<br>
<br>

## Properties

### .MoverPassedPosition

> Status boolean indicating that at least one mover has passed over the threshold position since being registered

Position Triggers are unique in that the Current Mover output *latches* even though the mover may not still qualify under the trigger condition

---
<br>
<br>

### .CurrentMover

> Reference variable that refers to the most recent registered mover to cross over the track position since being registered

- If no movers are tracked, or no tracked movers have yet satisfied the objective, then .CurrentMover is an invalid reference. In this case, an ErrorMover will replace the invalid reference. The user will receive notifications when this happens in the TwinCAT event logs. See Diagnostics > ErrorMovers for more information.

- For this reason, it is recommended that all evaluations are nested inside IF checks for .MoverPassedPosition.

- It is possible that multiple registered movers have crossed over a threshold position. Only the most recently valid mover handle is provided by .CurrentMover.

---
<br>
<br>

### .TrackPosition

> Current placement of the Position Trigger threshold along the track

---
<br>
<br>

## Extra Examples

```javascript
// Basic Position Trigger implementation

// Begin monitoring Mover 1
IF bInit THEN
	Mover[1].MoveToPosition( 1000 );
	PositionTrigger.TrackPosition	:= 800;
	PositionTrigger.RegisterMover( Mover[1] );
	bInit	:= FALSE;
END_IF

// When Mover 1 crosses position 800, slow it down to 100mm/s
IF PositionTrigger.MoverPassedPosition THEN
	PositionTrigger.CurrentMover.SetVelocity( 100 );
	// MuteCurrent clears MoverPassedPosition and CurrentMover so that
	// Line 12 does not continually evaluate as true
	PositionTrigger.MuteCurrent();
END_IF

```

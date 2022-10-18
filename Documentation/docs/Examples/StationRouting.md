
# Station Routing

> This example demonstrates the most basic Station to Station routing. There are two stations which dwell for a period of time before sending movers to the next station in the sequence

```javascript


MS_RUN: 	// ----------------------------------- OPERATING


	// Station 0 Logic
	IF Station[0].MoverInPosition THEN
		StationTimer[0]( IN := TRUE, PT := T#1000MS );

		IF StationTimer[0].Q THEN

			Station[0].CurrentMover.SetVelocity( 500 );
			Station[0].CurrentMover.MoveToStation( Station[1] );

		END_IF
	ELSE
		// Reset the timer for the next mover
		StationTimer[0]( IN := FALSE );
	END_IF




	// Station 1 Logic
	IF Station[1].MoverInPosition THEN
	StationTimer[1]( IN := TRUE, PT := T#2s );

		IF StationTimer[1].Q THEN

			// Same behavior as above, but utilizing method chaining
			Station[1].CurrentMover.SetVelocity( 90 ).MoveToStation( Station[0] );

		END_IF
	ELSE
		// Reset the timer for the next mover
		StationTimer[1]( IN := FALSE );
	END_IF

```

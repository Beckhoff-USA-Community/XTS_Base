
# Zone Object

> The Zone object provides a way to select all Movers within a specified geographic area on the track. A Zone is defined by a Start Position and an End Position, and returns every mover between these points. Zones are especially useful for programming recovery scenarios. When using track management, zones make use of TrackId and only return movers that are both within the zones defined geographic area and have a matching TrackId.


## Setup & Execution

```javascript
// Declaration
ZoneLeftSide		: Zone;
ZoneRightSide		: Zone;
```
```javascript
// Initialization
ZoneLeftSide.StartPosition	:= 0;
ZoneLeftSide.EndPosition	:= 2000;

ZoneRightSide.StartPosition	:= 2000;
ZoneRightSide.EndPosition	:= 4000;
```

Zones must also be added to the Mediator object. By default, this is handled automatically.

```javascript
// Example implementation
Mediator.AddZone( ZoneLeftSide );
Mediator.AddZone( ZoneRightSide );
```


## Methods

### GetMover

*GetMover( Index : USINT, Direction : Tc2_MC2.MC_Direction) : REFERENCE TO Mover*

> Provides a reference to the Mover that has *n = Index* movers between it and the end of the Zone specified by the Direction

```javascript
// Selects the mover closest to the Zone's End Position
// Then issues a motion command
ZoneLeftSide.GetMover( 1, MC_Positive_Direction ).MoveToPosition( 2000 );

// Selects the fourth-closest mover to the Zone's Start Position
// Then sets its velocity
ZoneRightSide.GetMover( 4, MC_Negative_Direction ).SetVelocity( 1200 );
```

### Set track

*SetTrack(<Track>)*


```javascript
Zone[1].StartPosition := 1000.0;
Zone[1].EndPosition := 500.0;
Zone[1].SetTrack(Track[2]);
```

## Properties

### .CurrentMoverCount

*USINT*

> Provides a count of the number of movers that are currently withing the boundaries set by Start Position and End Position.

```javascript
IF Zone.CurrentMoverCount < 3 THEN
	// code to allow more movers into this zone
END_IF
```

### .CurrentMoverList

*MoverList*

> Provides a MoverList object reference, containing all Movers that are currently within the boundaries set by Start Position and End Position. As a MoverList, methods are provided to command all movers as a group. See [MoverList](MoverList.md) objective for more information

```javascript
// Command all movers from position 0 through 2000 to Station#1
ZoneLeftSide.CurrentMoverList.MoveAllToStation( Station[1] );

// Command all movers from position 2000 through 4000 to Station#2
ZoneRightSide.CurrentMoverList.MoveAllToStation( Station[2] );

// Command the singular mover which is closest to the end of the zone to Station#3
Zone.CurrentMoverList.GetMoverByLocation(0,Zone.EndPosition,MC_Positive_Direction).MoveToStation( Station[3] );

```
### .EndPosition

*LREAL*

> Defines the upper bound for the track region considered by the Zone object.

### .StartPosition

*LREAL*

> Defines the lower bound for the track region considered by the Zone object.

### .Track

*REFERENCE TO Track*

> Track that the station is assigned to when using track management. See the [Track](Track.md) object.

### .ZoneLength

*LREAL*

> Calculates the length of the defined zone, in millimeters.

## Properties provided by MoverList

Some properties provided by the [MoverList](MoverList.md) and [Objective](Objective.md) have similar syntax but different meanings.

```javascript
// returns the number of movers currently registered with the Zone
Zone.TrackedMoverCount;

// returns the number of movers physically within the zone
Zone.CurrentMoverList.TrackedMoverCount;
Zone.CurrentMoverCount;
```

Take care when using these properties.



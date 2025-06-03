
# Design Approach

## Fundamentals

The core concept driving the project structure is that it is useful to address movers **contextually** as opposed to **directly**. With that mantra in mind, this code provides a suite of tools to:

1. Select a subset of movers that satisfy certain conditions
2. Issue high-level motion commands to those selected movers

---

High level commands are provided as methods, like this:

```javascript
Mover[1].Enable();
        .Disable();
        .MoveToPosition(targetPosition);
        .MoveToStation(targetStation);
        .SyncToAxis(...);
        ...etc.
```

Loads of different motion methods are available. Generally these are wrappers for PLCopen function blocks, and follow patterns you'd expect from typical point-to-point motion programming. 

---

Let's look at a single command in more depth:

```javascript
Station[2].Position     := 1200; // mm

Mover[1].MoveToStation( Station[2] );
```

Here, we *select* **Mover[1]**, and issue it a command move to position 1200. We'll cover Stations in a bit.

The code above functions as you would expect. Except, in this example we're *directly* addressing our mover. Instead, let's send this command to *whichever mover is currently located at Station[1]*:

```javascript
Station[1].Position     := 600; // mm
Station[2].Position     := 1200; // mm

Station[1].CurrentMover.MoveToStation( Station[2] );
```

Here, *Station[1].CurrentMover* is a POINTER to whichever mover currently sits at this position. Now our command functions regardless of which specific mover arrives at the station. It might be Mover #1, or Mover #2. In many cases, we don't really care. Furthermore, this logic will work regardless of the number of movers configured on our track, whether it's 2 or 200.

---

We missed one thing in the code above. We can't always guarantee that Station[2] *actually has* a mover docked. So we need to add a check:

```javascript
Station[1].Position     := 600; // mm

// Verify that a 'Current Mover' actually exists before commanding it
IF Station[1].MoverInPosition THEN
    Station[1].CurrentMover.MoveToStation( Station[2] );
END_IF
```

If we missed this addition, *Station[1].CurrentMover* could be a null pointer, and calling a method on this null pointer is bad.

---

As you've seen, Stations provide access to any mover currently docked at the station's location. But there are other objects which can also be used to select movers based on context. Here is the complete list:

- **Stations** select a mover that is "docked" with the Station
- **PositionTriggers** select a *registered* mover that has "crossed over" the Trigger
- **MoversLists** select all *registered* movers that you assign to the List
- **Zones** select all *registered* movers that are contained within a defined region

In addition to these *selectors*, it is even possible to select movers relative to other movers:

```javascript
// Selects Mover[2]
Mover[1].NextMover...
```


---

Let's go a little further with our Station example:

```javascript
Station[1].Position     := 600; // mm
Station[2].Position     := 1200; // mm

IF Station[1].MoverInPosition THEN
    myTimer[1]( IN := TRUE, PT := t#2s );    
    IF myTimer[1].Q THEN
        Station[1].CurrentMover.MoveToStation( Station[2] );
    END_IF;
ELSE
	myTimer[1]( IN := FALSE );
END_IF;

IF Station[2].MoverInPosition THEN
    myTimer[2]( IN := TRUE, PT := t#2s );    
    IF myTimer[2].Q THEN
        Station[2].CurrentMover.MoveToStation( Station[1] );
    END_IF;
ELSE
	myTimer[2]( IN := FALSE );
END_IF;
```

Now, we've got two Stations where movers stop for 2 seconds, after which they recieve a command to release onwards to the other station. Now let's imagine we run this logic with 5 movers on a small track. For the majority of the cycle, 2 movers will be stopped at the stations while the other 3 are in-transit somewhere between them. This code will function successfully, because the movers are able to negotiate traffic without any additional input from you. This brings us to the next topic:

## Collision Avoidance

Every motion command within this project implements Collision Avoidance. This feature is provided by TF5410.

The functionality is simple: movers will attempt to complete their commands as directed, but will decelerate appropriately behind other movers like cars on the highway. This behavior is parameterizable:

```javascript
Mover[1].MotionParameters.Gap       := 120;     // mm

Mover[1].MoveVelocity( 2000 );                  // mm/s
Mover[2].MoveVelocity( 100 );                   // mm/s
```

Here, Mover #1 is *behind* Mover #2, but commanded to move much faster. In this scenario Mover #1 will catch up to Mover #2 and follow along behind, leaving a center-to-center gap between the two of 120mm.

---

Collision avoidance removes a significant amount of work for you, the developer. In the example below, a single command will send every mover on the track (regardless of the # of movers) to a Station. The first mover to arrive will "dock" with the station, and can be handled with code like the examples above. The rest of the movers will sit and wait patiently with a *pending* motion command. As soon as the first mover leaves the station, the queue indexes forward and the next mover in line docks with the station.

```javascript
// Send every mover on the system to Station #4
System.CompleteMoverList.MoveAllToStation( Station[4] );
```

## Registration

Stations, Position Triggers, Zones, and MoverLists can only *select* (i.e. return) movers that have been **Registered** to them. Here's an example to illustrate:

```javascript
Station[1].Position     := 500; // mm

Mover[1].MoveToPosition( 500 ); // mm

Station[1].MoverInPosition; // will continue to return FALSE
```

In the example above, Mover #1 will arrive directly on top of the station, but *Station[1].MoverInPosition* will never return TRUE because the Mover was not *explicitly* sent to the Station.

```javascript
Station[1].Position     := 500;

Mover[1].MoveToStation( Station[1] );

Station[1].MoverInPosition; // will return TRUE once the mover arrives
```

Above, the command is an explicit directive. In this case, the .MoveToStation() command automatically **registers** the Mover with the Station.

Manually managing registration is easy. These methods are common to **Mover Lists**, **Position Triggers**, **Stations**, and **Zones**:

```javascript
// Manually register Movers #1-3 into a custom list
myMoverList.RegisterMover( Mover[1] );
myMoverList.RegisterMover( Mover[2] );
myMoverList.RegisterMover( Mover[3] );

// Register Movers #1-3 with a position trigger via that list
myPositionTrigger.RegisterMoverList( myMoverList );

// Manually unregister Mover #2. Movers #1 & #3 remain.
myPositionTrigger.UnregisterMover( Mover[2] );

// Manually unregister every mover from the custom list
myMoverList.UnregisterAll();

// Or, do it from the Mover itself
Mover[3].UnregisterFromAll();
```

Much of this is done for you, automatically:

- Registration with Stations is automatically managed by the .MoveToStation method call
- Registration for Position Triggers occurs during initialization. By default, all movers are registered to all Position Triggers. Unregistering movers from Position Triggers can be done manually. If a mover is not registered with a Position Trigger, the trigger will completely ignore the mover.
- Registration for Zones occurs during initialization. By default, all movers are registered to all Zones. Unregistering movers from Zones can be done manually. If a mover is not registered with a Zone, the zone will never return that mover.
- Registration for MoverLists is entirely manual, by design.





## Method Chaining

Most methods in the project return *the current object itself*, allowing for additional method calls on the same line of code. This approach is often referred to as a *fluent interface*. For example:

```javascript
// All these method calls apply to Mover 1 in sequence
Mover[1].SetVelocity( 2000 ).SetAcceleration( 20000).MoveToPosition( 1000 );
```

In some cases, the *order* of the methods calls must be carefully considered. In the following example, the *CurrentMover* of *Station #1* is commanded to move to *Station #2*. At this point, it is no longer considered the *CurrentMover* of *Station #1*. As a result, the *SetVelocity* method does not apply to the intended Mover object.

```javascript
IF Station[1].MoverInPosition THEN
    // The MoveToStation method returns an ErrorMover
    // The SetVelocity method will not function correctly
    Station[1].CurrentMover.MoveToStation(Station[2]).SetVelocity(1000);
END_IF;
```

However, the following example will work as intended:

```javascript
IF Station[0].MoverInPosition THEN
    // This line of code will function correctly
    Station[0].CurrentMover.SetVelocity(1000).MoveToStation(Station[1]);
END_IF;
```



# Project Overview

## Broad Concept

The core concept driving the project structure is that it is useful to address movers **contextually** as opposed to **directly**.

To illustrate, consider a scenario of recovering from an E-Stop where a robot is positioned directly in the path of the movers. In this instance, a first step might be to command any movers *downstream* of the robot to move further downstream, and movers *upstream* of the robot should be commanded further upstream in order to create some space for the arm to reset.

As a programmer, one wouldn't seek to specifically command Mover #4 and Mover #5 to carry out those instructions because during the next occurrence it may be an entirely different pair of movers in this situation. Instead, the commands are issued to *whichever movers are closest to the robot right now*.

In this sense, we take a list of all movers on the track system, and apply *filters* to help select only the movers we care to command. These filters are applied via a family of objects called [Objectives](./Objects/Objective.md).

## Method Chaining

Most methods in the project return *the current object itself*, allowing for additional method calls on the same line of code. This approach is often referred to as a *fluent interface*. For example:

```javascript
// All these method calls apply to Mover 0 in sequence
Mover[0].SetVelocity( 2000 ).SetAcceleration( 20000).MoveToPosition( 1000 );
```

In some cases, the *order* of the methods calls must be carefully considered. In the following example, the *CurrentMover* of *Station#0* is commanded to move to *Station#1*. At this point, it is no longer considered the *CurrentMover* of *Station#0*. As a result, the *SetVelocity* method does not apply to the intended Mover object.

```javascript
IF Station[0].MoverInPosition THEN
	// This line will return an ErrorMover and will not function as intended
	Station[0].CurrentMover.MoveToStation(Station[1]).SetVelocity(1000);
END_IF;
```

However, the following example will work as intended:

```javascript
IF Station[0].MoverInPosition THEN
	// This line of code will function correctly
	Station[0].CurrentMover.SetVelocity(1000).MoveToStation(Station[1]);
END_IF;
```

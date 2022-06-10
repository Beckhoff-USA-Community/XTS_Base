
### Programming Quick Tips

Here are a few quick tips to assist in developing with this project:

- Methods are designed to be called once, not repeatedly.

- Collision avoidance should be the source of buffering, there should rarely be a reason for a buffer location to be user defined. 

- Movers Contain an AxisRef, it can be useful to put 'Mover[i].AxisReference.ReadStatus();' in the code with the  Cyclic call for the same, the Status values of an AxisRef can be very useful. 

```javascript
// Call this method cyclically
FOR i := 0 TO GVL.NUM_MOVERS-1] DO
	Mover[i].Cyclic( GroupRef );
    Mover[i].AxisReference.ReadStatus();
END_FOR
```

---

- When using *.CurrentMover* properties (or similar), always wrap usage with a check for *.MoverInPosition* or similar, to verify that the pointers are valid. See ErrorMovers for more information.

- When reconfiguring the project for new hardware, be sure to delete elements of a previous configuration. More info on this is included in the First Steps section.

- When configuring new hardware, be sure that *Is Closed* and *Is Included in Detection* options are set correctly.

- When programming recovery scenarios, be sure to unregister movers from objectives where necessary.

### General Track Tips

More generally, here are some recommendations regarding the architecture of your solution:

- Proper code should not depend on the number of movers added to the system, and should be easily scalable when movers are added or removed from the system. Directly addressing specific movers by ID should be avoided when possible. See Overview for more information.

- "The fleet sails as fast as the slowest ship." Overall system throughput is governed by the slowest individual Station's throughput. Focus on understanding the limiting factors of your routing logic in order to optimize traffic flow. A Mover's maximum velocity rarely affects the overall system throughput.

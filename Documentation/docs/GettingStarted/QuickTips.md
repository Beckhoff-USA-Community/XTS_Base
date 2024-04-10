
## Programming Quick Tips

Here are a few quick tips to assist in developing with this project:

- Methods are designed to be called once, not repeatedly.

- Collision avoidance should be the source of buffering, there should rarely be a reason for a buffer location to be user defined. 

- Movers Contain an AxisRef. The status values of an AxisRef can be very useful. 

- When using *.CurrentMover* properties (or similar), always wrap usage with a check for *.MoverInPosition* or similar, to verify that the pointers are valid. See ErrorMovers for more information.

- When reconfiguring the project for new hardware, be sure to delete elements of a previous configuration. More info on this is included in the First Steps section.

- When configuring new hardware, be sure that *Is Closed* and *Is Included in Detection* options are set correctly.

- When programming recovery scenarios, be sure to unregister movers from objectives where necessary.

## General Code Tips

More generally, here are some recommendations regarding the architecture of your solution:

- Proper code should not depend on the number of movers added to the system, and should be easily scalable when movers are added or removed from the system. Directly addressing specific movers by ID should be avoided when possible. See Overview for more information.

- "The fleet sails as fast as the slowest ship." Overall system throughput is governed by the slowest individual Station's throughput. Focus on understanding the limiting factors of your routing logic in order to optimize traffic flow. A Mover's maximum velocity rarely affects the overall system throughput.

## Common issues

### Changing the forward direction of the track

Depending on the machine layout and the mounting needs of the motor modules the default direction of the track positions (clockwise for face-up motor modules) may not be desired. It is possible to invert the track's direction by changing the following settings.

For each Mover axis in the NC set `Enc > Parameter > Encoder Evaluation > Invert Encoder Counting Direction` to `TRUE`.

![Invert Encoder Direction](../Images/GettingStarted/InvertDirectionEncoder.png)

For each Mover axis in the NC set `Drive > Parameter > Output Settings > Invert Motor Polarity` to `TRUE`.

![Invert Drive Direction](../Images/GettingStarted/InvertDirectionDrive.png)

For each Track set in the XTS Processing Unit set `Parameter > General > Polarity` to `Negative`.

![Invert Track Direction](../Images/GettingStarted/InvertDirectionTrack.png)

There are some additional considerations to be aware of when inverting the track direction, especially if the track direction is changed after portions of code have been written and tested.

When looking for the next and previous movers on a system, for example when working with 2-up stations, movers that have been referenced directly such as Mover[i] and Mover[i+1] may no longer be in the expected order. Functions such as [`Mover.NextMover()`](../CodeReference/Mover.md#nextmover), [`Mover.PreviousMover()`](../CodeReference/Mover.md#previousmover) or [`Zone.CurrentMoverList.GetMoverByLocation`](../CodeReference/MoverList.md#getmoverbylocation) are the recommended methods will correctly handle an inverted track direction.


### MAIN state machine is stuck at MS_ACTIVATING_TRACKS

After reconfiguring the track shape or length, a new track hardware ID (`OTCID`) may be generated. The PLC needs to reference this new ID.

Check that the OTCID for the track is set under the PLC Project > Main Instance > Symbol Initialization tab. `MAIN.Track[0].OTCID` should remain at `00000000`. You can select `Track 1` or other tracks from the drop down list to automatically selected the correct OTCID.

![Track OTCID](../Images/OTCIDAssignment.png)


# Parameters

Paramaters configure the overall size of the XTS application including the number of objects such as stations or zones needed. Parameters can be viewed and adjusted in `GVL\Param`.

!!! Note
	Mover appears in parameters but is not addressed in this section as it has additional needs when increasing or decreasing its count. Please see [Project Customization](../GettingStarted/2_FirstSteps.md#project-customization).

## NUM_ Parameters

The provided FB_XTS object includes a set of the objects listed already pre-configured and ready to use.

- `XTS.Mover[]`
- `XTS.PositionTrigger[]`
- `XTS.Station[]`
- `XTS.Zone[]`

The size of these arrays is configured with the corresponding `NUM_` parameter.

In many XTS applications, especially ones that implement simple station-to-station motion, the provided objects are all that is needed for user code. An simple adjustment of the `NUM_` parameter to increase stations, zones or position triggers may be the only parameters that need to be adjusted to fit the application.

## MAX_ Parameters

The mediator can manage additional Position Triggers, Stations and Zones outside of those declared in FB_XTS if needed for an application. This can help with code modularity and allows you to create custom objects (as FBs) that contain their own stations, zones and position triggers as needed.

The `MAX_` parameters define the absolute maximum number of any of these objects that can be declared. The value selected should be high enough to include the number of objects created by the `NUM_` parameter, and any additional objects you will create in user code.

!!! Note
	There is no performance penalty for having more MAX_ objects than you actually use. Any objects in the array that have not been configured do not cause additional code to execute. There is a small amount of memory required for extra, unused objects but this often is insignificant in the overall size of the program and memory available on IPCs capable of running XTS.

## MAX_ Parameters Example

An example of declaring additional objects that consume space under the `MAX_` parameter limits is available in [Custom Objects](../Examples/CustomObjects.md)

## Trace Parameters

These parameters configure the tracing and logging system. They can be found alongside the other parameters in `GVL\Param`.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `TRACE_LEVEL` | `TcEventSeverityExt` | `Verbose` | Minimum severity level forwarded to loggers. Messages below this level are silently discarded. See [severity levels](Diagnostics/Trace.md#severity-levels). |
| `ENABLE_TC_EVENT_LOGGER` | `BOOL` | `TRUE` | Routes trace messages to the TwinCAT Event Logger (visible in the TwinCAT EventLog view). |
| `ENABLE_ADS_LOGGER` | `BOOL` | `FALSE` | Routes trace messages to the ADS output log (visible in the TwinCAT XAE output window). |
| `TRACE_LOGGERS` | `UDINT` | `2` | Maximum number of logger back-ends that can be subscribed simultaneously to the `Trace` function block. Increase this if custom loggers are added beyond `TcEventLogger` and `AdsLogger`. |

!!! Note
    `TRACE_LEVEL` controls what gets logged across **all** back-ends. Setting it to `Warning` suppresses `Verbose` and `Info` messages regardless of which loggers are enabled. Set it to `Off` to silence all output.
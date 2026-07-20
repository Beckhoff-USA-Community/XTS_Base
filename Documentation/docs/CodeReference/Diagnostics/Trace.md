
# Trace

The Trace system is the primary diagnostic logging mechanism in XTS Base. It replaces the earlier per-object Event Logger with a unified, application-wide publisher that routes structured messages to one or more configurable back-ends.

## Overview

The global instance `GVL_Trace.Trace` is a `Trace<MaxCount>` function block that implements the `iTrace` interface. Any code that holds a reference to `iTrace` can emit messages without depending on a specific back-end.

Two back-ends are included and instantiated automatically in `GVL_Trace`:

| Back-end | Output destination | Enabled by |
|---|---|---|
| `TcEventLogger` | TwinCAT EventLog (XAE / HMI) | `Param.ENABLE_TC_EVENT_LOGGER` |
| `AdsLogger` | ADS output window in XAE | `Param.ENABLE_ADS_LOGGER` |

Both back-ends self-subscribe to `GVL_Trace.Trace` during initialization based on their respective `Param` flags. See [Trace Parameters](../Parameters.md#trace-parameters) for configuration details.

## Severity Levels

Messages are emitted at one of five severity levels, defined by the `TcEventSeverityExt` enumeration:

| Value | Name | Typical use |
|---|---|---|
| `0` | `Verbose` | High-frequency, fine-grained diagnostic detail |
| `1` | `Info` | Normal operational milestones |
| `2` | `Warning` | Unexpected but recoverable conditions |
| `3` | `Error` | Recoverable failures requiring attention |
| `4` | `Critical` | Unrecoverable failures |
| `5` | `Off` | Suppresses all output (used for `TRACE_LEVEL` only) |

`Param.TRACE_LEVEL` acts as a threshold: a message is forwarded only when its severity is **greater than or equal to** `TRACE_LEVEL`. For example, setting `TRACE_LEVEL := TcEventSeverityExt.Warning` passes `Warning`, `Error`, and `Critical` messages while discarding `Verbose` and `Info`.

## Emitting Messages

Call the corresponding method on `Trace` (or on any `iTrace` reference):

```iecst
// Verbose — high-frequency diagnostic detail
Trace.Verbose('StationLogic', 'Mover arrived at Station 1');

// Info — normal milestone
Trace.Info('Initializing', 'XTS system ready');

// Warning
Trace.Warning('MoverCommand', 'Mover already at target station');

// Error
Trace.Error('Recovery', 'ErrorMover could not be reset');

// Critical
Trace.Critical('Track', 'XTS hardware fault detected');
```

Each method takes two `T_MaxString` arguments:

| Argument | Description |
|---|---|
| `Source` | Identifies the origin of the message (e.g., POU name, subsystem). Displayed as the event source in TwinCAT EventLog. |
| `Message` | The human-readable description of the event. |

## Rate Limiting

Both built-in back-ends enforce a per-scan message cap to prevent flooding the log during fast PLC cycles. The cap is derived from the task cycle time. Messages beyond this limit within the same scan are silently dropped until the next cycle.

## Adding a Custom Logger

Any function block that implements `iTraceLogger` can be subscribed as an additional back-end:

```iecst
// In FB_Init or initialization code:
GVL_Trace.Trace.Subscribe(MyCustomLogger);

// To remove it:
GVL_Trace.Trace.Unsubscribe(MyCustomLogger);
```

The maximum number of simultaneously subscribed loggers is set by `Param.TRACE_LOGGERS` (default: `2`). Increase this value before adding custom loggers beyond the two built-in ones.

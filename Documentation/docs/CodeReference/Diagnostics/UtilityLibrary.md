
# Utility Library

## Tc3_XTS_Utility

The XTS Utility library is installed as a component of TF5850 and provides advanced-level access to a huge amount of data and functionality in the XTS system. Nearly every value, parameter, or setting visible in the various windows of the XTS Configuration is accessible via this library.

Things like module temperatures, coil currents, and segment BTNs can all be read from the track system using this library. Write-access is provided to some parameters, allowing additional functionality like: on-the-fly tuning changes, manually triggering mover detection, and more.

## Documentation

Specific library documentation for Tc3_XTS_Utility can be found on Infosys in the link below.

[Infosys Tc3 XTS Utility Documentation](https://infosys.beckhoff.com/content/1033/xts_software/10350715275.html)

## Base Project Use

### Code Objects

An instance of the library's core object, **FB_TcIoXtsEnvironment**, is included in the Base Project for your use. Code within the Mediator also makes use of the utility library to perform bootstrapping functions such as mover 1 identification.

More code examples are provided in the Examples section below. Often, you can search through the interface inside TwinCAT using autocomplete to find the information you're looking for.

### Diagnostic VISU

An organized Diagnostic PLC VISU is also provided by the library, and has been added to this project. It contains a preconfigured GUI to check diagnostics for the XTS driver, parts, modules, movers, and tracks.

More information on this visualization tool is also available on the official Infosys page.

## General pattern

The accessors and methods in the XTS utility library refer to objects in a hierarchical order and contain groups of objects. For example, to access a motor module you must navigate through the XPU > Part > Module. It is also possible to have more than one XPU (XTS Processing Unit) on a system.

In the above example there is likely more than one part within the system, and multiple motor modules within the system. It is already recommended that you first query adjacent properties for the number of parts (motor modules, movers, etc) and do bounds checking before directly accessing an item.

The examples below show both how to query for the number of objects, and how to access data within those objects.

## Example Usage

All examples should first check for the XTS Environment to be ready. This is handled by the base code automatically, but takes several PLC scans to complete.

```javascript
// check that the XTS environment is ready
IF XTS.System.EnvironmentIsReady THEN
	// work with environment variables
END_IF
```

```javascript
// Get the number of Parts configured on the system
myPartCount	:= XTS.System.Environment.XpuTcIo(1).GetPartCount();
```

```javascript
// Get the number of Motor Modules configured on the system
myMotorCount	:= fbXtsEnvironment.XpuTcIo(1).PartTcIo(1).GetModuleCount();
```

```javascript
// get the number of movers in the system
myMoverCount := XTS.System.Environment.XpuTcIo(1).GetMoverCount()

// Get the total distance travelled for a all movers
FOR i := 1 TO myMoverCount DO
	// read the distance travelled per mover
	moverTotalDistance := moverTotalDistance + fbXtsEnvironment.XpuTcIo(1).MoverTcIo(i).GetDistanceDrivenInKm();
END_FOR;
```

```javascript
// Update mover tuning parameter (must be done with care)
fbXtsEnvironment.XpuTcIo(1).MoverTcIo(1).SoftDrive.VelocityControl.SetKp(1.5);
```

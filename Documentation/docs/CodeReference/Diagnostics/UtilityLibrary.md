
# Utility Library

## Tc3_XTS_Utility

The XTS Utility library is installed as a component of TF5850 and provides advanced-level access to a huge amount of data and functionality in the XTS system. Nearly every value, parameter, or setting visible in the various windows of the XTS Configuration is accessible via this library.

Things like module temperatures, coil currents, and segment BTNs can all be read from the track system using this library. Write-access is provided to some parameters, allowing additional functionality like: on-the-fly tuning changes, manually triggering mover detection, and more.

## Documentation

Specific library documentation for Tc3_XTS_Utility can be found on Infosys in the link below.

[Infosys Tc3 XTS Utility Documentation](https://infosys.beckhoff.com/english.php?content=../content/1033/tc3_xts_utility/index.html&id=1453293700768984828)

## Base Project Use

#### Code Objects

An instance of the library's core object, **FB_TcIoXtsEnvironment**, is included in the Base Project for your use along with a simple commmented example in the MAIN (PRG). More code examples are provided in the Examples section below. Often, you can search through the interface inside TwinCAT using autocomplete to find the information you're looking for.

#### Diagnostic VISU

An organized Diagnostic PLC VISU is also provided by the library, and has been added to this project. It contains a preconfigured GUI to check diagnostics for the XTS driver, parts, modules, movers, and tracks.

More information on this visualization tool is also available on the official Infosys page.

## Example Usage

```javascript
// Get the number of Motor Modules configured on the system
myMotorCount	:= fbXtsEnvironment.XpuTcIo(1).PartTcIo(1).GetModuleCount();
```

```javascript
// Get the total distance travelled for a specific mover
myMoverMileage	:= fbXtsEnvironment.XpuTcIo(1).MoverTcIo(1).GetDistanceDrivenInKm();
```

```javascript
// Update mover tuning parameter (must be done with care)
fbXtsEnvironment.XpuTcIo(1).MoverTcIo(1).SoftDrive.VelocityControl.SetKp(1.5);
```

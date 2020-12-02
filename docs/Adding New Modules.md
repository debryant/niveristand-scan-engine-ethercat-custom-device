## Overview

**Scan Engine Custom Device** supports various C series modules. In order to use newly released modules in VeriStand, custom device developers must add module support to [`Modules.lvlibp`](https://github.com/ni/niveristand-scan-engine-module-libraries) and add mutation code to [`Scan Engine Custom Device.lvlib`](https://github.com/ni/niveristand-scan-engine-ethercat-custom-device).

## Modules.lvlib

`RSI Module.lvclass` is the base class for all modules. The `RSI Module.lvclass` provides the functionality to configure and store settings for a module. Some modules have additional configuration which are implemented in derived classes (e.g. `9205.lvclass`). The class is serialized to store module configuration between sessions.

## Scan Engine Custom Device.lvlib

**Scan Engine Custom Device** hosts the module user configuration pages and provides mutation of VeriStand properties from version to version. Each release which adds new hardware must increment the version number.

## Adding New Modules

NOTE: Some of these steps may need to be run outside of the context of a LabVIEW project, as the project contains multiple targets resulting in the classes being locked.

1. Insert a new module enumeration into `RSI Module.lvclass:Module Models.ctl`.
1. Create a new class derived from `RSI Module.lvclass` if the module has additional configuration. For modules without additional configuration simply use `RSI Module.lvclass`.
    1. Override some or all of the abstract methods: `Initialize.vi`, `Configure Module.vi`, `Create Channels.vi`, and `Get UI Ref.vi`. The VI reference returned by `Get UI Ref.vi` should be a non-override member VI.
    1. Modules which are not supported in the source version of LabVIEW need to have their support saved-for-previous using a later version of LabVIEW, and require additional work. See `9224.class` for an example.
        1. Override `Get Minimum LabVIEW Version.vi` to disable the module from being selected in unsupported versions of VeriStand.
        1. Use a type-specialization structure in `Configure Module.vi` to avoid breaking the versions in which the module is not supported.
        1. Add a free label with the format `#MinVersion: yearnumber` to `Configure Module.vi` to assist in removing the conditional logic when it is no longer needed.
1. Add a case for the new module in `Modules.lvlib:Init Module.vi`. Initialize the module with appropriate subsystem and number of channels.
1. Update VI typedef in `Scan Engine Custom Device.lvlib:Get ECAT Config Wrapper.vi`.
1. Bump the version number in `Scan Engine Custom Device.lvlib:Custom Device Scan Engine.xml`. Bump the minor version for new hardware.
1. Add mutation code in `Scan Engine Custom Device.lvlib:Main - On Load.vi` to update the module model enumeration for older module classes.
1. Save the test system definition assets with the new support. This is required as mutation does not run when deploying programmatically.
1. Update the Modules.md document by adding the new module to the appropriate table.

## Adding Scripting Support for Modules

1. Open the `Custom Device Source/Scripting/Scan Engine Scripting.lvlib`
1. Identify if there are proper Mode and Channels
    1. If not, go to Adding New Modes
1. Create a new folder under XXXXX for the module
1. Create new class inheriting from Module.lvclass
1. Override the `Define Channel Count.vi` to specify the number of channels the Module has
1. Create a `Create NI XXXX - <mode name>.vi` and a `Read.vi` 
1. Open `Convert Module Model to Module Class.vi`
1. Update the case structure with your new class object
1. Open the appropriate polymorphic VI from `Modules/` which matches your mode
1. Add your `Create NI XXXX.vi` to that polymorphic vi

### Create Tests

1. Open Scan Engine.lvproj
1. Expand Tests\Unit\Scripting API\Scan Engine Scripting API Custom Device Tests.lvclass
1. Expand `tests`
1. Open `test_Set and Get 9225.vi` and save it as your own Module
    1. `Save as` and configure thusly

1. Add a check to the Scripting column

## Adding New Modes

If you have a unique channel type, or if your module has properties defined at 
the Module level you will have to make a new mode.

1. Open the Scan Engine.lvproj
1. Create a new class inheriting from Mode.lvclass
1. Add your unique Module-level data to the class
1. Add your accessors
1. Create a new `Create` vi under `Module.lvclass`
1. Create a new `Read` vi under `Module.lvclass`
    1. If you're creating a mode which inherits directly from `Mode.lvclass` you will need to create the following vis:
        * `Create Channel Array.vi`
        * `Create Channel.vi`
        * `Read Channel Array.vi`
1. Create a new `Convert <Your Mode> Module to Slot Configuration.vi`
1. Create a new `Convert to <Your Mode> Module.vi`


### Adding A Channel to a Mode

If you have a per-channel parameters which are not supported or if the default values for the existing channel does not match the value you need to support, you will need a new channel.

1. Create a new class inheriting from `Channel.lvclass`
1. Create new `Create` and `Read` vis which are unique for your new Channel

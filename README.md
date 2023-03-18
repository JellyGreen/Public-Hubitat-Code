# Public-Hubitat-Code

v1.0 (19 December 2022)

v1.1 (21 December 2022)

v1.2 (29 December 2022)

v1.6 (4 January 2023)

v2.6 (23 February 2023) - with Thermostat Capability for use with Thermostat Scheduler

_________________________________________

NOTES BELOW FOR EACH VERSION RELEASE
_________________________________________
v1.1

This is my first attempt at driver code.  I am only one month into any form of home automation.  So I have lots to learn and I am happy to have comments & suggestions in respect of this trial driver code.

I bought a hubitat C7 and a POPP zigbee radiator TRV (model 701721).  They are compatibile but I could not find an appropriate driver so I decided to write one.

It turns out that this TRV is a badged danfos TRV (Danfos Ally) for which there is a good published specification:

The following have proved key to the development of my (still limited) understanding:
  
'ZigBee Cluster Library Specification' - I used version 6 - ZigBee Document: 07-5123-06 - 14 January 2016.  This is not the easiest way to learn zigbee but it has proved to be an essential companion. https://zigbeealliance.org/wp-content/uploads/2019/12/07-5123-06-zigbee-cluster-library-specification.pdf

 Danfoss Ally eTRV1.18, Zigbee Cluster Specification, 08-11-2021 - again an essential specification for the TRV.  https://assets.danfoss.com/documents/193613/AM375549618098en-000102.pdf
 
A magic list of manufacture codes that are the key to making manufacturer specific attributes available for read / write. Created and kept on github by Chris Brandson -
 https://github.com/wireshark/wireshark/blob/cbbddcfa3a046157a26541a57bac7eb631148b5e/epan/dissectors/packet-zbee.h.  Thanks to Krassimir Kossev and Mike Maxwell for their tips in relation to this.  
 
The 2mqtt page relating to this product, https://www.zigbee2mqtt.io/devices/701721.html. I did not want to go down the 2mqtt route but this has useful descriptions of some of the TRV functions that are more fully documented in the Danfos Zigbee Cluster Spec.

Endless example drivers for other products, the (good but less than complete) hubitat documentation, lots of hubitat (and other devices) community chats.  

Special mention and thanks to ckpt-martin for his zigbee driver for the eCozy TRV. https://github.com/ckpt-martin/Hubitat/blob/fae5a4a455a209878c467fd6f91c70a7a9b223ca/README.md

So, how far have I got?  

Well I think I have a working driver - which I am using with Hubitat C7 to control three POPP TRVs with remote room temperature sensors (not necessary but a definite bonus whn controlling heat in the room not just near the TRV).  

This is still in development - so I have called it a 'trial driver' with 'version 1.0'  but is, I think, worth sharing with anyone else who might find it useful.

A few notes:

1.  I have not implemented this as a 'Thermostat' device driver.  I could not stand having all the HVAC controls and attributes messing up my screen when essentially all a TRV can do is heating.  So I have selected relevant capabilities from the Hubitat Capabilities List - https://docs2.hubitat.com/developer/driver/capability-list.
The only downside so far is that when writing associated Hubitat apps, in Rule Machine, TRV will not appear in a list of Thermostats - but you can get to all the available attributes and commands for the TRV by treating it as a temperature sensor and by setting custom attributes.

2.  As just stated, I have defined a number of custom attributes.  These are needed to work with the manufacturer specific functionality that this TRV has.  For example, the TRV has an 'orientation' setting, which makes a different to the accuracy of its local temperature mesurement; so I have a custom attribute called eTRVOrientation.  The attribute names are pretty self-explanitory but, see below and in the driver's Metadata for the list of attributes and implemented values.

The standard attributes (listed under Current States on the driver page once the driver is configured) are:

heatingSetpoint   - (obviously) the heating setpoint that the TRV will aim at - only celsius throughout this driver with permitted range 5 to 30 
                  - a command is implement for setting through the driver page or a hubitat app
                  
temperature       - the local temperature in celsius generated by the TRV's own sensor - read only

battery           - read only the remaining TRV's battery percentage

The custom attributes (also listed under Current States) are - at the moment: 

eTRVCallingForHeat : yes or no - read only - one day will be useful to fire up my boiler only when needed

eTRVExternalSensorTemperature : e.g. 19.2 - command is implement for setting through the driver page or a hubitat app  - celsius

eTRVOrientation : horizontal or vertical - toggles on the driver page

eTRVPIHeatingDemand : 0 to 100 - read only - the percentage that the TRV valve is openned by the TRV itself

eTRVRadiatorCovered : notCovered or covered - toggles on the driver page

eTRVViewingDirection : 'normal' or 'inverted' - toggles on the driver page  turns the digits over in the TRV display 


3.  The driver allows for the use of an external temperature sensor - but you will need to use Hubitat apps to provide regular remote temperature readings to the TRV. The max and minimum frequency Danfos suggest for sending this data depend on whether, in the driver, you set the TRV as 'covered' or 'notCovered' (custom attribute eTRVCovered.  The note in the Danfos specification on this says: (a) if TRV is set as not covered - External sensor temperature should be sent to it from hub at least every 3 hours but not more often than every 30 minutes @ every 0,1K change (and that after 3 hours the function is disabled and goes back to standard mode - operating based on the TRV's local temperature readings); (b) If if TRV is set as covered - External sensor temperature should be sent to it from hub at least every 30 minutes but not more often than every 5minutes @ every 0,1K change, and after 35 minutes the function is disabled and goes back to standard mode (attribute value -80)

4.  Events are generated by the driver only in the parse() method. i.e. on reading an attribute from the TRV.  So a zigbee write command is always coupled with a following read comand. This ensures that an Event is only generated when there is confirmation, through the parse() method, that the write command was effective.  It can take a few moments for this to occur.  Be patient.

5.  After installing the driver do press configure on the driver page.  The TRV will then be configured, via the config() method, to regularly report on key attributes, but not ones, like orientation and covered, which are unlikely ever to change once the TRV is set up.

6.  Yet to be implemented are things like - setting a weekly schedule (why would you when Hubitiat does that for you) - open windo detection - 'child' locking the TRV

I hope this is enough of an explanation.

Good luck...
Jon

p.s. - good tip I found in the community chats somewhere if you have been using another driver for the device already is to select and load 'Device' as a driver, use the device page to clear Current States and State Variables (requires the page to be refreshed after pressing the buttons) and then loading the device driver you want to use.


v1.1 (21 December 2022)
Implements user defined min and max heating setpoints between the manufacturer fixed values of 5 and 35; see attributes eTRVMinHeatSetpointLimit and eTRVMaxHeatSetpointLimit. Anttempt to set a heatingSetpoint value outside these limits is ignored by the driver.

v1.2 (29 December 2022)
Implements a Child Lock toggle in the deveice driver

v1.6 (4 January 2022) 
This implement most remaining features from the thermostat cluster (not scheduling ). 
Heatting Setpoint is set using the applicable thermostat cluster command with setting '01' for fast TRV reaction. (alternative obsolete code is included (but  'commented-out') in the driver 

The following attributes as used - NOTE the valid ENUM values:

        attribute "eTRVPIHeatingDemand", "number"                 //  0x0201 0x0008 PIHeatingDemand
        attribute "eTRVOrientation", "string"                     //  0x0201 0x4014 eTRV Orientation
        attribute "eTRVHeatingSetpointSource", "number"           //  0x201 0x0030 Setpoint Change Source  // not implemented yet
        attribute "eTRVExternalSensorTemperature", "number"       //  0x0201 0x4015 External Measured Room Sensor
        attribute "eTRVRadiatorCovered", "string"                 //  0x0201 0x4016 Radiator Covered - 0 for NOT covered, 1 for covered
        attribute "eTRVCallingForHeat", "string"                  //  0x0201 0x4031 Heat Supply Request
        attribute "eTRVViewingDirection", "string"                //  0x0204 0x4000 Viewing Direction
        attribute "eTRVMinHeatSetpointLimit", "number"            //  0x0201 0x0015 Min Heat Setpoint Limit / user set minimum, default and lower possible limit is 5°C celsius
        attribute "eTRVMaxHeatSetpointLimit", "number"            //  0x0201 0x0016 Max Heat Setpoint Limit / user set maximum, default and upper possible limit is 35°C celsius
	      attribute "eTRVLockState", "enum", ["unlocked", "locked"] //  0x0204 0x0001 Child Lock, default is unlocked alternative is locked (There are no apparent different levels of locking)
        attribute "eTRVAggressionFactor", "number"                //  0x0201 0x4020 Aggression Factor 1 to 10  0b0000 0001 to 0b0000 1010 note bit 4 0b000x 1 is enable and 0 is disable quick open feature
        attribute "eTRVHeatAvailable", "enum", ["no", "yes"]      //  0x0201 0x4030 Heat Available, 0 for No Heat Available; 1 for Heat Available 
        attribute "eTRVOpenWindowFeature", "enum", ["off", "on"]  //0x0201 0x4051 Open Window Feature on / off
        attribute "eTRVOpenWindowDetected", "enum", ["quarantine", "closed", "imminent", "open", "reported"]   //0x0201 0x4000 TRV Open Window Detection
        attribute "eTRVOpenWindowExtSen", "enum", ["closed", "open"]  // 0x0201 0x4003 TRV Open Window Detection
        attribute "eTRVLoadSharing",  "enum", ["off", "on"]       //  0x0201, 0x4032 TRV Load Balancing Enabled
        attribute "eTRVLoadRadiatorMean", "string"                //  0x0201, 0x4040, TRV Load Radiator Mean        
        attribute "eTRVLoadEstimate", "string"                    //  0x0201, 0x404A TRV Load Estimate for this radiator
        attribute "eTRVRegulationOffset", "number"                //  0x0201, 0x404B TRV Regulation Off-set -in steps of 0.1°C: range –2.5 °C to +2.5 °C (0xE7 … 0x19). 
        attribute "eTRVAdaptionRunControl", "string"              //  0x0201, 0x404C TRV Adaptation Run Control: default 00, 01 for initiate; 02 for cancel
        attribute "eTRVAdaptationRunStatus", "string"             //  0x0201, 0x404D TRV Adaptation Run Status: bit 0 in progress, bit 1 Valve Characteristic found, bit 2 Valve Characteristic lost
        attribute "eTRVAdaptationRunSetting", "enum", ["manual", "automatic"]    //  0x0201, 0x404E TRV Adaptation Run Settings: 00 default; 01 for enabled automatic     
        attribute "eTRVPreheatStatus", "string"                   //  0x0201, 0x404F TRV Preheat Status: 00 for off, 01 for on (default) - reportable
        attribute "eTRVPreheatTime", "string"                     //  0x0201, 0x4050 TRV Preheat Time: Time Stamp, default 00000000, max FFFFFFFF - reportable 
        attribute "eTRVExerciseDay", "enum", ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "undefined"] //TRV Exercise Day 00 for Sunday to 06 for Saturday, 07 for undefined
        attribute "eTRVExerciseTime", "string"                    //  0x0201, 0x4011, TRV Exercise Minutes from midnight max 1439
        attribute "eTRVMountingModeActive", "enum", ["no, mounted", "yes, not mounted or reset"]         //  0x0201, 0x4012, TRV Mounting Mode Active: 00 for mounted and 01 for not mounted or factory reset 
        attribute "eTRVMountingModeControl", "enum", ["Go to mounting mode", "Go to Mounted position"]   //  0x0201, 0x4013, TRV Mounting Mode Control: 00 for Ready for physical mounting on valve, 01 for mounted or as if mounted
                         
Also includes a 'switch' capability which sets the heating set point to Min Heat Setpoint Limit (off) or Max Heat Setpoint Limit (on).

You may need to read the Danfoss eTRV Zigbee Spec (see link above) to understand the attributes.

 *  Imortant Note
 *  Chat in HA community identifies a known trouble with the TRV rejecting attempts to write External Sensor Temperature when, as required by the Danfoss spec values are updated regularly.
 *  That problem is not an error in the driver but in the TRV firmware as far as I can tell.
 *  My  'work around' for this is to send write -80o as External Sensor Temperature first (turning the feature off) then pause for a few seconds and then send the actual External Sensor Temperature.  I am not clear whether this was successful and I now operate my TRVs successfully without external sensors.
 *  Although I had hoped to implement OTA firmware updating, I have not done this because I have been unable to find any firmware to update my eTRVs with.  

v2.6 Minor corrections in coding. 


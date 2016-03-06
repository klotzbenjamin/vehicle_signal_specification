(C) 2016 Jaguar Land Rover - All rights reserved.

All documents in this repository are available under the Creative
Commons Attribution 4.0 International (CC BY 4.0).

Click [here](https://creativecommons.org/licenses/by/4.0/) for
details.

All code in this repository is available under Mozilla Public License
v2 (MPLv2).

Click [here](https://www.mozilla.org/en-US/MPL/2.0/) for details.

# VEHICLE SIGNAL SPECIFICATION
This repository specifies a set of vehicle signals that can be used in
automotive applications to communicate the state of various vehicle
systems.

The collection of signal specifications, or simply signals, are vendor
independent. Vendor-specific extensions can be specified in a dediceated and
uncontrolled branch of the signal specification tree. 

The format of the directories and signal specification files is aimed
at allowing easy, git-based management with branching, merging, and
release.

A released signal specification can be used, together with tools in
this repository, to automatically generate a number of different
target specification formats, such as JSON, FrancaIDL, etc.

The release management process will be driven in the context of GENIVI
and its Remote Vehicle Interaction expert group.


# SIGNAL DEFINITION

A signal is a named entity, such as rpm, that at any time can have a
value, such as 3400.

Signals are organized into a tree such as outlined below.

![Signal tree](pics/tree.png)<br>
*Fig 1. A signal tree example*

## <a name="signal-type"/>SIGNAL TYPE
Each signal specifies a type from the following set (from FrancaIDL):

Name       | Type                       | Min  | Max 
-----------|----------------------------|------|---
UInt8      | unsigned 8-bit integer     |0     | 255
Int8       | signed 8-bit integer       | -128 | 127
UInt16     | unsigned 16-bit integer    |  0   | 65535 
Int16      | signed 16-bit integer      | -32768 | 32767
UInt32     | unsigned 32-bit integer    | 0 | 4294967295
Int32      | signed 32-bit integer      | -2147483648 | 2147483647
UInt64     | unsigned 64-bit integer    | 0    | 2^64
Int64      | signed 64-bit integer      | -2^63 | 2^63 - 1
Boolean    | boolean value              | 0/false | 1/true
Float      | floating point number      | -3.4e -38 | 3.4e 38
Double     | double precision floating point number | -1.7e -300 | 1.7e 300
String     | character string           | n/a  | n/a
ByteBuffer | buffer of bytes (aka BLOB) | n/a | n/a

Please note that the special type ```branch``` denotes a branch, not a
signal. See [branch entry](#branch-entry) chapter for details.


## SIGNAL RANGE [OPTIONAL]
A signal can optionally be specified with a min and max limit,
defining a range that the signal can assume a value within.

## SIGNAL ENUMERATION [OPTIONAL]
A signal can optionally be specified with a set of allowed values that
the signal can be assigned, effectively turning it into an enumerator.  The
values are of the same type as the signal itself.

## <a name="signal-unit-of-measurment"/>SIGNAL UNIT OF MEASUREMENT [OPTIONAL]
A signal can optionally specify a unit type from the following set:

Unit type  | Domain | Description
-----------|--------|-------------
kph        | Speed       | Kilometers per hour
celsius    | Temperature | Degrees celsius
mbar       | Pressure    | millibar
percent    | Percent     | Percent
hz         | frequency   | Frequency
lat        | position    | Decimal latitude 
lon        | position    | Decimal longitude
millimeter | distance    | Millimeter
meter      | distance    | Meter
kilometer  | distance    | Kilometer
[more to come] | ... | ...


## SIGNAL NAMING CONVENTION
Signals are named, left-to-right, from the root of the signal tree
toward the signal itself. Each element in the name is deliniated with
a period (".") .

In Fig 1 above the left mirror heated signal would be:

    body.mirrors.left.heated


If there are an array of elements, such as door 0 - 3, they will be
named with an index branch:

```
body.doors.0.lock
body.doors.0.windows_pos
body.doors.1.lock
body.doors.1.windows_pos
body.doors.2.lock
body.doors.2.windows_pos
body.doors.3.lock
body.doors.3.windows_pos
```


### PARENT NODES
If a signal is defined, all parent branches included in its name must
be included as well, as shown below:

```
[Signal] body.mirrors.left.heated
[Branch] body.mirrors.left
[Branch] body.mirrors
[Branch] body
```

# SIGNAL SPECIFICATION FORMAT
The signal specification is written in JSON format, where each signal
is a self-contained JSON object.

One or more JSON objects are aggregated into a single file, called a
*vspec* file.

A vspec can, in addition to JSON objects, also contain comments and
include directives.

An include directive refers to another vspec file that is to replace
the directive, much like ```#include``` in C/C++.

The schematics below shows an example of a top-level
file ```root.vspec``` that includes two other
files, ```body.vspec``` and ```chassis.vspec```. In
its turn ```chassis.vspec``` includes ```brakes.vspec```


## <a name="branch-entry"/>BRANCH ENTRY
A branch entry describes a tree branch (or node) containing other branches and signals.

A branch entry example is given below

```JSON
{
  "name": "body.door",
  "type": "branch",
  "aggregate": true,
  "description": "Some description"
}
```


The following elements are defined:

* **```name```**<br>
Defines the dot-notaded signal name to the signal. Please note that
all parental branches included in the name must be defined as well.

* **```type```**<br>
The value ```branch``` specifies that this is a branch entry (as
opposed to a signal entry).

* **```aggregate``` [optional]**<br>
Defines if this branch is an aggreaget or not. See
[aggregate](#aggregate) chapter for more information.<br>
Defaults to ```false``` if not defined.

* **```description```**<br>
A description string to be included (when applicable) in the various
specification files generated from this branch entry.

Below is an example a complete specification describing a geospatial position.

```JSON
{
  "name": "nav",
  "type": "branch",
  "description": "Navigitaional top-level branch."
}

{
  "name": "nav.location",
  "type": "branch",
  "aggregate": true,
  "description": "The current location of the vehicle."
}

{
  "name": "nav.location.lat",
  "type": "Float",
  "description": "Latitude."
}

{
  "name": "nav.location.lon",
  "type": "Float",
  "description": "Latitude."
}

{
  "name": "nav.location.alt",
  "type": "Float",
  "description": "Altitude."
}
```

The ```nav.location``` branch's ```aggregate``` member indicaates that
if any of ```lat```, ```lon```, or ```alt``` are are assiged a new
value, all three signals should be distrubuted as a single entity.


## SIGNAL ENTRY
A signal entry defines a single signal and its attributes. A signal
entry example is given below.

```JSON
{
	"name": "chassis.transmission.speed",
	"type": "Uint16",
	"unit": "km/h",
	"min": "1",
	"max": "300",
	"description": "The vehicle speed, as measured by the drivetrain."
}
```

* **```name```**<br>
Defines the dot-notaded signal name to the signal. Please note that
all parental branches included in the name must be defined as well.

* **```type```**<br>
The string value of the type specifies the scalar type of the signal
value. See [signal type](#signal-type) chapter for a list of available types.

* **```min``` [optional]**<br>
The minimum value, within the interval of the given ```type```, that
the signal can be assigned.<br>
If set to ```false```, the minimum value will be the "Min" value for
the given, as specified by the [signal type](#signal-type)
chapter.<br>
Default, if not specified, is ```false```.<br>
Cannot be specified if ```enum``` is specified for the same signal entry.

* **```max``` [optional]**<br>
The max value, within the interval of the given ```type```, that the
signal can be assigned.<br>
If set to ```false```, the maximum value will be the "Max" value for
the given, as specified by the [signal type](#signal-type)
chapter.<br>
Default, if not specified, is ```false```.<br>
Cannot be specified if ```enum``` is specified for the same signal entry.


* **```unit``` [optional]**<br>
The unit of measurement that the signal has.e. See
[signal unit of measurements](#signal-unit-of-measurment) chapter for
a list of available unit types.<br>
Cannot be specified if ```enum``` is specified for the same signal entry.

* **```description```**<br>
A description string to be included (when applicable) in the various
specification files generated from this signal entry.


## ENUMERATED SIGNAL ENTIRES
A signal can optionally be enumerated, allowing it to be assigned a value from a
specified set of values. An example of an enumerated signal is given below.

```JSON
{
	"name": "chassis.transmission.gear",
	"type": "Uint16",
	"enum": [ -1, 1, 2, 3, 4, 5, 6, 7, 8 ],
	"description": "The selected gear. -1 is reverse."
}
```

An enumerated signal entry has no ```min```, ```max```, or ```unit```
element.

The ```enum``` element is an array of values, all of the type specified
by ```type``` element, that the signal can be assigned.


# VSPEC FILE FORMAT
Apart from JSON objects, a vspec file can have two additional
elements, comments and include directives, described below


## COMMENTS.
A comment starts with a ```#``` and extends to the end of the line.
If a ```#``` is encountered as a part of a line, all characters
after ```#``` are ignored.

## INCLUDE DIRECTIVES

# TOOLS

# SPECIFICATION VERSIONING

## BRANCHING
## RELEASES

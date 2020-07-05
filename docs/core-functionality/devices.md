# Device Types

A device type represents a particular make and model of hardware that exists in the real world. Device types define the physical attributes of a device (rack height and depth) and its individual components (console, power, and network interfaces).

Device types are instantiated as devices installed within racks. For example, you might define a device type to represent a Juniper EX4300-48T network switch with 48 Ethernet interfaces. You can then create multiple devices of this type named "switch1," "switch2," and so on. Each device will inherit the components (such as interfaces) of its device type at the time of creation. (However, changes made to a device type will **not** apply to instances of that device type retroactively.)

Some devices house child devices which share physical resources, like space and power, but which functional independently from one another. A common example of this is blade server chassis. Each device type is designated as one of the following:

* A parent device (which has device bays)
* A child device (which must be installed in a device bay)
* Neither

!!! note
    This parent/child relationship is **not** suitable for modeling chassis-based devices, wherein child members share a common control plane.

    For that application you should create a single Device for the chassis, and add Interfaces directly to it.  Interfaces can be created in bulk using range patterns, e.g. "Gi1/[1-24]".

    Add Inventory Items if you want to record the line cards themselves as separate entities.  There is no explicit relationship between each interface and its line card, but it may be implied by the naming (e.g. interfaces "Gi1/x" are on line card 1)

## Manufacturers

Each device type must be assigned to a manufacturer. The model number of a device type must be unique to its manufacturer.

## Component Templates

Each device type is assigned a number of component templates which define the physical components within a device. These are:

* Console ports
* Console server ports
* Power ports
* Power outlets
* Network interfaces
* Front ports
* Rear ports
* Device bays (which house child devices)

Whenever a new device is created, its components are automatically created per the templates assigned to its device type. For example, a Juniper EX4300-48T device type might have the following component templates defined:

* One template for a console port ("Console")
* Two templates for power ports ("PSU0" and "PSU1")
* 48 templates for 1GE interfaces ("ge-0/0/0" through "ge-0/0/47")
* Four templates for 10GE interfaces ("xe-0/2/0" through "xe-0/2/3")

Once component templates have been created, every new device that you create as an instance of this type will automatically be assigned each of the components listed above.

!!! note
    Assignment of components from templates occurs only at the time of device creation. If you modify the templates of a device type, it will not affect devices which have already been created. However, you always have the option of adding, modifying, or deleting components on existing devices.

---

# Devices

Every piece of hardware which is installed within a rack exists in NetBox as a device. Devices are measured in rack units (U) and can be half depth or full depth. A device may have a height of 0U: These devices do not consume vertical rack space and cannot be assigned to a particular rack unit. A common example of a 0U device is a vertically-mounted PDU.

When assigning a multi-U device to a rack, it is considered to be mounted in the lowest-numbered rack unit which it occupies. For example, a 3U device which occupies U8 through U10 is said to be mounted in U8. This logic applies to racks with both ascending and descending unit numbering.

A device is said to be full depth if its installation on one rack face prevents the installation of any other device on the opposite face within the same rack unit(s). This could be either because the device is physically too deep to allow a device behind it, or because the installation of an opposing device would impede airflow.

## Device Components

There are eight types of device components which comprise all of the interconnection logic with NetBox:

* Console ports
* Console server ports
* Power ports
* Power outlets
* Network interfaces
* Front ports
* Rear ports
* Device bays

### Console

Console ports connect only to console server ports. Console connections can be marked as either *planned* or *connected*.

### Power

Power ports connect only to power outlets. Power connections can be marked as either *planned* or *connected*.

### Interfaces

Interfaces connect to one another in a symmetric manner: If interface A connects to interface B, interface B therefore connects to interface A. Each type of connection can be classified as either *planned* or *connected*.

Each interface is a assigned a type denoting its physical properties. Two special types exist: the "virtual" type can be used to designate logical interfaces (such as SVIs), and the "LAG" type can be used to desinate link aggregation groups to which physical interfaces can be assigned.

Each interface can also be enabled or disabled, and optionally designated as management-only (for out-of-band management). Fields are also provided to store an interface's MTU and MAC address.

VLANs can be assigned to each interface as either tagged or untagged. (An interface may have only one untagged VLAN.)

### Pass-through Ports

Pass-through ports are used to model physical terminations which comprise part of a longer path, such as a cable terminated to a patch panel. Each front port maps to a position on a rear port. A 24-port UTP patch panel, for instance, would have 24 front ports and 24 rear ports. Although this relationship is typically one-to-one, a rear port may have multiple front ports mapped to it. This can be useful for modeling instances where multiple paths share a common cable (for example, six different fiber connections sharing a 12-strand MPO cable).

Pass-through ports can also be used to model "bump in the wire" devices, such as a media convertor or passive tap.

### Device Bays

Device bays represent the ability of a device to house child devices. For example, you might install four blade servers into a 2U chassis. The chassis would appear in the rack elevation as a 2U device with four device bays. Each server within it would be defined as a 0U device installed in one of the device bays. Child devices do not appear within rack elevations or the "Non-Racked Devices" list within the rack view.

Child devices are first-class Devices in their own right: that is, fully independent managed entities which don't share any control plane with the parent.  Just like normal devices, child devices have their own platform (OS), role, tags, and interfaces.  You cannot create a LAG between interfaces in different child devices.

Therefore, Device bays are **not** suitable for modeling chassis-based switches and routers.  These should instead be modeled as a single Device, with the line cards as Inventory Items.

## Device Roles

Devices can be organized by functional roles. These roles are fully customizable. For example, you might create roles for core switches, distribution switches, and access switches.

---

# Platforms

A platform defines the type of software running on a device or virtual machine. This can be helpful when it is necessary to distinguish between, for instance, different feature sets. Note that two devices of the same type may be assigned different platforms: for example, one Juniper MX240 running Junos 14 and another running Junos 15.

The platform model is also used to indicate which [NAPALM](https://napalm-automation.net/) driver NetBox should use when connecting to a remote device. The name of the driver along with optional parameters are stored with the platform.

The assignment of platforms to devices is an optional feature, and may be disregarded if not desired.

---

# Inventory Items

Inventory items represent hardware components installed within a device, such as a power supply or CPU or line card. Currently, these are used merely for inventory tracking, although future development might see their functionality expand. Like device types, each item can optionally be assigned a manufacturer.

---

# Virtual Chassis

A virtual chassis represents a set of devices which share a single control plane: a stack of switches which are managed as a single device, for example. Each device in the virtual chassis is assigned a position and (optionally) a priority. Exactly one device is designated the virtual chassis master: This device will typically be assigned a name, secrets, services, and other attributes related to its management.

It's important to recognize the distinction between a virtual chassis and a chassis-based device. For instance, a virtual chassis is not used to model a chassis switch with removable line cards such as the Juniper EX9208, as its line cards are _not_ physically separate devices capable of operating independently.

---

# Cables

A cable represents a physical connection between two termination points, such as between a console port and a patch panel port, or between two network interfaces. Cables can be traced through pass-through ports to form a complete path between two endpoints. In the example below, three individual cables comprise a path between the two connected endpoints.

```
|<------------------------------------------ Cable Path ------------------------------------------->|

  Device A                   Patch Panel A                 Patch Panel B                  Device B
+-----------+               +-------------+               +-------------+               +-----------+
| Interface | --- Cable --- | Front Port  |               | Front Port  | --- Cable --- | Interface |
+-----------+               +-------------+               +-------------+               +-----------+
                            +-------------+               +-------------+
                            |  Rear Port  | --- Cable --- |  Rear Port  |
                            +-------------+               +-------------+
```

All connections between device components in NetBox are represented using cables. However, defining the actual cable plant is optional: Components can be be directly connected using cables with no type or other attributes assigned.

Cables are also used to associated ports and interfaces with circuit terminations. To do this, first create the circuit termination, then navigate the desired component and connect a cable between the two.

As reported in [this issue](https://github.com/AsteroidOS/asteroid/issues/340), the previously working Qt5 solution for GNSS no longer works in Qt6.  To address this, several possible approaches were considered:

## 1. Re-add geoclue v0 and re-create a plugin for QtPositioning
The Qt5 version used geoclue v0.12.99 which is obsolete and unmaintained, and [geoclue-providers-hybris](https://github.com/mer-hybris/geoclue-providers-hybris).  However, as mentioned above, Qt removed support for it in 2021.  The geoclue-providers-hybris supports both time and location injection (tell GNSS approximate current location, if available, to speed satellite acquisition) and A-GPS (assisted GPS uses internet, if available, to speed satellite acquisition).  It's not yet clear how a QtPositioning application would access these capabilities, but they exist.

So using this option would require:

1. re-adding geoclue support (could be done as a "new" plugin for QtPositioning)

### Advantages

1. known and familiar mechanism
2. no adaptations required for existing AsteroidOS software using QtPositioning
3. probably quickest to solution

### Drawbacks

1. continued dependency on geoclue v0
2. it's unknown whether a known, existing problem (GNSS does not stop, draining battery) would still be present
3. continued dependency on geoclue-providers-hybris

## 2. Create interface to use QtPositioning geoclue2 plugin
This was [proposed](https://github.com/mer-hybris/geoclue-providers-hybris/issues/46) in 2023 to the creators of geoclue-providers-hybris but there were two substantive objections:

1. "The problem with geoclue 2 has been that it removed the support for external location providers such as this one. In order to update to a new version we'd need again a mechanism to have location providers that don't live in main geoclue repository."
2. "In addition to the lack of support external plugin mentioned in the previous comment, geoclue 2 also seems a bit limited. It does not support for example satellite info which is supported in the old geoclue but instead seems to only handle getting location."

Using this option would require:

1. rewrite/patch of geoclue2 to include some capability from geoclue-providers-hybris

### Advantages

1. no changes required to QtPositioning

### Drawbacks

1. effort and mechanism of extension of geoclue2 in this way are unknown
2. both HAL and binder interfaces would need to be supported
3. lose the capability to track satellite data
4. no ability to do location injection (tell GNSS approximate current location, if available, to speed satellite acquisition)
5. no ability to use A-GPS (assisted GPS uses internet, if available, to speed satellite acquisition)

## 3. Create interface to use QtPositioning nmea plugin
QtPositioning already ships with an [nmea plugin](https://doc.qt.io/qt-6/position-plugin-nmea.html) which handles raw NMEA sentences.  

### Advantages

1. no changes required to QtPositioning
2. handling and parsing of NMEA sentences, including GPS, Glonass, Beidou handed by existing Qt code
3. avoids "double conversion" to/from D-Bus

### Drawbacks

1. Uncertain mechanism for controlling power on/off of underlying GNSS receiver (which uses a lot of battery energy)
2. no ability to do location injection (tell GNSS approximate current location, if available, to speed satellite acquisition)
3. no ability to use A-GPS (assisted GPS uses internet, if available, to speed satellite acquisition)

## 4. Create new plugin for QtPositioning
This option would create a [new plugin](https://doc.qt.io/qt-6/qtpositioning-plugins.html) for QtPositioning that could provide both location and satellite information.

### Advantages

1. no changes required to QtPositioning
2. avoids "double conversion" to/from D-Bus

### Drawbacks

1. only available to Qt applications, and not to GTK or non-Qt command line applications without adaptation
2. no ability to do location injection (tell GNSS approximate current location, if available, to speed satellite acquisition)
3. no ability to use A-GPS (assisted GPS uses internet, if available, to speed satellite acquisition)

----
# Decision
After discussion and debate, and some code and experimentation, we have a working solution.  No claim is made as to whether it is the optimal solution.  It is based on solution #1 above.  The pieces are:

## qtlocation-geoclue (this code)
This creates a QtLocation plugin, implementing both the location and satellite functions as described in the [plugin docs](https://doc.qt.io/qt-6/qtpositioning-plugins.html) for Qt6.  Although based on the plugin that was [removed](https://github.com/qt/qtpositioning/commit/3e9dc439c4a7fcb9e6773700e76a7ac40d555ae9) in 2021, we are now the sole maintainers of it, and it needed work to restore to full functionality.

## geoclue with a [patch](https://github.com/AsteroidOS/meta-asteroid/pull/287)
The version version 0.12.99 of geoclue, is long obsolete and has not been altered since 2012. Because of this, any fixes are now solely up to us.  One such fix was to address an existing problem (GNSS does not stop, draining battery) mentioned above.

## [geoclue-providers-hybris](https://github.com/mer-hybris/geoclue-providers-hybris)
This is the pair of adapters for the underlying hybris implementation.  Currently Jolla maintains these, both in HAL and binder versions.  It's worth noting that these drivers implement both A-GPS and location injection but these interfaces are not exposed to any of the layers above.

## libhybris
This is the lowest layer in the stack that we provide, which uses binary blob drivers with libhybris to gain access to the hardware and firmware for GNSS.

TODO: add diagram

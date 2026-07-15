cmake_minimum_required(VERSION 3.16)

project(QtPositioning-plugin-geoclue
    VERSION "1.0"
    DESCRIPTION "Geoclue plugin for QtPositioning"
    HOMEPAGE_URL ""
    LANGUAGES CXX C
)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

find_package(Qt6 ${PROJECT_VERSION} CONFIG REQUIRED COMPONENTS Core DBus Positioning)

qt_standard_project_setup()

set(
    dbus_INTERFACES
    org.freedesktop.Geoclue.xml
    org.freedesktop.Geoclue.Master.xml
    org.freedesktop.Geoclue.MasterClient.xml
    org.freedesktop.Geoclue.Position.xml
    org.freedesktop.Geoclue.Satellite.xml
    org.freedesktop.Geoclue.Velocity.xml
)
set_source_files_properties(
    ${dbus_INTERFACES}
    PROPERTIES
    NO_NAMESPACE ON
    INCLUDE geocluetypes.h
)

set(dbus_SRCS)
qt_add_dbus_interfaces(
    dbus_SRCS
    ${dbus_INTERFACES}
)

qt_add_plugin (
    qtposition_geoclue
    CLASS_NAME QGeoPositionInfoSourceFactoryGeoclue
        geocluetypes.cpp geocluetypes.h
        qgeocluemaster.cpp qgeocluemaster.h
        qgeopositioninfosource_geocluemaster.cpp qgeopositioninfosource_geocluemaster.h
        qgeopositioninfosourcefactory_geoclue.cpp qgeopositioninfosourcefactory_geoclue.h
        qgeosatelliteinfosource_geocluemaster.cpp qgeosatelliteinfosource_geocluemaster.h
        ${dbus_SRCS}
    # INCLUDE_DIRECTORIES
    #     ${CMAKE_CURRENT_BINARY_DIR}
    #     core.location.includes
)

target_link_libraries(
    qtposition_geoclue
    Qt::Core
    Qt::DBus
    Qt::Positioning
)

install(
    TARGETS qtposition_geoclue
    DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/plugins/position
)


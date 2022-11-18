cmake_minimum_required(VERSION 3.22)
project(valac)

set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS true)
# set(PREFIX C:/gnome)

include_directories("gee" "vala" "ccode" "codegen" "gobject-introspection" "./")

add_library(GLIB2 SHARED IMPORTED)
set_property(TARGET GLIB2 PROPERTY INTERFACE_INCLUDE_DIRECTORIES
    ${CMAKE_INSTALL_PREFIX}/include/glib-2.0 
    ${CMAKE_INSTALL_PREFIX}/lib/glib-2.0/include)
set_target_properties(GLIB2 PROPERTIES
#  IMPORTED_LOCATION ${PREFIX}/bin/glib-2.0-0.dll
 IMPORTED_IMPLIB ${CMAKE_INSTALL_PREFIX}/lib/glib-2.0.lib
)

add_library(GO SHARED IMPORTED)
set_target_properties(GO PROPERTIES
#  IMPORTED_LOCATION ${PREFIX}/bin/gobject-2.0-0.dll
 IMPORTED_IMPLIB ${CMAKE_INSTALL_PREFIX}/lib/gobject-2.0.lib
)

add_library(GMODULE SHARED IMPORTED)
set_target_properties(GMODULE PROPERTIES
#  IMPORTED_LOCATION ${PREFIX}/bin/gmodule-2.0-0.dll
 IMPORTED_IMPLIB ${CMAKE_INSTALL_PREFIX}/lib/gmodule-2.0.lib
)

file(GLOB GEE_SRC "./gee/*.c")
add_library(gee SHARED ${GEE_SRC})
target_link_libraries(gee PUBLIC GLIB2 GO GMODULE)

file(GLOB VALA_SRC "./vala/*.c")
add_library(vala SHARED ${VALA_SRC})
target_link_libraries(vala gee)

file(GLOB CCODE_SRC "./ccode/*.c")
add_library(ccode SHARED ${CCODE_SRC})
target_link_libraries(ccode gee vala)

file(GLOB CODEGEN_SRC "./codegen/*.c")
add_library(codegen SHARED ${CODEGEN_SRC})
target_link_libraries(codegen gee vala ccode)

file(GLOB COMPILER_SRC "./compiler/*.c")
add_executable(valac ${COMPILER_SRC})
target_link_libraries(valac gee vala ccode codegen)

file(GLOB GOBJECT_INTROSPECTION_SRC "./gobject-introspection/*.c")
add_library(gidl ${GOBJECT_INTROSPECTION_SRC})
target_link_libraries(gidl PUBLIC GLIB2)
# add_executable(gobject-introspect ${GOBJECT_INTROSPECTION_SRC})

set(VAPIGEN_SRC 
    "./vapigen/valavapigen.c"
    "./vapigen/valagidlparser.c")
add_executable(vapigen ${VAPIGEN_SRC})
target_link_libraries(vapigen gee vala gidl)

set(VAPIGEN_SRC 
    "./vapigen/valavapicheck.c"
    "./vapigen/valagidlparser.c")
add_executable(vapicheck ${VAPIGEN_SRC})
target_link_libraries(vapicheck gee vala gidl)

install(TARGETS valac vapicheck vapigen gee vala ccode codegen)

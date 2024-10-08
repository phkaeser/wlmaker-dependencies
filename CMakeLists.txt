# Copyright 2023 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# Default arguments:
# cmake -DCMAKE_INSTALL_PREFIX:PATH=${HOME}/.local -B build

CMAKE_MINIMUM_REQUIRED(VERSION 3.13)
PROJECT(wlmaker VERSION 0.1
  DESCRIPTION "Wayland Maker - Dependencies"
  LANGUAGES C)

# TODO(kaeser@gubbe.ch): Add a target for refreshing all submodules.
# See https://cliutils.gitlab.io/modern-cmake/chapters/projects/submodule.html

# If not found: Try 'pip3 install --user meson'
FIND_PROGRAM(MESON_EXECUTABLE NAMES meson REQUIRED)
FIND_PROGRAM(NINJA_EXECUTABLE NAMES ninja REQUIRED)
FIND_PACKAGE(PkgConfig REQUIRED)

# https://github.com/phkaeser/libbase
# Initialize: git submodule update --init --recursive --merge
# Checkout: git submodule update --checkout --recursive --merge
# Update: git submodule update --remote --merge
# Update: (cd libbase && git pull)
# ADD_SUBDIRECTORY(libbase)

# wlroots -- build and configure from the git submodule.
# Note: This will *NOT* automatically install the library.
INCLUDE(ExternalProject)

ExternalProject_Add(drm
  SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/drm"
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CONFIGURE_COMMAND ${MESON_EXECUTABLE} --prefix=<INSTALL_DIR> <BINARY_DIR> <SOURCE_DIR>
  BUILD_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR>
  INSTALL_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR> install
)

ExternalProject_Add(pixman
  SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/pixman"
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CONFIGURE_COMMAND ${MESON_EXECUTABLE} --prefix=<INSTALL_DIR> <BINARY_DIR> <SOURCE_DIR>
  BUILD_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR>
  INSTALL_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR> install
)

ExternalProject_Add(wayland
  SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/wayland"
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CONFIGURE_COMMAND ${MESON_EXECUTABLE} --prefix=<INSTALL_DIR> <BINARY_DIR> <SOURCE_DIR> -Ddocumentation=false
  BUILD_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR>
  INSTALL_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR> install
)

ExternalProject_Add(wayland_protocols
  SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/wayland-protocols"
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CONFIGURE_COMMAND ${MESON_EXECUTABLE} --prefix=<INSTALL_DIR> <BINARY_DIR> <SOURCE_DIR>
  BUILD_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR>
  INSTALL_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR> install
)

ExternalProject_Add(hwdata
  SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/hwdata"
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CONFIGURE_COMMAND ./configure --prefix=<INSTALL_DIR> --datadir=<INSTALL_DIR>/share --datarootdir=<INSTALLDIR>/share
  BUILD_COMMAND make
  INSTALL_COMMAND make install
  BUILD_IN_SOURCE 1
)

ExternalProject_Add(libdisplay_info
  SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/libdisplay-info"
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CONFIGURE_COMMAND ${MESON_EXECUTABLE} --prefix=<INSTALL_DIR> <BINARY_DIR> <SOURCE_DIR>
  BUILD_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR>
  INSTALL_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR> install
  DEPENDS hwdata
)

# XWayland as optional dependency, configure wlroots accordingly.
PKG_CHECK_MODULES(XWAYLAND xwayland>=22.1.9)
IF(XWAYLAND_FOUND)
  SET(WLROOTS_XWAYLAND "-Dxwayland=enabled")
ELSE(XWAYLAND_FOUND)
  SET(WLROOTS_XWAYLAND  "-Dxwayland=disabled")
ENDIF(XWAYLAND_FOUND)

ExternalProject_Add(wlroots
  SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/wlroots"
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CONFIGURE_COMMAND ${MESON_EXECUTABLE} --prefix=<INSTALL_DIR> <BINARY_DIR> <SOURCE_DIR> -Dexamples=true -Dbackends=drm,libinput,x11 ${WLROOTS_XWAYLAND}
  BUILD_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR>
  INSTALL_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR> install
  DEPENDS drm wayland pixman wayland_protocols hwdata libdisplay_info
)

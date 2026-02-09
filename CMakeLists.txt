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

ExternalProject_Add(libbacktrace
  SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/libbacktrace"
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --datadir=<INSTALL_DIR>/share --datarootdir=<INSTALL_DIR>/share
  BUILD_COMMAND make
  INSTALL_COMMAND make install
)

# Wayland
# Packages needed: xsltproc xmlto
ExternalProject_Add(wayland
  SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/wayland"
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CONFIGURE_COMMAND ${MESON_EXECUTABLE} --prefix=<INSTALL_DIR> <BINARY_DIR> <SOURCE_DIR>
  BUILD_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR>
  INSTALL_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR> install
)

# libdrm - libdrm-2.4.129
ExternalProject_Add(libdrm
  SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/libdrm"
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CONFIGURE_COMMAND ${MESON_EXECUTABLE} --prefix=<INSTALL_DIR> <BINARY_DIR> <SOURCE_DIR>
  BUILD_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR>
  INSTALL_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR> install
)

# xkbcommon - xkbcommon-1.8.0
# Packages needed: libxcb-xkb-dev
ExternalProject_Add(libxkbcommon
  SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/libxkbcommon"
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CONFIGURE_COMMAND ${MESON_EXECUTABLE} --prefix=<INSTALL_DIR> <BINARY_DIR> <SOURCE_DIR> -Denable-bash-completion=false
  BUILD_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR>
  INSTALL_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR> install
)

# wayland-protocols - 1.47
ExternalProject_Add(wayland-protocols
  SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/wayland-protocols"
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CONFIGURE_COMMAND ${MESON_EXECUTABLE} --prefix=<INSTALL_DIR> <BINARY_DIR> <SOURCE_DIR>
  BUILD_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR>
  INSTALL_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR> install
)

# pixman - pixman-0.46.0
ExternalProject_Add(pixman
  SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/pixman"
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CONFIGURE_COMMAND ${MESON_EXECUTABLE} --prefix=<INSTALL_DIR> <BINARY_DIR> <SOURCE_DIR>
  BUILD_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR>
  INSTALL_COMMAND ${NINJA_EXECUTABLE} -C <BINARY_DIR> install
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
)

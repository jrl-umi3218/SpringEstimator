# Copyright 2015-2016 CNRS-UM LIRMM, CNRS-AIST JRL
#
# This file is part of SpringEstimator.
#
# Tasks is free software: you can redistribute it and/or modify
# it under the terms of the GNU Lesser General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# Tasks is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public License
# along with Tasks.  If not, see <http://www.gnu.org/licenses/>.

INCLUDE(../../cmake/python.cmake)

# Look for Python 2.7
SET(Python_ADDITIONAL_VERSIONS 2.7)
FINDPYTHON()

## Define PYTHON_DISTLIB
EXECUTE_PROCESS(
  COMMAND "${PYTHON_EXECUTABLE}" "-c"
  "import sys, os; print os.sep.join(['lib', 'python' + sys.version[:3], 'dist-packages'])"
  OUTPUT_VARIABLE PYTHON_DISTLIB
  ERROR_QUIET)
# Remove final \n of the variable PYTHON_DISTLIB
STRING(REPLACE "\n" "" PYTHON_DISTLIB "${PYTHON_DISTLIB}")

# create the package in build dir for testing purpose
set(LIBRARY_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR}/spring_estimator)
configure_file(__init__.py ${CMAKE_CURRENT_BINARY_DIR}/spring_estimator/__init__.py COPYONLY)

set(OUTPUT_BINDING ${CMAKE_CURRENT_BINARY_DIR}/spring_estimator.cpp)

# generate python binding code
add_custom_command (
  OUTPUT ${OUTPUT_BINDING}
  COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/generate.py ${OUTPUT_BINDING}
  DEPENDS generate.py
)

# build the library
set(SOURCES ${OUTPUT_BINDING})
include_directories(.)
include_directories(../../src)
include_directories(${PYTHON_INCLUDE_DIRS})

add_library(_spring_estimator SHARED ${SOURCES})
PKG_CONFIG_USE_DEPENDENCY(_spring_estimator SpaceVecAlg)
PKG_CONFIG_USE_DEPENDENCY(_spring_estimator RBDyn)
target_link_libraries(_spring_estimator SpringEstimator)
set_target_properties(_spring_estimator PROPERTIES PREFIX "")

# install rules
set(INSTALL_PATH "${PYTHON_DISTLIB}/spring_estimator/")
install(TARGETS _spring_estimator DESTINATION ${INSTALL_PATH})
install(FILES __init__.py DESTINATION ${INSTALL_PATH})

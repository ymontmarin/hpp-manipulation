#
# Copyright (c) 2014, 2020 CNRS
# Authors: Florent Lamiraux, Guilhem Saurel
#
#
# This file is part of hpp-manipulation
# hpp-manipulation is free software: you can redistribute it
# and/or modify it under the terms of the GNU Lesser General Public
# License as published by the Free Software Foundation, either version
# 3 of the License, or (at your option) any later version.
#
# hpp-manipulation is distributed in the hope that it will be
# useful, but WITHOUT ANY WARRANTY; without even the implied warranty
# of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# General Lesser Public License for more details.  You should have
# received a copy of the GNU Lesser General Public License along with
# hpp-manipulation  If not, see
# <http://www.gnu.org/licenses/>.

CMAKE_MINIMUM_REQUIRED(VERSION 3.1)

SET(PROJECT_NAME hpp-manipulation)
SET(PROJECT_DESCRIPTION "Classes for manipulation planning.")

SET(PROJECT_USE_CMAKE_EXPORT TRUE)
SET(CXX_DISABLE_WERROR TRUE)

INCLUDE(cmake/hpp.cmake)
INCLUDE(cmake/boost.cmake)
INCLUDE(cmake/test.cmake)

COMPUTE_PROJECT_ARGS(PROJECT_ARGS LANGUAGES CXX)
PROJECT(${PROJECT_NAME} ${PROJECT_ARGS})

LIST(APPEND PKG_CONFIG_ADDITIONAL_VARIABLES cmake_plugin)

# Ask Doxygen to create a tree view in html documentation
SET(DOXYGEN_TREEVIEW "NO" CACHE STRING "Set to YES to generate a tree view in the html documentation")

SET(HPP_MANIPULATION_HAS_WHOLEBODY_STEP TRUE
  CACHE BOOL "Compile with dependency to hpp-wholebody-step")

ADD_PROJECT_DEPENDENCY(Boost REQUIRED COMPONENTS regex)

ADD_PROJECT_DEPENDENCY("hpp-core" REQUIRED)
IF(HPP_MANIPULATION_HAS_WHOLEBODY_STEP)
  ADD_PROJECT_DEPENDENCY("hpp-wholebody-step" REQUIRED)
ENDIF()
IF(BUILD_TESTING)
  FIND_PACKAGE(Boost REQUIRED COMPONENTS unit_test_framework)
  ADD_PROJECT_DEPENDENCY("example-robot-data" REQUIRED)
ENDIF()

CONFIG_FILES(doc/main.hh
  include/hpp/manipulation/package-config.hh)

SET(${PROJECT_NAME}_HEADERS
  ${CMAKE_BINARY_DIR}/include/hpp/manipulation/package-config.hh
  include/hpp/manipulation/fwd.hh
  include/hpp/manipulation/handle.hh
  include/hpp/manipulation/problem.hh
  include/hpp/manipulation/problem-solver.hh
  include/hpp/manipulation/device.hh
  include/hpp/manipulation/weighed-distance.hh
  include/hpp/manipulation/constraint-set.hh
  include/hpp/manipulation/roadmap.hh
  include/hpp/manipulation/roadmap-node.hh
  include/hpp/manipulation/connected-component.hh
  include/hpp/manipulation/leaf-connected-comp.hh
  include/hpp/manipulation/manipulation-planner.hh
  include/hpp/manipulation/graph-path-validation.hh
  include/hpp/manipulation/graph-optimizer.hh
  include/hpp/manipulation/graph/state.hh
  include/hpp/manipulation/graph/edge.hh
  include/hpp/manipulation/graph/state-selector.hh
  include/hpp/manipulation/graph/guided-state-selector.hh
  include/hpp/manipulation/graph/graph.hh
  include/hpp/manipulation/graph/statistics.hh
  include/hpp/manipulation/graph/graph-component.hh
  include/hpp/manipulation/graph/fwd.hh
  include/hpp/manipulation/graph/dot.hh
  include/hpp/manipulation/graph/helper.hh
  include/hpp/manipulation/graph/validation.hh

  include/hpp/manipulation/path-optimization/small-steps.hh
  include/hpp/manipulation/path-optimization/enforce-transition-semantic.hh
  include/hpp/manipulation/path-optimization/random-shortcut.hh
  include/hpp/manipulation/path-optimization/spline-gradient-based.hh

  include/hpp/manipulation/path-planner/end-effector-trajectory.hh

  include/hpp/manipulation/problem-target/state.hh

  include/hpp/manipulation/steering-method/cross-state-optimization.hh
  include/hpp/manipulation/steering-method/fwd.hh
  include/hpp/manipulation/steering-method/graph.hh
  include/hpp/manipulation/steering-method/end-effector-trajectory.hh
  )

SET(${PROJECT_NAME}_SOURCES
  src/handle.cc
  src/manipulation-planner.cc
  src/problem-solver.cc
  src/roadmap.cc
  src/connected-component.cc
  src/leaf-connected-comp.cc
  src/constraint-set.cc
  src/roadmap-node.cc
  src/device.cc
  src/weighed-distance.cc
  src/problem.cc
  src/graph-path-validation.cc
  src/graph-optimizer.cc

  src/graph/state.cc
  src/graph/edge.cc
  src/graph/graph.cc
  src/graph/graph-component.cc
  src/graph/state-selector.cc
  src/graph/guided-state-selector.cc
  src/graph/statistics.cc
  src/graph/helper.cc

  src/graph/dot.cc
  src/graph/validation.cc

  src/path-optimization/random-shortcut.cc
  src/path-optimization/enforce-transition-semantic.cc

  src/path-planner/end-effector-trajectory.cc

  src/problem-target/state.cc

  src/steering-method/end-effector-trajectory.cc
  src/steering-method/cross-state-optimization.cc
  src/steering-method/graph.cc
  )

IF(HPP_MANIPULATION_HAS_WHOLEBODY_STEP)
  SET(${PROJECT_NAME}_SOURCES
    ${${PROJECT_NAME}_SOURCES}
    src/path-optimization/small-steps.cc
    )
ENDIF(HPP_MANIPULATION_HAS_WHOLEBODY_STEP)

ADD_LIBRARY(${PROJECT_NAME} SHARED ${${PROJECT_NAME}_SOURCES} ${${PROJECT_NAME}_HEADERS})
TARGET_INCLUDE_DIRECTORIES(${PROJECT_NAME} PUBLIC $<INSTALL_INTERFACE:include>)
TARGET_LINK_LIBRARIES(${PROJECT_NAME} hpp-core::hpp-core Boost::regex)
IF(HPP_MANIPULATION_HAS_WHOLEBODY_STEP)
  TARGET_LINK_LIBRARIES(${PROJECT_NAME} hpp-wholebody-step::hpp-wholebody-step)
ENDIF(HPP_MANIPULATION_HAS_WHOLEBODY_STEP)

INSTALL(TARGETS ${PROJECT_NAME} EXPORT ${TARGETS_EXPORT_NAME} DESTINATION lib)

# GPL licenced part

SET(${PROJECT_NAME}_SOURCES_GPL
  src/path-optimization/spline-gradient-based.cc
  )

ADD_LIBRARY(${PROJECT_NAME}-gpl SHARED ${${PROJECT_NAME}_SOURCES_GPL})
TARGET_LINK_LIBRARIES(${PROJECT_NAME}-gpl ${PROJECT_NAME} hpp-core::hpp-core-gpl)

INSTALL(TARGETS ${PROJECT_NAME}-gpl EXPORT ${TARGETS_EXPORT_NAME} DESTINATION lib)

ADD_SUBDIRECTORY(plugins)
IF(BUILD_TESTING)
  ADD_SUBDIRECTORY(tests)
ENDIF()

PKG_CONFIG_APPEND_LIBS(${PROJECT_NAME})

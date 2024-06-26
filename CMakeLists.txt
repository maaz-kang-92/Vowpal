if(NOT DEFINED PY_VERSION)
  set(PY_VERSION 2.7)
  message(WARNING "No PY_VERSION specified, Python ${PY_VERSION} will be used for VowpalWabbit Python bindings")
else()
  message(STATUS "Python ${PY_VERSION} will be used for VowpalWabbit Python bindings")
endif()
string(REPLACE . "" PY_VERSION_STRIPPED ${PY_VERSION})

if(STATIC_LINK_VW_JAVA)
  set(Boost_USE_STATIC_LIBS ON)
endif()
set(Boost_USE_MULTITHREADED ON)
set(Boost_USE_STATIC_RUNTIME OFF)
find_package(Boost REQUIRED)

# In boost version 1.67 Python version suffixes were instroduced for major and minor versions.
if(Boost_MINOR_VERSION GREATER 66)
  set(BOOST_PY_VERSION_SUFFIX ${PY_VERSION_STRIPPED})
else(Boost_MINOR_VERSION GREATER 66)
  string(SUBSTRING ${PY_VERSION_STRIPPED} 0 1 PY_VERSION_STRIPPED_FIRST_CHAR)
  if(PY_VERSION_STRIPPED_FIRST_CHAR STREQUAL "2")
    set(BOOST_PY_VERSION_SUFFIX "")
  elseif(PY_VERSION_STRIPPED_FIRST_CHAR STREQUAL "3")
    set(BOOST_PY_VERSION_SUFFIX "3")
  endif(PY_VERSION_STRIPPED_FIRST_CHAR STREQUAL "2")
endif(Boost_MINOR_VERSION GREATER 66)

find_package(Boost REQUIRED COMPONENTS python${BOOST_PY_VERSION_SUFFIX})

# Using find_package(Python...) allows us to only require the Python headers to be found.
if(${CMAKE_VERSION} VERSION_LESS "3.12.0")
  find_package(PythonInterp ${PY_VERSION} REQUIRED)
  find_package(PythonLibs ${PY_VERSION} REQUIRED)
elseif(${CMAKE_VERSION} VERSION_LESS "3.18.0")
  # Prior to 3.18 only the Development component is exposed
  find_package(Python ${PY_VERSION} EXACT COMPONENTS Development REQUIRED)
else()
  # This was fixed in 3.18 to allow for just the Development.Module component to be used
  find_package(Python ${PY_VERSION} EXACT COMPONENTS Development.Module REQUIRED)
endif()

if (NOT WIN32)
  # Use position independent code for all targets in this directory
  set(CMAKE_POSITION_INDEPENDENT_CODE ON)
endif()

if (NOT ${CMAKE_SYSTEM_NAME} MATCHES "Darwin" AND NOT WIN32)
  set(EXPORT_DYNAMIC -Wl,--export-dynamic)
endif()

# Note: Python bindings previously built under c++0x

# create the lib
add_library(pylibvw SHARED pylibvw.cc)
set_target_properties(pylibvw PROPERTIES PREFIX "")

# The python setup.py script should communicate what suffix it wants.
if(VW_PYTHON_SHARED_LIB_SUFFIX)
  set_target_properties(pylibvw PROPERTIES SUFFIX ${VW_PYTHON_SHARED_LIB_SUFFIX})
else()
  # fallback to previous suffix selection logic
  if (NOT WIN32)
    # OSX builds a .dylib and then attempts to find a .so. Just build a .so file and things work
    set_target_properties(pylibvw PROPERTIES SUFFIX ".so")
  else()
    # Force Windows to build with a .pyd extension
    set_target_properties(pylibvw PROPERTIES SUFFIX ".pyd")
  endif()
endif()

if (NOT WIN32)
  # Windows links dynamically
  target_compile_definitions(pylibvw PUBLIC BOOST_PYTHON_STATIC_LIB)
endif()

if(${CMAKE_VERSION} VERSION_LESS "3.12.0")
  target_link_libraries(pylibvw PUBLIC ${PYTHON_LIBRARIES} Boost::python${BOOST_PY_VERSION_SUFFIX})
  target_include_directories(pylibvw PRIVATE ${PYTHON_INCLUDE_PATH})
elseif(${CMAKE_VERSION} VERSION_LESS "3.15.7")
  target_link_libraries(pylibvw PUBLIC Python::Python Boost::boost Boost::python${BOOST_PY_VERSION_SUFFIX})
else()
  target_link_libraries(pylibvw PUBLIC Python::Module Boost::boost Boost::python${BOOST_PY_VERSION_SUFFIX})
endif()
target_link_libraries(pylibvw PUBLIC vw_core)
target_compile_options(pylibvw PRIVATE ${EXPORT_DYNAMIC})

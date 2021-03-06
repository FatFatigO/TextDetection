CMAKE_MINIMUM_REQUIRED( VERSION 2.8 )

SET( DEFAULT_BUILD_TYPE Release )
SET( DEFAULT_CXX_FLAGS_DEBUG " -Wall -std=c++0x -fPIC" )
SET( DEFAULT_CXX_FLAGS_RELEASE " -Wall -O3 -std=c++0x -fPIC" )
SET( BUILD_TYPE_DOC "Choose the type of build, options are: None(CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Debug Release RelWithDebInfo MinSizeRel." )
SET( CXX_FLAGS_DEBUG_DOC "Flags used by the compiler during debug builds (/MD /Ob1 /Oi /Ot /Oy /Gs will produce slightly smaller files" )
SET( CXX_FLAGS_RELEASE_DOC "Flags used by the compiler during release builds (/MD /Ob1 /Oi /Ot /Oy /Gs will produce slightly smaller files" )

IF( DEFINED CMAKE_BUILD_TYPE )
   SET( CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE} CACHE STRING ${BUILD_TYPE_DOC} )
ELSE()
   SET( CMAKE_BUILD_TYPE ${DEFAULT_BUILD_TYPE} CACHE STRING ${BUILD_TYPE_DOC} )
ENDIF()

IF( DEFINED CMAKE_CXX_FLAGS_DEBUG )
   SET( CMAKE_CXX_FLAGS_DEBUG ${CMAKE_CXX_FLAGS_DEBUG} CACHE STRING ${CXX_FLAGS_DEBUG_DOC} )
ELSE()
   SET( CMAKE_CXX_FLAGS_DEBUG ${DEFAULT_CXX_FLAGS_DEBUG} CACHE STRING ${CXX_FLAGS_DEBUG_DOC} )
ENDIF()

IF( DEFINED CMAKE_CXX_FLAGS_RELEASE )
   SET( CMAKE_CXX_FLAGS_RELEASE ${CMAKE_CXX_FLAGS_RELEASE} CACHE STRING ${CXX_FLAGS_RELEASE_DOC} )
ELSE()
   SET( CMAKE_CXX_FLAGS_RELEASE ${DEFAULT_CXX_FLAGS_RELEASE} CACHE STRING ${CXX_FLAGS_RELEASE_DOC} )
ENDIF()


PROJECT( TextDetection )

LIST(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

# External dependencies
INCLUDE( FindPkgConfig )

FIND_PACKAGE( Boost REQUIRED )
FIND_PACKAGE( Eigen3 REQUIRED )
pkg_check_modules( OPENCV REQUIRED opencv-2.3.1 )

FIND_PACKAGE( SWIG REQUIRED )
INCLUDE( ${SWIG_USE_FILE} )

FIND_PACKAGE( PythonInterp )
FIND_PACKAGE( PythonLibs )

EXECUTE_PROCESS(COMMAND ${PYTHON_EXECUTABLE} --version
              ERROR_VARIABLE PYTHON_VERSION_FULL
              OUTPUT_STRIP_TRAILING_WHITESPACE)

STRING(REGEX MATCH "[0-9].[0-9]" PYTHON_VERSION_MAJOR_MINOR "${PYTHON_VERSION_FULL}")
IF(UNIX)
    SET(PYTHON_PLUGIN_INSTALL_PATH lib/python${PYTHON_VERSION_MAJOR_MINOR}/site-packages/TextDetection )
    SET(PYTHON_PACKAGES_PATH lib/python${PYTHON_VERSION_MAJOR_MINOR}/site-packages)
ENDIF()
IF(WIN32)
    GET_FILENAME_COMPONENT(PYTHON_PATH "[HKEY_LOCAL_MACHINE\\SOFTWARE\\Python\\PythonCore\\${PYTHON_VERSION_MAJOR_MINOR}\\InstallPath]" ABSOLUTE CACHE)
    SET(PYTHON_PLUGIN_INSTALL_PATH "${PYTHON_PATH}/Lib/site-packages/TextDetection")
    SET(PYTHON_PACKAGES_PATH "${PYTHON_PATH}/Lib/site-packages")
ENDIF()

INCLUDE_DIRECTORIES(${PYTHON_INCLUDE_PATH})

# Main library
INCLUDE_DIRECTORIES(
    .
    ${Boost_INCLUDE_DIRS}
    ${EIGEN3_INCLUDE_DIRS}
    ${OPENCV_INCLUDE_DIRS}
    ${PROJECT_SOURCE_DIR}/include )

#LINK_DIRECTORIES()

ADD_LIBRARY( TextDetectionStatic 
    src/TextAreaFinder.cpp
     )

SET_TARGET_PROPERTIES( TextDetectionStatic
    PROPERTIES OUTPUT_NAME TextDetection )
TARGET_LINK_LIBRARIES( TextDetectionStatic ) 

INSTALL( TARGETS TextDetectionStatic
        ARCHIVE DESTINATION lib )
INSTALL( DIRECTORY include/TextDetection DESTINATION include
        FILES_MATCHING PATTERN "*.h" )

# Python interface using SWIG
SET(CMAKE_SWIG_FLAGS "")

ADD_SUBDIRECTORY(interfaces/python)
SET_SOURCE_FILES_PROPERTIES(interfaces/python/TextDetection.i PROPERTIES CPLUSPLUS ON)
SWIG_ADD_MODULE( TextDetection python interfaces/python/TextDetection.i )
SWIG_LINK_LIBRARIES( TextDetection ${PYTHON_LIBRARIES} TextDetectionStatic )
set_source_files_properties(${swig_generated_file_fullname} PROPERTIES 
                  COMPILE_FLAGS -fno-strict-aliasing)

# Example programs
ADD_SUBDIRECTORY( examples/MaximallyStableRegions )

#SET(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE) 
#SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

INSTALL( FILES ${CMAKE_BINARY_DIR}/${outdir}/_TextDetection.so
          DESTINATION ${PYTHON_PACKAGES_PATH} )
INSTALL( FILES ${CMAKE_BINARY_DIR}/${outdir}/TextDetection.py
          DESTINATION ${PYTHON_PACKAGES_PATH} )



cmake_minimum_required(VERSION 3.5)

project(SlicerHeart)

#-----------------------------------------------------------------------------
# Extension meta-information
set(EXTENSION_HOMEPAGE "http://www.slicer.org/slicerWiki/index.php/Documentation/Nightly/Extensions/SlicerHeart")
set(EXTENSION_CATEGORY "Cardiac")
set(EXTENSION_CONTRIBUTORS "Andras Lasso (PerkLab, Queen's University), Steve Pieper (Isomics), Adam Rankin (Robarts), Csaba Pinter (Pixel Medical / Ebatinca), Christian Herz (CHOP), Matthew Jolley (CHOP/UPenn)")
set(EXTENSION_DESCRIPTION "Modules for cardiac analysis and intervention planning and guidance")
set(EXTENSION_ICONURL "http://www.slicer.org/slicerWiki/images/f/ff/SlicerHeart_Logo_128x128.png")
set(EXTENSION_SCREENSHOTURLS "http://www.slicer.org/slicerWiki/images/a/ae/BasicHeartView.png")
set(EXTENSION_STATUS Beta)
set(EXTENSION_DEPENDS SlicerIGT)
set(EXTENSION_BUILD_SUBDIRECTORY inner-build)

set(SUPERBUILD_TOPLEVEL_PROJECT inner)

#-----------------------------------------------------------------------------
# Extension dependencies
find_package(Slicer REQUIRED)
include(${Slicer_USE_FILE})
mark_as_superbuild(Slicer_DIR)

find_package(Git REQUIRED)
mark_as_superbuild(GIT_EXECUTABLE)

# Only build ITK filters if ITK5 is available
# Disable by default, as it is currently not used and may break the build
option(SlicerHeart_BUILD_ITK_FILTERS "Build ITK noise filters." OFF)
message(STATUS "ITK version: ${ITK_VERSION_MAJOR}.${ITK_VERSION_MINOR}.${ITK_VERSION_PATCH}.")
if(${ITK_VERSION_MAJOR}.${ITK_VERSION_MINOR} VERSION_LESS 5.0)
  message(STATUS "ITK version: ${ITK_VERSION_MAJOR}.${ITK_VERSION_MINOR}.${ITK_VERSION_PATCH}. ITK filters are not built.")
  set(SlicerHeart_BUILD_ITK_FILTERS OFF)
endif()

#-----------------------------------------------------------------------------
# SuperBuild setup
option(${EXTENSION_NAME}_SUPERBUILD "Build ${EXTENSION_NAME} and the projects it depends on." ON)
mark_as_advanced(${EXTENSION_NAME}_SUPERBUILD)
if(${EXTENSION_NAME}_SUPERBUILD)
  include("${CMAKE_CURRENT_SOURCE_DIR}/SuperBuild.cmake")
  return()
endif()

# We do not have compile-time dependency on SlicerIGT therefore
# there is no need for find_package. Uncomment the following lines
# if compile-time dependency is added.
# find_package(SlicerIGT REQUIRED)

#-----------------------------------------------------------------------------
# Extension modules
add_subdirectory(ValveView)
add_subdirectory(Philips4dUsDicomPatcher)
# add_subdirectory(Philips4dUsReader) # Disabled until fixing the gdcm build errors
add_subdirectory(KretzFileReader)
add_subdirectory(GeUsMovieReader)
add_subdirectory(DicomUltrasoundPlugin)
add_subdirectory(CardiacDeviceSimulator)
add_subdirectory(AsdVsdDeviceSimulator)
add_subdirectory(TCAVValveSimulator)
add_subdirectory(ValveAnnulusAnalysis)
add_subdirectory(ValveClipDeviceSimulator)
add_subdirectory(ValveQuantification)
add_subdirectory(TomTecUcdPlugin)
add_subdirectory(CartoExport)
add_subdirectory(BafflePlanner)
if (SlicerHeart_BUILD_ITK_FILTERS)
  add_subdirectory(PhaseSymmetryFilter)
endif()
add_subdirectory(ConformalTextureMapping)
if (WIN32)
  # Image3dAPI only works on Windows
  # (see https://github.com/MedicalUltrasound/Image3dAPI)
  add_subdirectory(UltrasoundImage3dReader)
endif()
add_subdirectory(Reconstruct4DCineMRI)
## NEXT_MODULE

#-----------------------------------------------------------------------------
set(EXTENSION_CPACK_INSTALL_CMAKE_PROJECTS)
if (SlicerHeart_BUILD_ITK_FILTERS)
  list(APPEND EXTENSION_CPACK_INSTALL_CMAKE_PROJECTS "${ITKStrain_DIR};Strain;RuntimeLibraries;/")
  list(APPEND EXTENSION_CPACK_INSTALL_CMAKE_PROJECTS "${ITKPhaseSymmetry_DIR};PhaseSymmetry;RuntimeLibraries;/")
  # Note: no need to install lscm (executable will be installed by the ConformalTextureMapping module)
endif()
set(${EXTENSION_NAME}_CPACK_INSTALL_CMAKE_PROJECTS "${EXTENSION_CPACK_INSTALL_CMAKE_PROJECTS}" CACHE STRING "List of external projects to install" FORCE)
#-----------------------------------------------------------------------------
list(APPEND CPACK_INSTALL_CMAKE_PROJECTS "${CMAKE_BINARY_DIR};${EXTENSION_NAME};ALL;/")
list(APPEND CPACK_INSTALL_CMAKE_PROJECTS "${${EXTENSION_NAME}_CPACK_INSTALL_CMAKE_PROJECTS}")
include(${Slicer_EXTENSION_GENERATE_CONFIG})
include(${Slicer_EXTENSION_CPACK})

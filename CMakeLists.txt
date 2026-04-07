cmake_minimum_required(VERSION 3.23)

# Set default for CMAKE_BUILD_TYPE on first configure
if (NOT EXISTS ${CMAKE_BINARY_DIR}/CMakeCache.txt)
  if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "" FORCE)
  endif()
endif()

project(ParMETIS 
  VERSION 4.0.3
  DESCRIPTION "Parallel mesh partitioning library"
  LANGUAGES C)

# User options
set(INDEXTYPEWIDTH 64 CACHE STRING "Bit-width of index types")
set(REALTYPEWIDTH 64 CACHE STRING "Bit-width of floating-point types")

set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)
set(CMAKE_C_STANDARD 99)

include(CheckIncludeFiles)
check_include_files(regex.h HAVE_REGEX_H)

##############
# Build GKlib
##############

set(GKLIB_SOURCES
  GKlib/src/b64.c
  GKlib/src/blas.c
  GKlib/src/cache.c
  GKlib/src/csr.c
  GKlib/src/error.c
  GKlib/src/evaluate.c
  GKlib/src/fkvkselect.c
  GKlib/src/fs.c
  GKlib/src/getopt.c
  GKlib/src/gkregex.c
  GKlib/src/gk_util.c
  GKlib/src/graph.c
  GKlib/src/htable.c
  GKlib/src/io.c
  GKlib/src/itemsets.c
  GKlib/src/mcore.c
  GKlib/src/memory.c
  GKlib/src/pqueue.c
  GKlib/src/random.c
  GKlib/src/rw.c
  GKlib/src/seq.c
  GKlib/src/sort.c
  GKlib/src/string.c
  GKlib/src/timers.c
  GKlib/src/tokenizer.c
)

if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
  list(APPEND GKLIB_SOURCES GKlib/src/win32/adapt.c)
endif()

add_library(gklib STATIC)
target_sources(gklib PRIVATE ${GKLIB_SOURCES})
target_sources(gklib PUBLIC
  FILE_SET gklib_headers
  TYPE HEADERS
  BASE_DIRS GKlib/include
  FILES
    GKlib/include/gk_arch.h
    GKlib/include/gk_defs.h
    GKlib/include/gk_externs.h
    GKlib/include/gk_getopt.h
    GKlib/include/GKlib.h
    GKlib/include/gk_macros.h
    GKlib/include/gk_mkblas.h
    GKlib/include/gk_mkmemory.h
    GKlib/include/gk_mkpqueue2.h
    GKlib/include/gk_mkpqueue.h
    GKlib/include/gk_mkrandom.h
    GKlib/include/gk_mksort.h
    GKlib/include/gk_mkutils.h
    GKlib/include/gk_ms_inttypes.h
    GKlib/include/gk_ms_stat.h
    GKlib/include/gk_ms_stdint.h
    GKlib/include/gk_proto.h
    GKlib/include/gkregex.h
    GKlib/include/gk_struct.h
    GKlib/include/gk_types.h
)

if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
  target_sources(gklib PUBLIC FILE_SET gklib_headers TYPE HEADERS BASE_DIRS GKlib/include FILES GKlib/include/win32/adapt.h)
endif()

target_compile_definitions(gklib PUBLIC IDXTYPEWIDTH=${INDEXTYPEWIDTH})
target_compile_definitions(gklib PUBLIC REALTYPEWIDTH=${REALTYPEWIDTH})

if(NOT HAVE_REGEX_H)
  target_compile_definitions(gklib PUBLIC USE_GKREGEX)
endif()

##############
# Build METIS
##############


set(METIS_SOURCES
  METIS/libmetis/auxapi.c
  METIS/libmetis/balance.c
  METIS/libmetis/bucketsort.c
  METIS/libmetis/checkgraph.c
  METIS/libmetis/coarsen.c
  METIS/libmetis/compress.c
  METIS/libmetis/contig.c
  METIS/libmetis/debug.c
  METIS/libmetis/fm.c
  METIS/libmetis/fortran.c
  METIS/libmetis/frename.c
  METIS/libmetis/gklib.c
  METIS/libmetis/graph.c
  METIS/libmetis/initpart.c
  METIS/libmetis/kmetis.c
  METIS/libmetis/kwayfm.c
  METIS/libmetis/kwayrefine.c
  METIS/libmetis/mcutil.c
  METIS/libmetis/mesh.c
  METIS/libmetis/meshpart.c
  METIS/libmetis/minconn.c
  METIS/libmetis/mincover.c
  METIS/libmetis/mmd.c
  METIS/libmetis/ometis.c
  METIS/libmetis/options.c
  METIS/libmetis/parmetis.c
  METIS/libmetis/pmetis.c
  METIS/libmetis/refine.c
  METIS/libmetis/separator.c
  METIS/libmetis/sfm.c
  METIS/libmetis/srefine.c
  METIS/libmetis/stat.c
  METIS/libmetis/timing.c
  METIS/libmetis/util.c
  METIS/libmetis/wspace.c
)

add_library(metis STATIC)
target_sources(metis PRIVATE ${METIS_SOURCES})
target_sources(metis PRIVATE
  FILE_SET metis_private_headers
  TYPE HEADERS
  BASE_DIRS METIS/libmetis
  FILES
    METIS/libmetis/defs.h
    METIS/libmetis/gklib_defs.h
    METIS/libmetis/gklib_rename.h
    METIS/libmetis/macros.h
    METIS/libmetis/metislib.h
    METIS/libmetis/proto.h
    METIS/libmetis/rename.h
    METIS/libmetis/stdheaders.h
    METIS/libmetis/struct.h
)

target_sources(metis PUBLIC
  FILE_SET metis_public_headers
  TYPE HEADERS
  BASE_DIRS METIS/include
  FILES
    METIS/include/metis.h
)
target_link_libraries(metis PUBLIC gklib)

target_compile_definitions(metis PUBLIC IDXTYPEWIDTH=${INDEXTYPEWIDTH})
target_compile_definitions(metis PUBLIC REALTYPEWIDTH=${REALTYPEWIDTH})

#################
# Build parmetis
#################

set(PARMETIS_SOURCES
  ParMETIS/libparmetis/akwayfm.c
  ParMETIS/libparmetis/ametis.c
  ParMETIS/libparmetis/balancemylink.c
  ParMETIS/libparmetis/comm.c
  ParMETIS/libparmetis/csrmatch.c
  ParMETIS/libparmetis/ctrl.c
  ParMETIS/libparmetis/debug.c
  ParMETIS/libparmetis/diffutil.c
  ParMETIS/libparmetis/frename.c
  ParMETIS/libparmetis/gklib.c
  ParMETIS/libparmetis/gkmetis.c
  ParMETIS/libparmetis/gkmpi.c
  ParMETIS/libparmetis/graph.c
  ParMETIS/libparmetis/initbalance.c
  ParMETIS/libparmetis/initmsection.c
  ParMETIS/libparmetis/initpart.c
  ParMETIS/libparmetis/kmetis.c
  ParMETIS/libparmetis/kwayrefine.c
  ParMETIS/libparmetis/match.c
  ParMETIS/libparmetis/mdiffusion.c
  ParMETIS/libparmetis/mesh.c
  ParMETIS/libparmetis/mmetis.c
  ParMETIS/libparmetis/move.c
  ParMETIS/libparmetis/msetup.c
  ParMETIS/libparmetis/node_refine.c
  ParMETIS/libparmetis/ometis.c
  ParMETIS/libparmetis/pspases.c
  ParMETIS/libparmetis/redomylink.c
  ParMETIS/libparmetis/remap.c
  ParMETIS/libparmetis/renumber.c
  ParMETIS/libparmetis/rmetis.c
  ParMETIS/libparmetis/selectq.c
  ParMETIS/libparmetis/serial.c
  ParMETIS/libparmetis/stat.c
  ParMETIS/libparmetis/timer.c
  ParMETIS/libparmetis/util.c
  ParMETIS/libparmetis/wave.c
  ParMETIS/libparmetis/weird.c
  ParMETIS/libparmetis/wspace.c
  ParMETIS/libparmetis/xyzpart.c
)

add_library(parmetis STATIC)
target_sources(parmetis PRIVATE ${PARMETIS_SOURCES})
target_sources(parmetis PRIVATE
  FILE_SET parmetis_private_headers
  TYPE HEADERS
  BASE_DIRS ParMETIS/libparmetis
  FILES
    ParMETIS/libparmetis/defs.h
    ParMETIS/libparmetis/gklib_defs.h
    ParMETIS/libparmetis/gklib_rename.h
    ParMETIS/libparmetis/macros.h
    ParMETIS/libparmetis/parmetislib.h
    ParMETIS/libparmetis/proto.h
    ParMETIS/libparmetis/rename.h
    ParMETIS/libparmetis/struct.h
)
target_sources(parmetis PUBLIC
  FILE_SET parmetis_public_headers
  TYPE HEADERS
  BASE_DIRS ParMETIS/include
  FILES
    ParMETIS/include/parmetis.h
)

target_compile_definitions(parmetis PUBLIC IDXTYPEWIDTH=${INDEXTYPEWIDTH})
target_compile_definitions(parmetis PUBLIC REALTYPEWIDTH=${REALTYPEWIDTH})

# Compile parmetis with 64 bit indices and floating point values
target_link_libraries(parmetis PUBLIC gklib metis)

find_package(MPI REQUIRED)
target_link_libraries(parmetis PRIVATE MPI::MPI_C)

# Setup alias for usage with FetchContent
add_library(ParMETIS::parmetis ALIAS parmetis)
add_library(ParMETIS::metis ALIAS metis)
add_library(ParMETIS::gklib ALIAS gklib)

############
# Packaging
############

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# Create package version file
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/parmetis-config-version.cmake
  COMPATIBILITY AnyNewerVersion
)

# Install package version file and package config file
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/parmetis-config-version.cmake
  ${CMAKE_CURRENT_SOURCE_DIR}/parmetis-config.cmake
  DESTINATION ${CMAKE_INSTALL_DATADIR}/parmetis
)

# Add target to export
install(
  TARGETS parmetis metis gklib
  EXPORT parmetisTargets
  FILE_SET parmetis_public_headers
  FILE_SET metis_public_headers
  FILE_SET gklib_headers
)

# Generate parmetisTargets.cmake
install(
  EXPORT parmetisTargets
  NAMESPACE ParMETIS::
  DESTINATION ${CMAKE_INSTALL_DATADIR}/parmetis
)



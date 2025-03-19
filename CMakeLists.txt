cmake_minimum_required(VERSION 3.28)

project(ParMETIS 
  VERSION 4.0.3
  DESCRIPTION "Parallel mesh partitioning library"
  LANGUAGES C)

##############
# Build GKlib
##############

set(GKLIB_SOURCES
  GKlib/b64.c
  GKlib/blas.c
  GKlib/cache.c
  GKlib/csr.c
  GKlib/error.c
  GKlib/evaluate.c
  GKlib/fkvkselect.c
  GKlib/fs.c
  GKlib/getopt.c
  GKlib/gkregex.c
  GKlib/gk_util.c
  GKlib/graph.c
  GKlib/htable.c
  GKlib/io.c
  GKlib/itemsets.c
  GKlib/mcore.c
  GKlib/memory.c
  GKlib/pqueue.c
  GKlib/random.c
  GKlib/rw.c
  GKlib/seq.c
  GKlib/sort.c
  GKlib/string.c
  GKlib/timers.c
  GKlib/tokenizer.c
)
add_library(gklib STATIC)
target_sources(gklib PRIVATE ${GKLIB_SOURCES})
target_sources(gklib PUBLIC
  FILE_SET gklib_headers
  TYPE HEADERS
  BASE_DIRS GKlib
  FILES
    GKlib/gk_arch.h
    GKlib/gk_defs.h
    GKlib/gk_externs.h
    GKlib/gk_getopt.h
    GKlib/GKlib.h
    GKlib/gk_macros.h
    GKlib/gk_mkblas.h
    GKlib/gk_mkmemory.h
    GKlib/gk_mkpqueue2.h
    GKlib/gk_mkpqueue.h
    GKlib/gk_mkrandom.h
    GKlib/gk_mksort.h
    GKlib/gk_mkutils.h
    GKlib/gk_ms_inttypes.h
    GKlib/gk_ms_stat.h
    GKlib/gk_ms_stdint.h
    GKlib/gk_proto.h
    GKlib/gkregex.h
    GKlib/gk_struct.h
    GKlib/gk_types.h
)

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
# Compile metis with 64 bit indices and floating point values
target_compile_definitions(metis PUBLIC IDXTYPEWIDTH=64 REALTYPEWIDTH=64)
target_link_libraries(metis PUBLIC gklib)

#################
# Build parmetis
#################

set(PARMETIS_SOURCES
  libparmetis/akwayfm.c
  libparmetis/ametis.c
  libparmetis/balancemylink.c
  libparmetis/comm.c
  libparmetis/csrmatch.c
  libparmetis/ctrl.c
  libparmetis/debug.c
  libparmetis/diffutil.c
  libparmetis/frename.c
  libparmetis/gklib.c
  libparmetis/gkmetis.c
  libparmetis/gkmpi.c
  libparmetis/graph.c
  libparmetis/initbalance.c
  libparmetis/initmsection.c
  libparmetis/initpart.c
  libparmetis/kmetis.c
  libparmetis/kwayrefine.c
  libparmetis/match.c
  libparmetis/mdiffusion.c
  libparmetis/mesh.c
  libparmetis/mmetis.c
  libparmetis/move.c
  libparmetis/msetup.c
  libparmetis/node_refine.c
  libparmetis/ometis.c
  libparmetis/pspases.c
  libparmetis/redomylink.c
  libparmetis/remap.c
  libparmetis/renumber.c
  libparmetis/rmetis.c
  libparmetis/selectq.c
  libparmetis/serial.c
  libparmetis/stat.c
  libparmetis/timer.c
  libparmetis/util.c
  libparmetis/wave.c
  libparmetis/weird.c
  libparmetis/wspace.c
  libparmetis/xyzpart.c
)

add_library(ParMETIS STATIC)
target_sources(ParMETIS PRIVATE ${PARMETIS_SOURCES})
target_sources(ParMETIS PRIVATE
  FILE_SET parmetis_private_headers
  TYPE HEADERS
  BASE_DIRS libparmetis
  FILES
    libparmetis/defs.h
    libparmetis/gklib_defs.h
    libparmetis/gklib_rename.h
    libparmetis/macros.h
    libparmetis/parmetislib.h
    libparmetis/proto.h
    libparmetis/rename.h
    libparmetis/struct.h
)
target_sources(ParMETIS PUBLIC
  FILE_SET parmetis_public_headers
  TYPE HEADERS
  BASE_DIRS include
  FILES
    include/parmetis.h
)
# Compile parmetis with 64 bit indices and floating point values
target_compile_definitions(ParMETIS PUBLIC IDXTYPEWIDTH=64 REALTYPEWIDTH=64)
target_link_libraries(ParMETIS PUBLIC gklib metis)

find_package(MPI REQUIRED)
target_link_libraries(ParMETIS PRIVATE MPI::MPI_C)

# Setup alias for usage with FetchContent
add_library(ParMETIS::ParMETIS ALIAS ParMETIS)

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
  TARGETS ParMETIS metis gklib
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



cmake_minimum_required(VERSION 3.18)

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
add_library(gklib STATIC ${GKLIB_SOURCES})
target_include_directories(gklib PUBLIC GKlib/)

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

add_library(metis STATIC ${METIS_SOURCES})
target_include_directories(metis PRIVATE METIS/libmetis)
target_include_directories(metis PUBLIC METIS/include)
# Compile metis with 64 bit indices and floating point values
target_compile_definitions(metis PRIVATE IDXTYPEWIDTH=64 REALTYPEWIDTH=64)
target_link_libraries(metis PRIVATE gklib)

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

add_library(parmetis STATIC ${PARMETIS_SOURCES})
target_include_directories(parmetis PRIVATE libparmetis)
target_include_directories(parmetis PUBLIC include)
# Compile parmetis with 64 bit indices and floating point values
target_compile_definitions(parmetis PRIVATE IDXTYPEWIDTH=64 REALTYPEWIDTH=64)
target_link_libraries(parmetis PRIVATE gklib metis)

find_package(MPI REQUIRED)
target_link_libraries(parmetis PRIVATE MPI::MPI_C)

# Setup alias for usage with FetchContent
add_library(ParMETIS::ParMETIS ALIAS parmetis)


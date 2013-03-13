# exact C++ implementation of lowe's sift program
# author: zerofrog(@gmail.com), Sep 2008
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU Lesser General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# Lesser GNU General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.
cmake_minimum_required (VERSION 2.4.7)
project (libsiftfast)
set( CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS TRUE )
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}")

# Add the automatically determined parts of the RPATH
# which point to directories outside the build tree to the install RPATH
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

include(CheckCXXSourceRuns)

if( CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX )
  set(CMAKE_CXX_FLAGS_RELEASE "-O3 -g")
  add_definitions(" -Wall -fPIC ")

  # check for x86-64 system
  check_cxx_source_runs("
  int main()
  {
    int a = 0;
    int*pa = &a;
    asm(\".intel_syntax\\\\n\"
	\"mov %%rax, %0\\\\n\"
    \"mov %%eax, [%%rax]\\\\n\"
    \".att_syntax\\\\n\"
    : : \"r\"(pa) : \"%rax\");
    return 0;
  }"
  IS_X86_64)

  if( IS_X86_64 )
    add_definitions("-D__x86_64__")
  endif()
else()
  set(IS_X86_64 0)
endif()

include(CheckIncludeFile)
include(CheckLibraryExists)
include(CheckCXXSourceRuns)
include(CheckCXXCompilerFlag)

set(STDC_LIBRARY stdc++)

# check for SSE extensions
if( CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX )
  set(SSE_FLAGS)

  set(CMAKE_REQUIRED_FLAGS "-msse2")
  check_cxx_source_runs("
    #include <emmintrin.h>

    int main()
    {
        __m128d a, b;
        double vals[2] = {0};
        a = _mm_loadu_pd(vals);
        b = _mm_add_pd(a,a);
        _mm_storeu_pd(vals,b);
        return 0;
     }"
     HAS_SSE2_EXTENSIONS)

  set(CMAKE_REQUIRED_FLAGS "-msse")
  check_cxx_source_runs("
    #include <xmmintrin.h>
    int main()
    {
        __m128 a, b;
        float vals[4] = {0};
        a = _mm_loadu_ps(vals);
        b = a;
        b = _mm_add_ps(a,b);
        _mm_storeu_ps(vals,b);
        return 0;
    }"
    HAS_SSE_EXTENSIONS)

  set(CMAKE_REQUIRED_FLAGS)

  if(HAS_SSE2_EXTENSIONS)
    message(STATUS "Using SSE2 extensions")
  elseif(HAS_SSE_EXTENSIONS)
    message(STATUS "Using SSE extensions")
    set(SSE_FLAGS "-msse -mfpmath=sse")
  endif()

  add_definitions(${SSE_FLAGS})
endif()

add_library(libsiftfast SHARED libsiftfast.cpp)
add_library(libsiftfast-static STATIC libsiftfast.cpp)

target_link_libraries(libsiftfast m stdc++)

set(LIBSIFTFAST_NAME siftfast)
set_target_properties(libsiftfast-static PROPERTIES OUTPUT_NAME ${LIBSIFTFAST_NAME} CLEAN_DIRECT_OUTPUT 1 SOVERSION ${SIFTFAST_SOVERSION} VERSION ${SIFTFAST_VERSION})

# check for OpenMP
if( NOT DEFINED USE_OPENMP OR USE_OPENMP  )

  if( WIN32 )
    CHECK_INCLUDE_FILE(omp.h HAVE_OMP_H)
    if( HAVE_OMP_H )
      message(STATUS "Using OpenMP")
      check_cxx_compiler_flag(/openmp HAVE_OPENMP)

      if( HAVE_OPENMP )
        message(STATUS "compiling with openmp support")
        add_definitions("/openmp")
      endif()
    endif()
  elseif(CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)

    # check if compilers supports -fopenmp
    INCLUDE(CheckCCompilerFlag)
    check_cxx_compiler_flag(-fopenmp HAVE_OPENMP)
    check_library_exists(gomp omp_get_num_threads "" HAS_GOMP_LIB)

    if( HAVE_OPENMP AND HAS_GOMP_LIB )
      message(STATUS "compiling with openmp support")
      add_definitions("-fopenmp")
      target_link_libraries(libsiftfast gomp)
      target_link_libraries(libsiftfast-static gomp)
      set(OPENMP_LFLAGS "-lgomp")
    endif()
  endif()
endif()

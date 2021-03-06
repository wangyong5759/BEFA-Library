CMAKE_MINIMUM_REQUIRED(VERSION 3.2)
PROJECT(BEFA)

SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g -std=c++17 -Wall -Wextra -pedantic -DNASSERT_EX=0")
SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -std=c++17 -Ofast")
# SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -std=c++17 -O3 -D_GLIBCXX_PARALLEL -fopenmp")


IF (ARRAY_VIEW_CHECK_BORDERS)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DARRAY_VIEW_CHECK_BORDERS=1")
ENDIF ()

ENABLE_TESTING()
SET(CTEST_OUTPUT_ON_FAILURE TRUE)

# Finding BFD libraries and headers #
FIND_LIBRARY(BFD_LIBRARY libbfd.a REQUIRED)
IF (EXISTS ${BFD_LIBRARY})
    MESSAGE(STATUS "Binary File Descriptor library found '${BFD_LIBRARY}'")
ELSE ()
    MESSAGE(FATAL_ERROR "Binary File Descriptor library was not found!")
ENDIF ()

FIND_LIBRARY(OPCODE_LIBRARY libopcodes.a REQUIRED)
IF (EXISTS ${OPCODE_LIBRARY})
    MESSAGE(STATUS "OpCodes library found '${OPCODE_LIBRARY}'")
ELSE ()
    MESSAGE(FATAL_ERROR "OpCodes library was not found!")
ENDIF ()

SET(BFD_LIBRARIES bfd opcodes)

# - Find PCRECPP
# Find the native PCRECPP headers and libraries.
#
#  PCRECPP_INCLUDE_DIRS - where to find pcre.h, etc.
#  PCRECPP_LIBRARIES    - List of libraries when using pcrecpp
#  PCRECPP_FOUND        - True if pcrecpp found.

# Look for the header file.
FIND_PATH(PCRECPP_INCLUDE_DIR NAMES pcrecpp.h)
MARK_AS_ADVANCED(PCRECPP_INCLUDE_DIR)

# Look for the library.
FIND_LIBRARY(PCRECPP_LIBRARY NAMES
        pcrecpp
        )
MARK_AS_ADVANCED(PCRECPP_LIBRARY)

FIND_LIBRARY(PCRE_LIBRARY NAMES
        pcre
        pcred
        )
MARK_AS_ADVANCED(PCRE_LIBRARY)

# handle the QUIETLY and REQUIRED arguments and set PCRECPP_FOUND to TRUE if
# all listed variables are TRUE
INCLUDE(FindPackageHandleStandardArgs)
FIND_PACKAGE_HANDLE_STANDARD_ARGS(PCRECPP DEFAULT_MSG PCRECPP_LIBRARY PCRECPP_INCLUDE_DIR PCRE_LIBRARY)

IF(PCRECPP_FOUND)
    SET(PCRECPP_LIBRARIES ${PCRECPP_LIBRARY} ${PCRE_LIBRARY})
    SET(PCRECPP_INCLUDE_DIRS ${PCRECPP_INCLUDE_DIR})
ENDIF(PCRECPP_FOUND)

FIND_PACKAGE (Threads)

ADD_SUBDIRECTORY(bin)

INCLUDE_DIRECTORIES(rxcpp/Rx/v2/src rxcpp/ext/catch/include)
ADD_SUBDIRECTORY(include)
ADD_SUBDIRECTORY(src)

INCLUDE_DIRECTORIES(include)

ADD_SUBDIRECTORY(examples)

ADD_SUBDIRECTORY(tests)

ADD_EXECUTABLE(a.out main.cpp)
TARGET_LINK_LIBRARIES(a.out befa ${RxCpp_Lib})

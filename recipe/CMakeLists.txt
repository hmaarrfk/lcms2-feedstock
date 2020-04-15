cmake_minimum_required(VERSION 3.5)

project(lcms2 C)

option(CMAKE_VERBOSE_MAKEFILE "Create verbose Makefile" OFF)
option(BUILD_SHARED_LIBS "Create ${PROJECT_NAME} as a shared library" ON)
option(BUILD_STATIC_LIBS "Create ${PROJECT_NAME} as a static library" ON)

set(CMAKE_DEBUG_POSTFIX d)

file(GLOB CMS_SOURCES ${CMAKE_CURRENT_LIST_DIR}/src/*.c)

set(CMS_HDRS ${CMAKE_CURRENT_LIST_DIR}/src/lcms2_internal.h)

set(CMS_INCLUDE_DIRS ${CMAKE_CURRENT_LIST_DIR}/include)

if(BUILD_SHARED_LIBS)
    add_library(lcms2 SHARED ${CMS_SOURCES} ${CMS_HDRS})
    # add_library(lcms2 SHARED ${CMS_SOURCES} ${CMS_HDRS} ${CMAKE_CURRENT_LIST_DIR}/src/lcms2.def)
    target_compile_options(lcms2 PRIVATE -DCMS_DLL_BUILD)
    target_compile_options(lcms2 PUBLIC -DCMS_DLL)
    target_compile_options(lcms2 PRIVATE -DUNICODE -D_UNICODE)
    target_include_directories(lcms2 PRIVATE ${CMS_INCLUDE_DIRS})
    set_target_properties(lcms2 PROPERTIES PUBLIC_HEADER "${CMAKE_CURRENT_LIST_DIR}/include/lcms2.h;${CMAKE_CURRENT_LIST_DIR}/include/lcms2_plugin.h")
endif()

if(BUILD_STATIC_LIBS)
    add_library(lcms2_static STATIC ${CMS_SOURCES} ${CMS_HDRS})
    target_compile_options(lcms2_static PRIVATE -DUNICODE -D_UNICODE)
    target_include_directories(lcms2_static PRIVATE ${CMS_INCLUDE_DIRS})
    # set_target_properties(lcms2_static PROPERTIES PUBLIC_HEADER "${CMAKE_CURRENT_LIST_DIR}/include/lcms2.h;${CMAKE_CURRENT_LIST_DIR}/include/lcms2_plugin.h")
endif()

set(UTILS_SOURCES 
    "${CMAKE_CURRENT_LIST_DIR}/utils/common/vprf.c"
    "${CMAKE_CURRENT_LIST_DIR}/utils/common/xgetopt.c"
)

set(UTILS_INCLUDE_DIRS
    "${CMAKE_CURRENT_LIST_DIR}/utils/common"
    "${CMAKE_CURRENT_LIST_DIR}/include"
)

add_executable(jpgicc
    "${CMAKE_CURRENT_LIST_DIR}/utils/jpgicc/iccjpeg.c"
    "${CMAKE_CURRENT_LIST_DIR}/utils/jpgicc/iccjpeg.h"
    "${CMAKE_CURRENT_LIST_DIR}/utils/jpgicc/jpgicc.c"
    ${UTILS_SOURCES}
)
find_package(JPEG REQUIRED)
if(JPEG_FOUND)
  include_directories(${JPEG_INCLUDE_DIRS})
  target_link_libraries(jpgicc ${JPEG_LIBRARIES} -static)
endif(JPEG_FOUND)
target_link_libraries(jpgicc lcms2 -static)
target_include_directories(jpgicc PRIVATE ${UTILS_INCLUDE_DIRS})

add_executable(linkicc
    "${CMAKE_CURRENT_LIST_DIR}/utils/linkicc/linkicc.c"
    ${UTILS_SOURCES}
)
target_link_libraries(linkicc lcms2)
target_include_directories(linkicc PRIVATE ${UTILS_INCLUDE_DIRS})

add_executable(psicc
    "${CMAKE_CURRENT_LIST_DIR}/utils/psicc/psicc.c"
    ${UTILS_SOURCES}
)
target_link_libraries(psicc lcms2)
target_include_directories(psicc PRIVATE ${UTILS_INCLUDE_DIRS})

add_executable(tificc
    "${CMAKE_CURRENT_LIST_DIR}/utils/tificc/tificc.c"
    ${UTILS_SOURCES}
)
find_package(TIFF REQUIRED)
if(TIFF_FOUND)
  include_directories(${TIFF_INCLUDE_DIRS})
  target_link_libraries(tificc ${TIFF_LIBRARIES} -static)
endif(TIFF_FOUND)
target_link_libraries(tificc lcms2 -static)
target_include_directories(tificc PRIVATE ${UTILS_INCLUDE_DIRS})

add_executable(transicc
    "${CMAKE_CURRENT_LIST_DIR}/utils/transicc/transicc.c"
    ${UTILS_SOURCES}
)
target_link_libraries(transicc lcms2)
target_include_directories(transicc PRIVATE ${UTILS_INCLUDE_DIRS})

install(TARGETS lcms2 lcms2_static jpgicc tificc transicc linkicc psicc
    EXPORT lcmsConfig
    RUNTIME DESTINATION "bin"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    PUBLIC_HEADER DESTINATION "include"
    COMPONENT dev
)

export(TARGETS lcms2
    NAMESPACE lcms::
    FILE "share/lcms/lcmsConfig.cmake"
)

install(EXPORT lcmsConfig
    DESTINATION "share/lcms"
    NAMESPACE lcms::
)

cmake_minimum_required(VERSION 3.5)

project(lcms2 C)

set(CMAKE_DEBUG_POSTFIX d)

set(CMS_SOURCES
    ${CMAKE_CURRENT_LIST_DIR}/src/cmsalpha.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmscam02.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmscgats.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmscnvrt.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmserr.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmsgamma.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmsgmt.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmshalf.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmsintrp.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmsio0.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmsio1.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmslut.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmsmd5.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmsmtrx.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmsnamed.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmsopt.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmspack.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmspcs.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmsplugin.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmsps2.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmssamp.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmssm.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmstypes.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmsvirt.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmswtpnt.c
    ${CMAKE_CURRENT_LIST_DIR}/src/cmsxform.c
)

set(CMS_HEADERS ${CMAKE_CURRENT_LIST_DIR}/src/lcms2_internal.h)

set(CMS_INCLUDE_DIRS ${CMAKE_CURRENT_LIST_DIR}/include)

add_library(lcms2 SHARED ${CMS_SOURCES} ${CMS_HEADERS}
    # ${CMAKE_CURRENT_LIST_DIR}/src/lcms2.def   # handles name mangling for 32-bit Windows only
    ${CMAKE_CURRENT_LIST_DIR}/Projects/VC2019/resource.h
    ${CMAKE_CURRENT_LIST_DIR}/Projects/VC2019/lcms2.rc
)
target_compile_options(lcms2 PRIVATE -DUNICODE -D_UNICODE)
target_compile_options(lcms2 PRIVATE -DCMS_DLL_BUILD)
target_compile_options(lcms2 PUBLIC -DCMS_DLL)
target_include_directories(lcms2 PRIVATE ${CMS_INCLUDE_DIRS})
set_target_properties(lcms2 PROPERTIES PUBLIC_HEADER "${CMAKE_CURRENT_LIST_DIR}/include/lcms2.h;${CMAKE_CURRENT_LIST_DIR}/include/lcms2_plugin.h")

add_library(lcms2_static STATIC ${CMS_SOURCES} ${CMS_HEADERS})
target_compile_options(lcms2_static PRIVATE -DUNICODE -D_UNICODE)
target_include_directories(lcms2_static PRIVATE ${CMS_INCLUDE_DIRS})

install(TARGETS lcms2 lcms2_static 
    RUNTIME DESTINATION "bin"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    PUBLIC_HEADER DESTINATION "include"
)

# Sample programs
set(UTILS_SOURCES 
    ${CMAKE_CURRENT_LIST_DIR}/utils/common/vprf.c
    ${CMAKE_CURRENT_LIST_DIR}/utils/common/xgetopt.c
)

set(UTILS_INCLUDE_DIRS
    ${CMAKE_CURRENT_LIST_DIR}/utils/common
    ${CMAKE_CURRENT_LIST_DIR}/include
)

add_executable(jpgicc
    ${CMAKE_CURRENT_LIST_DIR}/utils/jpgicc/iccjpeg.c
    ${CMAKE_CURRENT_LIST_DIR}/utils/jpgicc/iccjpeg.h
    ${CMAKE_CURRENT_LIST_DIR}/utils/jpgicc/jpgicc.c
    ${UTILS_SOURCES}
)
find_package(JPEG REQUIRED)
if(JPEG_FOUND)
  include_directories(${JPEG_INCLUDE_DIRS})
  target_link_libraries(jpgicc ${JPEG_LIBRARIES})
endif(JPEG_FOUND)
target_link_libraries(jpgicc lcms2_static)
target_include_directories(jpgicc PRIVATE ${UTILS_INCLUDE_DIRS})

add_executable(linkicc
    ${CMAKE_CURRENT_LIST_DIR}/utils/linkicc/linkicc.c
    ${UTILS_SOURCES}
)
target_link_libraries(linkicc lcms2_static)
target_include_directories(linkicc PRIVATE ${UTILS_INCLUDE_DIRS})

add_executable(psicc
    ${CMAKE_CURRENT_LIST_DIR}/utils/psicc/psicc.c
    ${UTILS_SOURCES}
)
target_link_libraries(psicc lcms2_static)
target_include_directories(psicc PRIVATE ${UTILS_INCLUDE_DIRS})

add_executable(tifdiff
    ${CMAKE_CURRENT_LIST_DIR}/utils/tificc/tifdiff.c
    ${UTILS_SOURCES}
)
add_executable(tificc
    ${CMAKE_CURRENT_LIST_DIR}/utils/tificc/tificc.c
    ${UTILS_SOURCES}
)
find_package(TIFF REQUIRED)
if(TIFF_FOUND)
  include_directories(${TIFF_INCLUDE_DIRS})
  target_link_libraries(tifdiff ${TIFF_LIBRARIES})
  target_link_libraries(tificc ${TIFF_LIBRARIES})
endif(TIFF_FOUND)
target_link_libraries(tifdiff lcms2_static)
target_include_directories(tifdiff PRIVATE ${UTILS_INCLUDE_DIRS})
target_link_libraries(tificc lcms2_static)
target_include_directories(tificc PRIVATE ${UTILS_INCLUDE_DIRS})

add_executable(transicc
    ${CMAKE_CURRENT_LIST_DIR}/utils/transicc/transicc.c
    ${UTILS_SOURCES}
)
target_link_libraries(transicc lcms2_static)
target_include_directories(transicc PRIVATE ${UTILS_INCLUDE_DIRS})

add_executable(wtpt
    ${CMAKE_CURRENT_LIST_DIR}/utils/samples/wtpt.c
    ${UTILS_SOURCES}
)
target_link_libraries(wtpt lcms2_static)
target_include_directories(wtpt PRIVATE ${UTILS_INCLUDE_DIRS})

install(TARGETS jpgicc linkicc psicc tifdiff tificc transicc wtpt
    RUNTIME DESTINATION "bin"
)

cmake_minimum_required(VERSION 3.4)

project(yaml-cpp VERSION 0.5.3)

include(ExternalProject)

find_package(boost REQUIRED headers)
set(Boost_FOUND true)
get_target_property(Boost_INCLUDE_DIRS boost INTERFACE_INCLUDE_DIRECTORIES)
#get_target_property(BOOST_INCLUDEDIR boost INTERFACE_INCLUDE_DIRECTORIES)
message(STATUS "boost_DIR: ${boost_DIR}")
message(STATUS "Boost_INCLUDE_DIRS: ${Boost_INCLUDE_DIRS}")
set(Boost_DIR ${boost_DIR})


ExternalProject_Add(${PROJECT_NAME}
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}
    URL https://github.com/jbeder/yaml-cpp/archive/release-${PROJECT_VERSION}.tar.gz
    URL_MD5 64200ca0bf5e0af065804d8d3e8f6d42
    INSTALL_DIR ${CMAKE_CURRENT_BINARY_DIR}
    CMAKE_CACHE_ARGS
        -DBoost_FOUND:bool=true
        -DBoost_INCLUDE_DIRS:path=${Boost_INCLUDE_DIRS}
        -DBoost_INCLUDE_DIR:path=${Boost_INCLUDE_DIRS}
        -DYAML_CPP_BUILD_TOOLS:bool=OFF
        -DCMAKE_INSTALL_PREFIX:path=<INSTALL_DIR>
    BUILD_IN_SOURCE 1
)

#        -DBoost_DIR:path=${boost_DIR}
#        -Dboost_DIR:path=${boost_DIR}
#        -DBOOST_DIR:path=${boost_DIR}
#        -DBOOST_INCLUDEDIR:path=${boost_DIR}/include
#        -DBoost_FOUND:bool=true
#        -DBoost_FOUND:bool=true
#        -DBoost_INCLUDE_DIR=${Boost_INCLUDE_DIR}
#        -DCMAKE_INSTALL_PREFIX:string=<INSTALL_DIR>
#        -DNO_DEFAULT_PATH:bool=true
#        -DNO_CMAKE_PATH:bool=true
#        -DNO_CMAKE_ENVIRONMENT_PATH:bool=true


ExternalProject_Get_Property(${PROJECT_NAME} INSTALL_DIR)

string(TOLOWER "${PROJECT_NAME}" PROJECT_NAME_LOWER)

configure_file(config.cmake.in
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME_LOWER}-config.cmake"
    @ONLY)

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME_LOWER}-config-version.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)


install(DIRECTORY
    "${INSTALL_DIR}/include"
    DESTINATION "lib/${CMAKE_LIBRARY_ARCHITECTURE}/${PROJECT_NAME_LOWER}-${PROJECT_VERSION}"
)

install(DIRECTORY
    "${INSTALL_DIR}/lib"
    DESTINATION "lib/${CMAKE_LIBRARY_ARCHITECTURE}/${PROJECT_NAME_LOWER}-${PROJECT_VERSION}"
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME_LOWER}-config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME_LOWER}-config-version.cmake"
    DESTINATION "lib/${CMAKE_LIBRARY_ARCHITECTURE}/${PROJECT_NAME_LOWER}-${PROJECT_VERSION}"
)

export(PACKAGE ${PROJECT_NAME})

set(CPACK_GENERATOR "TBZ2")
set(CPACK_PACKAGE_FILE_NAME "${PROJECT_NAME_LOWER}-${PROJECT_VERSION}-${CMAKE_LIBRARY_ARCHITECTURE}")
include(CPack)

if(CMAKE_CURRENT_BINARY_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    set_property(DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
        PROPERTY ADDITIONAL_MAKE_CLEAN_FILES
            include/
            lib/
            ${PROJECT_NAME}-config.cmake
            ${PROJECT_NAME}-config-version.cmake
            ${CPACK_PACKAGE_FILE_NAME}.tar.bz2
    )
endif()

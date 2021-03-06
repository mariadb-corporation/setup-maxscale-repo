project(configure-maxscale-repo)
cmake_minimum_required(VERSION 2.8)

include(macros.cmake)

set_maxscale_version()

# Installation directory
set(CMAKE_INSTALL_PREFIX "/usr/local/mariadb-maxscale-setup" CACHE INTERNAL "Prefix prepended to install directories." FORCE)
set(CMAKE_BUILD_TYPE "None" CACHE STRING "Build type, possible values are:None, Debug, Optimized.")
set(REPO_URL "downloads.mariadb.com/software/mariadb-maxscale/1.1")

aux_source_directory(. SRC_LIST)

configure_file(${CMAKE_SOURCE_DIR}/postinst.in ${CMAKE_BINARY_DIR}/postinst @ONLY)
configure_file(${CMAKE_SOURCE_DIR}/configure-maxscale-repo.sh.in ${CMAKE_BINARY_DIR}/configure-maxscale-repo @ONLY)


install(FILES configure-maxscale-repo DESTINATION . PERMISSIONS WORLD_READ WORLD_EXECUTE OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE)
install(FILES Maxscale-GPG-KEY.public DESTINATION .)


# See if we are on a RPM-capable or DEB-capable system
find_program(RPMBUILD rpmbuild)
find_program(DEBBUILD dpkg-buildpackage)

if(NOT ( ${RPMBUILD} STREQUAL "RPMBUILD-NOTFOUND" ) )
  message(STATUS "Generating RPM packages")
  set(CPACK_GENERATOR "${CPACK_GENERATOR};RPM")
endif()

if(NOT ( ${DEBBUILD} STREQUAL "DEBBUILD-NOTFOUND" ) )
  set(CPACK_GENERATOR "${CPACK_GENERATOR};DEB")
  execute_process(COMMAND dpgk --print-architecture OUTPUT_VARIABLE DEB_ARCHITECTURE)
  set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "all")
  set (CPACK_DEBIAN_PACKAGE_SHLIBDEPS OFF)
#  set(CPACK_DEBIAN_PACKAGE_DEPENDS "bash, sed, apt-transport-https")
  set(CPACK_DEBIAN_PACKAGE_DEPENDS "bash, sed")
  set(CPACK_DEBIAN_PACKAGE_CONTROL_EXTRA ${CMAKE_BINARY_DIR}/postinst)
  message(STATUS "Generating DEB packages")
endif()

set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "configure-maxscale-repo")
set(CPACK_PACKAGE_VERSION_MAJOR "${MAXSCALE_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${MAXSCALE_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${MAXSCALE_VERSION_PATCH}")
set(CPACK_RPM_PACKAGE_ARCHITECTURE "noarch")
set(CPACK_RPM_POST_INSTALL_SCRIPT_FILE ${CMAKE_BINARY_DIR}/postinst)
set(CPACK_PACKAGE_CONTACT "MariaDB Corporation Ab")
set(CPACK_PACKAGE_FILE_NAME "configure-maxscale-repo-${MAXSCALE_VERSION}")
set(CPACK_PACKAGE_NAME "configure-maxscale-repo")
set(CPACK_PACKAGE_VENDOR "MariaDB Corporation Ab")
set(CPACK_PACKAGE_DESCRIPTION_FILE ${CMAKE_SOURCE_DIR}/README)
set(CPACK_PACKAGING_INSTALL_PREFIX "${CMAKE_INSTALL_PREFIX}")
set(CPACK_RPM_PACKAGE_NAME "configure-maxscale-repo")
set(CPACK_RPM_PACKAGE_VENDOR "MariaDB Corporation Ab")
set(CPACK_RPM_PACKAGE_LICENSE "GPLv2")
include(CPack)


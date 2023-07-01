cmake_minimum_required(VERSION 3.20)

set(NAME mpc)
set(VERSION 1.3.1)

project(cmeel-${NAME} VERSION ${VERSION})

include(FindPkgConfig)
pkg_check_modules(GMP gmp)

if(APPLE)
  set(ORIGIN "@loader_path")
else()
  set(ORIGIN "\$\$ORIGIN")
endif()

include(ExternalProject)
ExternalProject_Add(
  ${NAME}
  URL "https://ftpmirror.gnu.org/gnu/${NAME}/${NAME}-${VERSION}.tar.gz"
      DOWNLOAD_EXTRACT_TIMESTAMP OFF
  BUILD_IN_SOURCE ON
  CONFIGURE_COMMAND
    ${CMAKE_COMMAND} "-E" "env" "LDFLAGS=-Wl,-rpath,'${ORIGIN}/../lib'" "--"
    "./configure" "--prefix=${CMAKE_INSTALL_PREFIX}" "--with-gmp=${GMP_PREFIX}"
    "--with-mpfr=${GMP_PREFIX}" # this is not super nice.
  BUILD_COMMAND "make"
  TEST_COMMAND ${CMAKE_COMMAND} "-E" "env" "LD_LIBRARY_PATH=${GMP_PREFIX}/lib"
               "--" "make" "check"
  INSTALL_COMMAND "make" "install"
  COMMAND "${CMAKE_COMMAND}" "-E" "rm"
          "${CMAKE_INSTALL_PREFIX}/lib/lib${NAME}.la")

# dummy file for install target
install(FILES README.md DESTINATION share/cmeel-${NAME}/)

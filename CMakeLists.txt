cmake_minimum_required(VERSION 3.20)

set(NAME mpc)
set(VERSION 1.3.1)

project(cmeel-${NAME} VERSION ${VERSION})

include(FindPkgConfig)
pkg_check_modules(GMP REQUIRED gmp)
pkg_check_modules(MPFR REQUIRED mpfr)

if(APPLE)
  set(ORIGIN "@loader_path")
  set(TEST_COMMAND ${CMAKE_COMMAND} "-E" "env"
                   "DYLD_LIBRARY_PATH=${GMP_PREFIX}/lib" "--" "make" "check")
else()
  set(ORIGIN "\$\$ORIGIN")
  set(TEST_COMMAND ${CMAKE_COMMAND} "-E" "env"
                   "LD_LIBRARY_PATH=${GMP_PREFIX}/lib" "--" "make" "check")
endif()

include(ExternalProject)
ExternalProject_Add(
  ${NAME}
  URL "https://ftpmirror.gnu.org/gnu/${NAME}/${NAME}-${VERSION}.tar.gz"
  URL_HASH "SHA1=bac1c1fa79f5602df1e29e4684e103ad55714e02"
  DOWNLOAD_EXTRACT_TIMESTAMP OFF
  BUILD_IN_SOURCE ON
  CONFIGURE_COMMAND
    ${CMAKE_COMMAND} "-E" "env" "LDFLAGS=-Wl,-rpath,'${ORIGIN}/../lib'" "--"
    "./configure" "--prefix=${CMAKE_INSTALL_PREFIX}" "--with-gmp=${GMP_PREFIX}"
    "--with-mpfr=${MPFR_PREFIX}"
  BUILD_COMMAND "make"
  TEST_COMMAND ${TEST_COMMAND}
  INSTALL_COMMAND "make" "install"
  COMMAND "${CMAKE_COMMAND}" "-E" "rm"
          "${CMAKE_INSTALL_PREFIX}/lib/lib${NAME}.la")

# dummy file for install target
install(FILES README.md DESTINATION share/cmeel-${NAME}/)

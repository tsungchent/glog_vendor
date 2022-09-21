cmake_minimum_required(VERSION 3.5)
project(glog_vendor)

find_package(ament_cmake REQUIRED)

include(ExternalProject)

# We can't check version numbers easily as the GLOG script doesn't export them.
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/include)

set(VERSION 0.3.5)
ExternalProject_Add(glog_src
  URL https://github.91chi.fun//https://github.com/google/glog/archive/v${VERSION}.zip
  UPDATE_COMMAND ""
  PATCH_COMMAND patch -p0 < ${CMAKE_CURRENT_SOURCE_DIR}/fix-unused-typedef-warning.patch
  CONFIGURE_COMMAND cd ../glog_src && 
    cmake .
      -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_CURRENT_BINARY_DIR}/glog_install
      -DBUILD_SHARED_LIBS:BOOL=OFF
      -DCMAKE_BUILD_TYPE:STRING=Release
      -DCMAKE_TOOLCHAIN_FILE:STRING=${CMAKE_TOOLCHAIN_FILE}
  BUILD_COMMAND cd ../glog_src && cmake --build . --config Release
  INSTALL_COMMAND cd ../glog_src && cmake --install .
  #CONFIGURE_COMMAND cd ../glog_src/ && autoreconf -fi && ./configure --with-pic
  #  --with-gflags=${gflags_vendor_PREFIX}
  #  --prefix=${CMAKE_CURRENT_BINARY_DIR}
  #BUILD_COMMAND cd ../glog_src/ && make -j 8
  #INSTALL_COMMAND cd ../glog_src/ && make install -j 8
)

install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/glog_install/include/glog
        DESTINATION ${CMAKE_INSTALL_PREFIX}/include
        FILES_MATCHING PATTERN "*.h")
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/glog_install/lib/
        DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
        FILES_MATCHING PATTERN "*glog*")

ament_package(CONFIG_EXTRAS "cmake/glog-vendor-extras.cmake.in")
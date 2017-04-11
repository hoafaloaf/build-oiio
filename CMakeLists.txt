##  Example: cmake .. -DINSTALL_ROOT=/build_source/payload -DBUILD_PYTHON=1 -DUSE_NUKE=1 -DNuke_ROOT=/apps/nuke/10.0v3 -DBUILD_OCIO=1 -DOIIO_LIBRARY_SUFFIX=HOAF -DWRAP_EXECUTABLES=1 -DUSE_QT4=1 -DQT_HOME=/apps/qt

CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

project(oiio)

include(ExternalProject)

set(CMAKE_BUILD_TYPE release)

set(OIIO_VERSION "1.6.18" CACHE STRING "Version of OpenImageIO you would like to build.")
set(EXTERNAL_DIR ".lib" CACHE STRING "Sub-directory where the external libraries will live.")
set(INSTALL_ROOT "" CACHE PATH "Where you'd like to install OpenImageIO!")
set(USE_QT4 ON CACHE BOOL "Whether you'd like to build any executables that actually need Qt4.")
set(QT_HOME "" CACHE PATH "(Optional) root of Qt installation")
set(SOURCE_ROOT "" CACHE PATH "Where you want the collected source files to go.")
set(SOURCE_DIR ".src" CACHE STRING "Sub-directory where the source tarballs will live.")
set(INSTALL_SOURCE OFF CACHE BOOL "Whether to add the source tarballs to the installation")
set(BUILD_PYTHON OFF CACHE BOOL "Build against a custom version of Python.")
set(BUILD_PYTHON_VERSION "2.7.3" CACHE STRING "Custom Python version.")
set(BUILD_DIR "${PROJECT_NAME}_build" CACHE STRING "Location where to which you'd like to build the project.")
set(Nuke_ROOT "" CACHE PATH "Root of Nuke installation for which you'd like to build nodes.")
set(USE_NUKE OFF CACHE BOOL "Build Nuke plugins if Nuke is found.")
set(USE_QT4 ON CACHE BOOL "Whether you'd like to build any executables that actually need Qt4.")
set(QT_HOME "" CACHE PATH "(Optional) root of Qt installation")
set(BUILD_OCIO OFF CACHE BOOL "(Optional) whether to build OpenColorIO at the same time.")
set(OCIO_VERSION "1.0.9" CACHE STRING "The version of OCIO that you'd like to build.")
set(OCIO_PATH "" CACHE PATH "External location of the OCIO installation against which you'd like to build OIIO.")
set(USE_OCIO ON CACHE BOOL "Whether you'd like to build OIIO against OCIO.")
set(OIIO_LIBRARY_SUFFIX "HOAF" CACHE STRING "Suffix you'd like to add to your OIIO and (optional) OCIO libraries.")
set(WRAP_EXECUTABLES ON CACHE BOOL "Whether to wrap all executables upon installation.")

string(REGEX MATCHALL "[0-9]+" GCC_VERSION_COMPONENTS ${CMAKE_C_COMPILER_VERSION})
list(GET GCC_VERSION_COMPONENTS 0 GCC_VERSION_MAJOR)
list(GET GCC_VERSION_COMPONENTS 1 GCC_VERSION_MINOR)

if(NOT "$ENV{CC}" STREQUAL "")
    get_filename_component(gcc_root "${CMAKE_C_COMPILER}" DIRECTORY)
    get_filename_component(gcc_root "${gcc_root}" DIRECTORY)
    set(gcc_lib64 "${gcc_root}/lib64")
endif()

set(gcc_version_nodots ${GCC_VERSION_MAJOR}${GCC_VERSION_MINOR})

string(REGEX MATCHALL "[0-9]+" OIIO_VERSION_COMPONENTS ${OIIO_VERSION})
list(GET OIIO_VERSION_COMPONENTS 0 OIIO_VERSION_MAJOR)
list(GET OIIO_VERSION_COMPONENTS 1 OIIO_VERSION_MINOR)
list(GET OIIO_VERSION_COMPONENTS 2 OIIO_VERSION_PATCH)

string(REGEX MATCHALL "[0-9]+" OCIO_VERSION_COMPONENTS ${OCIO_VERSION})
list(GET OCIO_VERSION_COMPONENTS 0 OCIO_VERSION_MAJOR)
list(GET OCIO_VERSION_COMPONENTS 1 OCIO_VERSION_MINOR)
list(GET OCIO_VERSION_COMPONENTS 2 OCIO_VERSION_PATCH)

set(CMAKE_MODULE_PATH 
    "${PROJECT_SOURCE_DIR}/cmake/modules"
    "${PROJECT_SOURCE_DIR}/cmake"
)

set(boost_ver "1.60.0")
#set(boost_ver "1.61.0")
set(field3d_ver "1.4.1")
set(hdf5_ver "1.8.17")
set(ilmbase_ver "2.2.0")
set(jasper_ver "1.900.1")
set(jbigkit_ver "2.1")
set(freeglut_ver "2.8.1")
set(giflib_ver "5.1.0")
set(glew_ver "1.10.0")
#set(jpeg_ver "6.2")
set(lcms2_ver "2.8")
set(libjpegturbo_ver "1.5.0")
set(libpng_ver "1.2.56")
set(libraw_ver "0.17.2")
set(libtiff_ver "4.0.6")
set(libwebp_ver "0.5.1")
set(nasm_ver "2.12.02")
set(openexr_ver "2.2.0")
set(openjpeg_ver "2.1.1")
set(openssl_ver "1.0.1u")
set(patchelf_ver "0.9")
set(ptex_ver "2.1.28")
set(szip_ver "2.1")
set(xz_ver "5.2.2")
set(zlib_ver "1.2.11")

string(REGEX MATCHALL "[0-9]+" OPENJPEG_VERSION_COMPONENTS ${openjpeg_ver})
list(GET OPENJPEG_VERSION_COMPONENTS 0 OPENJPEG_VERSION_MAJOR)
list(GET OPENJPEG_VERSION_COMPONENTS 1 OPENJPEG_VERSION_MINOR)
set(openjpeg_ver_major ${OPENJPEG_VERSION_MAJOR}.${OPENJPEG_VERSION_MINOR})

# Setting up the URLs ...
string(REGEX REPLACE "\\." "_" boost_ver_underscores ${boost_ver})
string(REGEX REPLACE "_[0-9]+$" "" boost_ver_major ${boost_ver_underscores})

string(REGEX REPLACE "\\." "_" openssl_ver_underscores ${openssl_ver})

set(boost_url "https://sourceforge.net/projects/boost/files/boost/${boost_ver}/boost_${boost_ver_underscores}.tar.gz/download")
set(freeglut_url "http://sourceforge.net/projects/freeglut/files/freeglut/${freeglut_ver}/freeglut-${freeglut_ver}.tar.gz")
set(giflib_url "http://sourceforge.net/projects/giflib/files/giflib-${giflib_ver}.tar.gz")
set(glew_url "http://pilotfiber.dl.sourceforge.net/project/glew/glew/${glew_ver}/glew-${glew_ver}.tgz")
set(hdf5_url "http://support.hdfgroup.org/ftp/HDF5/releases/hdf5-${hdf5_ver}/src/hdf5-${hdf5_ver}.tar.gz")
set(ilmbase_url "http://download.savannah.nongnu.org/releases/openexr/ilmbase-${ilmbase_ver}.tar.gz")
set(jasper_url "http://www.ece.uvic.ca/~frodo/jasper/software/jasper-${jasper_ver}.zip")
set(jbigkit_url "http://www.cl.cam.ac.uk/~mgk25/jbigkit/download/jbigkit-${jbigkit_ver}.tar.gz")
set(libpng_url "http://download.sourceforge.net/libpng/libpng-${libpng_ver}.tar.gz")
set(libpng_patch_url "http://downloads.sourceforge.net/libpng-apng/libpng-${libpng_ver}-apng.patch.gz")
set(libraw_url "http://www.libraw.org/data/LibRaw-${libraw_ver}.tar.gz")
set(libtiff_url "http://download.osgeo.org/libtiff/tiff-${libtiff_ver}.tar.gz")
set(libjpegturbo_url "https://github.com/libjpeg-turbo/libjpeg-turbo/archive/${libjpegturbo_ver}.tar.gz")
set(libwebp_url "http://storage.googleapis.com/downloads.webmproject.org/releases/webp/libwebp-${libwebp_ver}.tar.gz")
set(nasm_url "http://www.nasm.us/pub/nasm/releasebuilds/${nasm_ver}/nasm-${nasm_ver}.tar.xz")
set(openexr_url "http://download.savannah.nongnu.org/releases/openexr/openexr-${openexr_ver}.tar.gz")
set(python_url "https://www.python.org/ftp/python/${BUILD_PYTHON_VERSION}/Python-${BUILD_PYTHON_VERSION}.tar.xz")
set(szip_url "http://support.hdfgroup.org/ftp/lib-external/szip/${szip_ver}/src/szip-${szip_ver}.tar.gz")
set(xz_url "http://tukaani.org/xz/xz-${xz_ver}.tar.gz")
set(zlib_url "http://zlib.net/zlib-${zlib_ver}.tar.gz")

set(ocio_repo "https://github.com/imageworks/OpenColorIO.git")
set(ocio_tag "v${OCIO_VERSION}")
set(oiio_repo "https://github.com/OpenImageIO/oiio.git")
set(oiio_tag "Release-${OIIO_VERSION}")

set(field3d_repo "https://github.com/imageworks/Field3D.git")
set(field3d_tag "v${field3d_ver}")
set(jbigkit_repo "https://www.cl.cam.ac.uk/~mgk25/git/jbigkit")
set(jbigkit_tag "v${jbigkit_ver}")
set(lcms2_repo "https://github.com/mm2/Little-CMS.git")
set(lcms2_tag "lcms${lcms2_ver}")
set(openjpeg_repo "https://github.com/uclouvain/openjpeg.git")
set(openjpeg_tag "v${openjpeg_ver}")
set(openssl_repo "https://github.com/openssl/openssl.git")
set(openssl_tag "OpenSSL_${openssl_ver_underscores}")
set(patchelf_repo "https://github.com/NixOS/patchelf.git")
set(patchelf_tag "${patchelf_ver}")
set(ptex_repo "https://github.com/wdas/ptex.git")
set(ptex_tag "v${ptex_ver}")

##  Storing a list of all the tarballs for easy source archive installation ...
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/oiio-${OIIO_VERSION}/src/oiio-${OIIO_VERSION}.tar.gz)

list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/boost-${boost_ver}/src/boost-${boost_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/field3d-${field3d_ver}/src/field3d-${field3d_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/freeglut-${freeglut_ver}/src/freeglut-${freeglut_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/glew-${glew_ver}/src/glew-${glew_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/giflib-${giflib_ver}/src/giflib-${giflib_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/hdf5-${hdf5_ver}/src/hdf5-${hdf5_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/ilmbase-${ilmbase_ver}/src/ilmbase-${ilmbase_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/jasper-${jasper_ver}/src/jasper-${jasper_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/jbigkit-${jbigkit_ver}/src/jbigkit-${jbigkit_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/lcms2-${lcms2_ver}/src/lcms2-${lcms2_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/libpng-${libpng_ver}/src/libpng-${libpng_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/libjpegturbo-${libjpegturbo_ver}/src/libjpegturbo-${libjpegturbo_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/libwebp-${libwebp_ver}/src/libwebp-${libwebp_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/xz-${xz_ver}/src/xz-${xz_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/nasm-${nasm_ver}/src/nasm-${nasm_ver}.tar.xz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/openexr-${openexr_ver}/src/openexr-${openexr_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/patchelf-${patchelf_ver}/src/patchelf-${patchelf_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/ptex-${ptex_ver}/src/ptex-${ptex_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/szip-${szip_ver}/src/szip-${szip_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/openjpeg-${openjpeg_ver}/src/openjpeg-${openjpeg_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/openssl-${openssl_ver}/src/openssl-${openssl_ver}.tar.gz)
list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/zlib-${zlib_ver}/src/zlib-${zlib_ver}.tar.gz)

##  Adding in the patches ...
list(APPEND patches ${CMAKE_CURRENT_BINARY_DIR}/libpng-${libpng_ver}/src/libpng-${libpng_ver}-apng.patch.gz)
list(APPEND patches ${CMAKE_SOURCE_DIR}/patches/jbigkit-${jbigkit_ver}.patch)
list(APPEND patches ${CMAKE_SOURCE_DIR}/patches/ptex-${ptex_ver}.patch)
list(APPEND patches ${CMAKE_SOURCE_DIR}/patches/libraw-${libraw_ver}.patch)
list(APPEND patches ${CMAKE_SOURCE_DIR}/patches/oiio-${OIIO_VERSION}.patch)

find_package(Doxygen)

if(USE_QT4)
    if(QT_HOME)
        find_program(QT_QMAKE_EXECUTABLE NAMES qmake qmake4 qmake-qt4 PATHS ${QT_HOME}/bin)
    else()
        find_program(QT_QMAKE_EXECUTABLE NAMES qmake qmake4 qmake-qt4)
    endif()

    if(QT_QMAKE_EXECUTABLE)
        find_package(Qt4 REQUIRED QtCore QtDBus QtGui QtOpenGL)
        set(qt_libraries ${QT_QTCORE_LIBRARY} ${QT_QTDBUS_LIBRARY} ${QT_QTGUI_LIBRARY} ${QT_QTOPENGL_LIBRARY})
        message(STATUS "QT_QMAKE_EXECUTABLE: ${QT_QMAKE_EXECUTABLE}")
        message(STATUS "QT_LIBRARIES: ${qt_libraries}")
    else()
        message(STATUS "Unable to find qmake, will not build Qt interfaces.")
        set(USE_QT4 OFF)
        set(QT_HOME "")
    endif()
endif()

if(NOT DOXYGEN_FOUND)
    message(FATAL_ERROR "Unfortunately, you'll need to have Doxygen installed "
        "to continue."
    )
endif()

if(NOT INSTALL_ROOT)
    message(FATAL_ERROR "Ya gotta put your installation somewhere! Please "
        "specify \$INSTALL_ROOT on the command line."
    )
endif()

if(INSTALL_SOURCE)
    get_filename_component(SOURCE_ROOT "${SOURCE_ROOT}" REALPATH)
    if(NOT SOURCE_ROOT)
        set(default_source_root ON)
        set(SOURCE_ROOT "${INSTALL_ROOT}/${SOURCE_DIR}")
    endif()
endif()

if(NOT DOXYGEN_FOUND)
    message(FATAL_ERROR "Unfortunately, you'll need to have Doxygen installed "
        "to continue."
    )
endif()

if(WRAP_EXECUTABLES)
    message(STATUS "Wrapping all executables!")
endif()

##  Build the prerequesites (static libraries/includes go here) ...
set(prereq_root "${CMAKE_CURRENT_BINARY_DIR}/prerequisites")
file(MAKE_DIRECTORY ${prereq_root})

if(USE_NUKE)
    find_package(Nuke)
    if(NUKE_FOUND)
        message(STATUS "NUKE_EXECUTABLE: ${NUKE_EXECUTABLE}")
    else()
        message(STATUS "Could not Find Nuke. Skipping build of Nuke plugins.")
        set(USE_NUKE OFF)
    endif()
endif()

if(BUILD_PYTHON)
    message(
        STATUS "Building custom version of Python (${BUILD_PYTHON_VERSION}).")

    list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/python-${BUILD_PYTHON_VERSION}/src/python-${BUILD_PYTHON_VERSION}.tar.gz)

    ExternalProject_add(
        python
        PREFIX python-${BUILD_PYTHON_VERSION}
        BUILD_COMMAND make
        BUILD_IN_SOURCE 1
        CONFIGURE_COMMAND
            CFLAGS=-fPIC
            CPPFLAGS=-fPIC
            LDFLAGS=-Wl,-rpath,${prereq_root}/lib
            ./configure --prefix=${prereq_root} --enable-shared
                --enable-unicode=ucs4 --enable-big-digits --enable-ipv6
        DOWNLOAD_COMMAND curl -L -k ${python_url} -o python-${BUILD_PYTHON_VERSION}.tar.xz
        COMMAND ${CMAKE_COMMAND} -E make_directory python
        COMMAND unxz < python-${BUILD_PYTHON_VERSION}.tar.xz > python-${BUILD_PYTHON_VERSION}.tar
        COMMAND tar xf python-${BUILD_PYTHON_VERSION}.tar -C python --strip 1
        INSTALL_COMMAND make install
        URL ${python_url}
    )

    string(REGEX MATCHALL "[0-9]+" PYTHON_VERSION_COMPONENTS ${BUILD_PYTHON_VERSION})
    list(GET PYTHON_VERSION_COMPONENTS 0 PYTHON_VERSION_MAJOR)
    list(GET PYTHON_VERSION_COMPONENTS 1 PYTHON_VERSION_MINOR)

    set(python_ver "${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR}")

    set(PYTHON_EXECUTABLE "${prereq_root}/bin/python")
    set(PYTHON_INCLUDE_DIR "${prereq_root}/include/python${python_ver}")
    set(PYTHON_LIBRARY "${prereq_root}/lib/libpython${python_ver}.so")

else()
    find_package(PythonInterp)
    find_package(PythonLibs)
    set(python_ver "${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR}")

    add_custom_target(python ALL ${CMAKE_COMMAND} -E echo "Building against python ${PYTHON_VERSION_STRING}")
endif()

set(python_ver_nodots "${PYTHON_VERSION_MAJOR}${PYTHON_VERSION_MINOR}")
get_filename_component(PYTHON_LIBRARY_DIR "${PYTHON_LIBRARY}" DIRECTORY)

get_filename_component(BUILD_ROOT "./${BUILD_DIR}" REALPATH)
get_filename_component(EXTERNAL_ROOT "${BUILD_ROOT}/${EXTERNAL_DIR}" REALPATH)
get_filename_component(INSTALL_ROOT "${INSTALL_ROOT}" REALPATH)
get_filename_component(SOURCE_ROOT "${SOURCE_ROOT}" REALPATH)

message("BUILDING TO: ${BUILD_ROOT}")
message("INSTALLING TO: ${INSTALL_ROOT}")
if(INSTALL_SOURCE)
    message("SOURCE ROOT: ${SOURCE_ROOT}")
endif()
message("EXTERNAL LIBRARIES: ${INSTALL_ROOT}/${EXTERNAL_DIR}")

set(CMAKE_INSTALL_PREFIX "${INSTALL_ROOT}")

if(BUILD_OCIO)
    set(OCIO_PATH "${BUILD_ROOT}")
    set(USE_OCIO ON)
    message("Building OCIO ${OCIO_VERSION} to ${OCIO_PATH}")
endif()

##  These will be used by multiple build scripts below.
set(shared_cppflags "-I${EXTERNAL_ROOT}/include -I${prereq_root}/include")
set(shared_ldflags "-L${EXTERNAL_ROOT}/lib -L${prereq_root}/lib -Wl,-rpath,${EXTERNAL_ROOT}/lib")

ExternalProject_add(
    patchelf
    PREFIX patchelf-${patchelf_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND ./configure --prefix=${prereq_root}
    GIT_REPOSITORY ${patchelf_repo}
    GIT_TAG ${patchelf_tag}
    INSTALL_COMMAND make install
    PATCH_COMMAND ./bootstrap.sh
    URL ${patchelf_url}
)

ExternalProject_Add_Step(
    patchelf install_source
    COMMAND
        git archive --format=tar.gz --prefix=patchelf-${patchelf_ver}/
            ${patchelf_tag} > ../patchelf-${patchelf_ver}.tar.gz
    DEPENDEES install
    COMMENT "Export versioned snapshot of patchelf repository ..."
    WORKING_DIRECTORY
        ${CMAKE_CURRENT_BINARY_DIR}/patchelf-${patchelf_ver}/src/patchelf
)

ExternalProject_add(
    nasm
    PREFIX nasm-${nasm_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND ./configure --prefix=${prereq_root}
    DOWNLOAD_COMMAND curl -L -k ${nasm_url} -o nasm-${nasm_ver}.tar.xz
    COMMAND ${CMAKE_COMMAND} -E make_directory nasm
    COMMAND unxz < nasm-${nasm_ver}.tar.xz > nasm-${nasm_ver}.tar
    COMMAND tar xf nasm-${nasm_ver}.tar -C nasm --strip 1
    INSTALL_COMMAND make install
    URL ${nasm_url}
)

configure_file(in/boost_configure.in boost_configure)

ExternalProject_add(
    boost
    PREFIX boost-${boost_ver}
    BUILD_IN_SOURCE 1
    BUILD_COMMAND ./bootstrap.sh --prefix=${EXTERNAL_ROOT}
    CONFIGURE_COMMAND ""
    DOWNLOAD_NAME boost-${boost_ver}.tar.gz
    INSTALL_COMMAND bash ${CMAKE_CURRENT_BINARY_DIR}/boost_configure
    URL ${boost_url}
    DEPENDS python

)

ExternalProject_add(
    ilmbase
    PREFIX ilmbase-${ilmbase_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND ./configure --prefix=${prereq_root} --with-pic
        --disable-shared
    DOWNLOAD_NAME ilmbase-${ilmbase_ver}.tar.gz
    INSTALL_COMMAND make install
    URL ${ilmbase_url}
)

ExternalProject_add(
    openexr
    PREFIX openexr-${openexr_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND ./configure --prefix=${prereq_root} --with-pic
        --disable-shared --with-ilmbase-prefix=${prereq_root}
    DOWNLOAD_NAME openexr-${openexr_ver}.tar.gz
    INSTALL_COMMAND make install
    URL ${openexr_url}
    DEPENDS ilmbase
)

ExternalProject_add(
    glew
    PREFIX glew-${glew_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND ""
    INSTALL_COMMAND GLEW_DEST=${EXTERNAL_ROOT} make install
    UPDATE_COMMAND
        perl -p -i -e "s#(LIBDIR = .*/lib)64#\$1#" ./config/Makefile.linux
    DOWNLOAD_NAME glew-${glew_ver}.tar.gz
    URL ${glew_url}
)

ExternalProject_add(
    freeglut
    PREFIX freeglut-${freeglut_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND
        PKG_CONFIG_PATH=${EXTERNAL_ROOT}/lib/pkgconfig:${prereq_root}/lib/pkgconfig
        ./configure --prefix=${EXTERNAL_ROOT} --with-pic --enable-shared
            --disable-static
    DOWNLOAD_COMMAND
        curl -L -k ${freeglut_url} -o freeglut-${freeglut_ver}.tar.gz
    COMMAND ${CMAKE_COMMAND} -E make_directory freeglut
    COMMAND tar xf freeglut-${freeglut_ver}.tar.gz -C freeglut --strip 1
    DOWNLOAD_NAME freeglut-${freeglut_ver}.tar.gz
    INSTALL_COMMAND make install
    URL ${freeglut_url}
)

ExternalProject_add(
    xz
    PREFIX xz-${xz_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND
        CFLAGS=-fPIC
        ./configure --prefix=${prereq_root} --with-pic --disable-shared
    DOWNLOAD_NAME xz-${xz_ver}.tar.gz
    INSTALL_COMMAND make install
    URL ${xz_url}
)

ExternalProject_add(
    zlib
    PREFIX zlib-${zlib_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND
        CFLAGS=-fPIC
        ./configure --prefix=${prereq_root} --64 --static
    DOWNLOAD_NAME zlib-${zlib_ver}.tar.gz
    INSTALL_COMMAND make install
    URL ${zlib_url}
)

ExternalProject_add(
    szip
    PREFIX szip-${szip_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND
        PKG_CONFIG_PATH=${EXTERNAL_ROOT}/lib/pkgconfig:${prereq_root}/lib/pkgconfig
        ./configure --prefix=${prereq_root} --with-pic --disable-shared
    DOWNLOAD_NAME szip-${szip_ver}.tar.gz
    INSTALL_COMMAND make install
    URL ${szip_url}
)

ExternalProject_add(
    hdf5
    PREFIX hdf5-${hdf5_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND
        PKG_CONFIG_PATH=${EXTERNAL_ROOT}/lib/pkgconfig:${prereq_root}/lib/pkgconfig
        ./configure --prefix=${EXTERNAL_ROOT} --with-pic --enable-production
            --enable-cxx --with-pthread=/usr/include,/usr/lib
            --with-szlib=${prereq_root} --with-zlib=${prereq_root}
    DOWNLOAD_NAME hdf5-${hdf5_ver}.tar.gz
    INSTALL_COMMAND make install
    URL ${hdf5_url}
    DEPENDS szip zlib
)

ExternalProject_add(
    field3d
    PREFIX field3d-${field3d_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CMAKE_ARGS
        -DCMAKE_INSTALL_PREFIX:PATH=${EXTERNAL_ROOT}
        -DCMAKE_INSTALL_RPATH:PATH=${EXTERNAL_ROOT}/lib:::::::::::::::::::::::::::::::::::::::::
        -DCMAKE_INCLUDE_PATH:PATH=${EXTERNAL_ROOT}/include:${prereq_root}/include
        -DCMAKE_LIBRARY_PATH:PATH=${EXTERNAL_ROOT}/lib:${prereq_root}/lib
        -DBoost_INCLUDE_DIR:PATH=${EXTERNAL_ROOT}/include/boost-${boost_ver_major}
        -DBoost_LIBRARY_DIR:PATH=${EXTERNAL_ROOT}/lib
        -DDOXYGEN_EXECUTABLE:FILEPATH=${DOXYGEN_EXECUTABLE}
        -DHDF5_C_LIBRARIES:PATH=${EXTERNAL_ROOT}/lib/libhdf5.so
        -DHDF5_INCLUDE_DIRS:PATH=${EXTERNAL_ROOT}/include
        -DIlmbase_Base_Dir:PATH=${prereq_root}
    INSTALL_COMMAND make install
    GIT_REPOSITORY ${field3d_repo}
    GIT_TAG ${field3d_tag}
    DEPENDS boost ilmbase hdf5
)

ExternalProject_Add_Step(
    field3d install_source
    COMMAND
        git archive --format=tar.gz --prefix=field3d-${field3d_ver}/
            ${field3d_tag} > ../field3d-${field3d_ver}.tar.gz
    DEPENDEES install
    COMMENT "Export versioned snapshot of field3d repository ..."
    WORKING_DIRECTORY
        ${CMAKE_CURRENT_BINARY_DIR}/field3d-${field3d_ver}/src/field3d
)

ExternalProject_add(
    libpng
    PREFIX libpng-${libpng_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND
        PKG_CONFIG_PATH=${EXTERNAL_ROOT}/lib/pkgconfig:${prereq_root}/lib/pkgconfig
        ./configure --prefix=${EXTERNAL_ROOT} --disable-static --with-pic
    DOWNLOAD_COMMAND curl -L -k ${libpng_url} -o libpng-${libpng_ver}.tar.gz
    COMMAND ${CMAKE_COMMAND} -E make_directory libpng
    COMMAND tar xf libpng-${libpng_ver}.tar.gz -C libpng --strip 1
    INSTALL_COMMAND make install
    PATCH_COMMAND
        curl -L -k ${libpng_patch_url} -o ../libpng-${libpng_ver}-apng.patch.gz
    COMMAND gzip -cd ../libpng-${libpng_ver}-apng.patch.gz | patch -p1
    URL ${libpng_url}
)

ExternalProject_add(
    giflib
    PREFIX giflib-${giflib_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND
        PKG_CONFIG_PATH=${EXTERNAL_ROOT}/lib/pkgconfig:${prereq_root}/lib/pkgconfig
        ./configure --prefix=${EXTERNAL_ROOT} --with-pic --disable-static
    DOWNLOAD_COMMAND curl -L -k ${giflib_url} -o giflib-${giflib_ver}.tar.gz
    COMMAND ${CMAKE_COMMAND} -E make_directory giflib
    COMMAND tar xf giflib-${giflib_ver}.tar.gz -C giflib --strip 1
    INSTALL_COMMAND make install
    URL ${giflib_url}
)

configure_file(in/jbigkit_make.in jbigkit_make)
configure_file(in/jbigkit_install.in jbigkit_install)

ExternalProject_add(
    jbigkit
    PREFIX jbigkit-${jbigkit_ver}
    BUILD_COMMAND
        PKG_CONFIG_PATH=${EXTERNAL_ROOT}/lib/pkgconfig:${prereq_root}/lib/pkgconfig
        bash ${CMAKE_CURRENT_BINARY_DIR}/jbigkit_make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND ""
    DOWNLOAD_NAME jbigkit-${jbigkit_ver}.tar.gz
    INSTALL_COMMAND bash ${CMAKE_CURRENT_BINARY_DIR}/jbigkit_install
    PATCH_COMMAND
        patch -p1 < ${CMAKE_SOURCE_DIR}/patches/jbigkit-${jbigkit_ver}.patch
    GIT_REPOSITORY ${jbigkit_repo}
    GIT_TAG ${jbigkit_tag}
)

ExternalProject_Add_Step(
    jbigkit export_source
    COMMAND
        git archive --format=tar.gz --prefix=jbigkit-${jbigkit_ver}/
            ${jbigkit_tag} > ../jbigkit-${jbigkit_ver}.tar.gz
    DEPENDEES install
    COMMENT "Export versioned snapshot of jbigkit repository"
    WORKING_DIRECTORY
        ${CMAKE_CURRENT_BINARY_DIR}/jbigkit-${jbigkit_ver}/src/jbigkit
)

ExternalProject_add(
    ptex
    PREFIX ptex-${ptex_ver}
    BUILD_COMMAND make
    CMAKE_ARGS
        -DCMAKE_INSTALL_LIBDIR:PATH=${EXTERNAL_ROOT}/lib
        -DCMAKE_INSTALL_PREFIX:PATH=${EXTERNAL_ROOT}
        -DDOXYGEN_EXECUTABLE:FILEPATH=${DOXYGEN_EXECUTABLE}
        -DZLIB_ROOT:PATH=${prereq_root}
    INSTALL_COMMAND make install
    COMMAND make doc
    GIT_REPOSITORY ${ptex_repo}
    GIT_TAG ${ptex_tag}
    PATCH_COMMAND
        patch -p1 < ${CMAKE_SOURCE_DIR}/patches/ptex-${ptex_ver}.patch
    DEPENDS zlib
)

ExternalProject_Add_Step(
    ptex install_source
    COMMAND
        git archive --format=tar.gz --prefix=ptex-${ptex_ver}/
            ${ptex_tag} > ../ptex-${ptex_ver}.tar.gz
    DEPENDEES install
    COMMENT "Export versioned snapshot of ptex repository ..."
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/ptex-${ptex_ver}/src/ptex
)

ExternalProject_add(
    libjpegturbo
    PREFIX libjpegturbo-${libjpegturbo_ver}
    BUILD_COMMAND PATH=${prereq_root}/bin:$ENV{PATH} make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND
        CFLAGS=-fPIC
        CXXFLAGS=-fPIC
        PATH=${prereq_root}/bin:$ENV{PATH}
        ./configure --prefix=${prereq_root} --with-pic --disable-shared
            --with-jpeg8
    DOWNLOAD_COMMAND
        curl -L -k ${libjpegturbo_url}
            -o libjpegturbo-${libjpegturbo_ver}.tar.gz
    COMMAND ${CMAKE_COMMAND} -E make_directory libjpegturbo
    COMMAND
        tar xf libjpegturbo-${libjpegturbo_ver}.tar.gz -C libjpegturbo --strip 1
    INSTALL_COMMAND make install
    COMMAND ${CMAKE_COMMAND} -E remove ${prereq_root}/lib/libjpeg.so
        ${prereq_root}/lib/libjpeg.so.62 ${prereq_root}/lib/libjpeg.so.62.2.0
    PATCH_COMMAND autoreconf -fiv
    URL ${libjpegturbo_url}
    DEPENDS nasm
)

configure_file(in/jasper_configure.in jasper_configure @ONLY)

ExternalProject_add(
    jasper
    PREFIX jasper-${jasper_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND
        PKG_CONFIG_PATH=${EXTERNAL_ROOT}/lib/pkgconfig:${prereq_root}/lib/pkgconfig
        bash ${CMAKE_CURRENT_BINARY_DIR}/jasper_configure
    DOWNLOAD_NAME jasper-${jasper_ver}.tar.gz
    INSTALL_COMMAND make install
    URL ${jasper_url}
    DEPENDS libjpegturbo
    #DEPENDS jpeg
)

configure_file(in/libtiff_configure.in libtiff_configure)

ExternalProject_add(
    libtiff
    PREFIX libtiff-${libtiff_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND
        PATH=${prereq_root}/bin:$ENV{PATH}
        PKG_CONFIG_PATH=${EXTERNAL_ROOT}/lib/pkgconfig:${prereq_root}/lib/pkgconfig
        bash ${CMAKE_CURRENT_BINARY_DIR}/libtiff_configure
    INSTALL_COMMAND make install
    URL ${libtiff_url}
    DEPENDS freeglut jbigkit libjpegturbo zlib xz
    #DEPENDS freeglut jbigkit jpeg zlib xz
)

configure_file(in/lcms2_configure.in lcms2_configure)

ExternalProject_add(
    lcms2
    PREFIX lcms2-${lcms2_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND
        PKG_CONFIG_PATH=${EXTERNAL_ROOT}/lib/pkgconfig:${prereq_root}/lib/pkgconfig
        bash ${CMAKE_CURRENT_BINARY_DIR}/lcms2_configure
    GIT_REPOSITORY ${lcms2_repo}
    GIT_TAG ${lcms2_tag}
    INSTALL_COMMAND make install
    DEPENDS libjpegturbo libtiff zlib
    #DEPENDS jpeg libtiff zlib
)

ExternalProject_Add_Step(
    lcms2 install_source
    COMMAND
        git archive --format=tar.gz --prefix=lcms2-${lcms2_ver}/
            ${lcms2_tag} > ../lcms2-${lcms2_ver}.tar.gz
    DEPENDEES install
    COMMENT "Export versioned snapshot of lcms2 repository ..."
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/lcms2-${lcms2_ver}/src/lcms2
)

ExternalProject_add(
    openjpeg
    PREFIX openjpeg-${openjpeg_ver}
    BUILD_COMMAND make
    #BUILD_IN_SOURCE 1
    CMAKE_ARGS
        -DCMAKE_INSTALL_PREFIX:PATH=${EXTERNAL_ROOT}
        -DCMAKE_INSTALL_RPATH:PATH=${EXTERNAL_ROOT}/lib:::::::::::::::::::::::::::::::::::::::::
        -DPNG_LIBRARY:FILEPATH=${EXTERNAL_ROOT}/lib/libpng.so
        -DPNG_PNG_INCLUDE_DIR:PATH=${EXTERNAL_ROOT}/include
        -DTIFF_LIBRARY:FILEPATH=${EXTERNAL_ROOT}/lib/libtiff.so
        -DTIFF_INCLUDE_DIR:PATH=${EXTERNAL_ROOT}/include
        -DZLIB_ROOT:PATH=${prereq_root}
    DEPENDS jasper lcms2 libpng
    INSTALL_COMMAND make install
    GIT_REPOSITORY ${openjpeg_repo}
    GIT_TAG ${openjpeg_tag}
)

ExternalProject_Add_Step(
    openjpeg install_source
    COMMAND
        git archive --format=tar.gz --prefix=openjpeg-${openjpeg_ver}/
            ${openjpeg_tag} > ../openjpeg-${openjpeg_ver}.tar.gz
    DEPENDEES install
    COMMENT "Export versioned snapshot of openjpeg repository ..."
    WORKING_DIRECTORY
        ${CMAKE_CURRENT_BINARY_DIR}/openjpeg-${openjpeg_ver}/src/openjpeg
)

configure_file(in/libraw_configure.in libraw_configure @ONLY)

ExternalProject_add(
    libraw
    PREFIX libraw-${libraw_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND
        PKG_CONFIG_PATH=${EXTERNAL_ROOT}/lib/pkgconfig:${prereq_root}/lib/pkgconfig
        bash ${CMAKE_CURRENT_BINARY_DIR}/libraw_configure
    DOWNLOAD_NAME libraw-${libraw_ver}.tar.gz
    INSTALL_COMMAND make install
    PATCH_COMMAND patch -p1 < ${CMAKE_SOURCE_DIR}/patches/libraw-${libraw_ver}.patch
    URL ${libraw_url}
    DEPENDS jasper lcms2 libjpegturbo
    #DEPENDS jasper jpeg lcms2
)

ExternalProject_add(
    libwebp
    PREFIX libwebp-${libwebp_ver}
    BUILD_COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND
        CPPFLAGS=-I${EXTERNAL_ROOT}/include
        LDFLAGS=-L${EXTERNAL_ROOT}/lib
        PKG_CONFIG_PATH=${EXTERNAL_ROOT}/lib/pkgconfig:${prereq_root}/lib/pkgconfig
        ./configure --prefix=${prereq_root} --with-pic --disable-shared
            --enable-libwebpmux --enable-libwebpdemux  --enable-libwebpdecoder
            --enable-libwebpextras --enable-swap-16bit-csp
    DOWNLOAD_NAME libwebp-${libwebp_ver}.tar.gz
    INSTALL_COMMAND make install
    URL ${libwebp_url}
    DEPENDS freeglut giflib libpng libtiff libjpegturbo
    #DEPENDS freeglut giflib libpng libtiff jpeg
)

ExternalProject_add(
    openssl
    PREFIX openssl-${openssl_ver}
    BUILD_COMMAND make depend
    COMMAND make
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND
        PKG_CONFIG_PATH=${EXTERNAL_ROOT}/lib/pkgconfig:${prereq_root}/lib/pkgconfig
        ./config --prefix=${EXTERNAL_ROOT} shared zlib-dynamic
            --openssldir=${EXTERNAL_ROOT}/openssl
    GIT_REPOSITORY ${openssl_repo}
    GIT_TAG ${openssl_tag}
    INSTALL_COMMAND make install
    URL ${openssl_url}
    DEPENDS zlib
)

ExternalProject_Add_Step(
    openssl install_source
    COMMAND
        git archive --format=tar.gz --prefix=openssl-${openssl_ver}/
            ${openssl_tag} > ../openssl-${openssl_ver}.tar.gz
    DEPENDEES install
    COMMENT "Export versioned snapshot of openssl repository ..."
    WORKING_DIRECTORY
        ${CMAKE_CURRENT_BINARY_DIR}/openssl-${openssl_ver}/src/openssl
)

##  We'll use these lists to update rpath/runpath at install time.
set(runpath_dependencies oiio)
set(runpath_oiio_binaries iconvert idiff igrep iinfo maketx oiiotool)
set(runpath_oiio_libraries libOpenImageIO_Util.so libOpenImageIO.so)
set(runpath_oiio_python_libraries OpenImageIO.so)

set(oiio_dependencies
    boost field3d freeglut glew hdf5 libpng libraw libwebp openexr openjpeg
    openssl ptex
)

if(BUILD_OCIO)
    list(APPEND oiio_dependencies ocio)

    list(APPEND tarballs ${CMAKE_CURRENT_BINARY_DIR}/ocio-${OCIO_VERSION}/src/ocio-${OCIO_VERSION}.tar.gz)
    list(APPEND patches ${CMAKE_SOURCE_DIR}/patches/ocio-${OCIO_VERSION}.patch)
    list(APPEND runpath_dependencies ocio)

    set(runpath_ocio_binaries ociobakelut ociocheck ocioconvert ociodisplay ociolutimage)
    set(runpath_ocio_libraries libOpenColorIO.so)
    set(runpath_ocio_python_libraries PyOpenColorIO.so)

    set(ocio_cmake_args
        -DBOOST_INCLUDE_DIR:PATH=${EXTERNAL_ROOT}/include/boost-${boost_ver_major}
        -DCMAKE_INCLUDE_PATH:PATH=${EXTERNAL_ROOT}/include:${prereq_root}/include
        -DCMAKE_INSTALL_PREFIX:PATH=${BUILD_ROOT}
        -DCMAKE_INSTALL_RPATH:PATH=${BUILD_ROOT}/lib:${EXTERNAL_ROOT}/lib
        -DCMAKE_LIBRARY_PATH:PATH=${EXTERNAL_ROOT}/lib:${prereq_root}/lib
        -DGLEW_INCLUDES:PATH=${EXTERNAL_ROOT}/include
        -DGLEW_LIBRARIES:FILEPATH=${EXTERNAL_ROOT}/lib/libGLEW.so
        -DGLUT_glut_LIBRARY:FILEPATH=${EXTERNAL_ROOT}/lib/libglut.so
        -DGLUT_INCLUDE_DIR:PATH=${EXTERNAL_ROOT}/include
        -DFIELD3D_LIBRARY:FILEPATH=${EXTERNAL_ROOT}/lib/libField3D.so
        -DLCMS2_INCLUDE_DIR:PATH=${EXTERNAL_ROOT}/include
        -DLCMS2_LIBRARY:FILEPATH=${EXTERNAL_ROOT}/lib/liblcms2.a
        -DOCIO_BUILD_STATIC:BOOL=OFF
        -DOCIO_BUILD_DOCS:BOOL=OFF
        -DOCIO_NAMESPACE:STRING=OpenColorIO_${OIIO_LIBRARY_SUFFIX}
        -DPYTHON_INCLUDE_DIR=${PYTHON_INCLUDE_DIR}
        -DPYTHON_LIBRARY=${PYTHON_LIBRARY}
        -DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE}
    )

    ExternalProject_add(
        ocio
        PREFIX ocio-${OCIO_VERSION}
        CMAKE_ARGS ${ocio_cmake_args}
        GIT_REPOSITORY ${ocio_repo}
        GIT_TAG ${ocio_tag}
        INSTALL_COMMAND make install
        PATCH_COMMAND
            patch -p1 < ${CMAKE_SOURCE_DIR}/patches/ocio-${OCIO_VERSION}.patch
        DEPENDS boost freeglut glew hdf5 ilmbase lcms2 openexr libwebp
    )

    ExternalProject_Add_Step(
        ocio export_source
        COMMAND
            git archive --format=tar.gz --prefix=ocio-${ocio_VERSION}/
                ${ocio_tag} > ../ocio-${OCIO_VERSION}.tar.gz
        DEPENDEES install
        COMMENT "Export versioned snapshot of ocio repository"
        WORKING_DIRECTORY
            ${CMAKE_CURRENT_BINARY_DIR}/ocio-${OCIO_VERSION}/src/ocio
    )
endif()

set(oiio_cmake_args
    # OIIO build implicit variables ...
    -DCMAKE_INCLUDE_PATH:PATH=${prereq_root}/include:${BUILD_ROOT}/include
    -DCMAKE_INSTALL_PREFIX:PATH=${BUILD_ROOT}
    -DCMAKE_LIBRARY_PATH:PATH=${prereq_root}/lib:${BUILD_ROOT}/lib
    #-DLINKSTATIC:BOOL=ON
    -DOIIO_BUILD_TESTS:BOOL=ON
    -DOIIO_NAMESPACE:STRING=OpenImageIO_${OIIO_LIBRARY_SUFFIX}
    -DSTOP_ON_WARNING:BOOL=OFF
    -DUSE_OPENSSL:BOOL=ON
    -DVERBOSE:BOOL=ON
    # Added by Geoff!
    -DOIIO_EXTERNAL:PATH=${EXTERNAL_ROOT}
    -DPYTHON_VERSION:STRING=${python_ver}
    # Library-based overrides.
    -DBOOST_ROOT:PATH=${EXTERNAL_ROOT}
    -DBOOST_INCLUDEDIR:PATH=${EXTERNAL_ROOT}/include/boost-${boost_ver_major}
    -DFIELD3D_HOME:PATH=${EXTERNAL_ROOT}
    -DGIF_LIBRARY:FILEPATH=${EXTERNAL_ROOT}/lib/libgif.so
    -DGIF_INCLUDE_DIR:PATH=${EXTERNAL_ROOT}/include
    -DILMBASE_HOME:PATH=${prereq_root}
    -DJPEG_LIBRARY:FILEPATH=${prereq_root}/lib/libjpeg.a
    -DJPEG_INCLUDE_DIR:PATH=${prereq_root}/include
    -DJPEGTURBO_PATH:PATH=${prereq_root}
    -DLIBRAW_PATH:PATH=${EXTERNAL_ROOT}
    #-DOpenCV_DIR:PATH=${prereq_root}
    -DOPENEXR_HOME:PATH=${prereq_root}
    -DOPENJPEG_HOME:PATH=${EXTERNAL_ROOT}
    -DOPENJPEG_INCLUDE_DIR:PATH=${EXTERNAL_ROOT}/include/openjpeg-${openjpeg_ver_major}
    -DOPENSSL_ROOT_DIR=${EXTERNAL_ROOT}
    -DPNG_LIBRARY:FILEPATH=${EXTERNAL_ROOT}/lib/libpng.so
    -DPNG_PNG_INCLUDE_DIR:PATH=${EXTERNAL_ROOT}/include
    -DPTEX_INCLUDE_DIR:PATH=${EXTERNAL_ROOT}/include
    -DPTEX_LIBRARY:FILEPATH=${EXTERNAL_ROOT}/lib/libPtex.a
    -DPYTHON_INCLUDE_DIR=${PYTHON_INCLUDE_DIR}
    -DPYTHON_LIBRARY=${PYTHON_LIBRARY}
    -DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE}
    -DTHIRD_PARTY_TOOLS_HOME:PATH=${EXTERNAL_ROOT}
    -DTIFF_INCLUDE_DIR=${EXTERNAL_ROOT}/include
    -DTIFF_LIBRARY=${EXTERNAL_ROOT}/lib/libtiff.so
    -DWEBP_INCLUDE_DIR:PATH=${prereq_root}/include
    -DWEBP_LIBRARY:FILEPATH=${prereq_root}/lib/libwebp.a
    -DZLIB_ROOT:PATH=${prereq_root}
)

if(USE_QT4)
    list(APPEND runpath_oiio_binaries iv)
    list(APPEND oiio_cmake_args
        -DQT_QMAKE_EXECUTABLE:FILEPATH=${QT_QMAKE_EXECUTABLE}
        -DUSE_QT:BOOL=ON
    )
else()
    list(APPEND oiio_cmake_args -DUSE_QT:BOOL=OFF)
endif()

if(OCIO_PATH AND NOT BUILD_OCIO)
    message("OCIO_PATH DEFINED: ${OCIO_PATH}")
    list(APPEND oiio_cmake_args
        -DOCIO_INCLUDE_PATH:PATH=${OCIO_PATH}/include
        -DOCIO_LIBRARY_PATH:PATH=${OCIO_PATH}/lib
    )
elseif(BUILD_OCIO)
    list(APPEND oiio_cmake_args
        -DOCIO_INCLUDE_PATH:PATH=${BUILD_ROOT}/include
        -DOCIO_LIBRARY_PATH:PATH=${BUILD_ROOT}/lib
    )
endif()

if(USE_NUKE)
    set(runpath_nuke_plugins txReader.so txWriter.so)
    list(APPEND oiio_cmake_args
        -DNuke_ROOT:PATH=${Nuke_ROOT}
        -DUSE_NUKE:BOOL=ON
    )
else()
    list(APPEND oiio_cmake_args -DUSE_NUKE=${USE_NUKE})
endif()

ExternalProject_add(
    oiio
    PREFIX oiio-${OIIO_VERSION}
    CMAKE_ARGS ${oiio_cmake_args}
    GIT_REPOSITORY ${oiio_repo}
    GIT_TAG ${oiio_tag}
    INSTALL_COMMAND make install
    PATCH_COMMAND
        patch -p1 < ${CMAKE_SOURCE_DIR}/patches/oiio-${OIIO_VERSION}.patch
    DEPENDS patchelf ${oiio_dependencies}
)

ExternalProject_Add_Step(
    oiio export_source
    COMMAND
        git archive --format=tar.gz --prefix=oiio-${OIIO_VERSION}/
            ${oiio_tag} > ../oiio-${OIIO_VERSION}.tar.gz
    DEPENDEES install
    COMMENT "Export versioned snapshot of oiio repository"
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/oiio-${OIIO_VERSION}/src/oiio
)

ExternalProject_Add_Step(
    oiio make_documentation
    COMMAND make doxygen
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        ${CMAKE_CURRENT_BINARY_DIR}/oiio-${OIIO_VERSION}/src/oiio/doc
        ${BUILD_ROOT}/doc
    DEPENDEES install
    COMMENT "Make OpenImageIO documentation"
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/oiio-${OIIO_VERSION}/src/oiio
)

if(BUILD_OCIO)
    ExternalProject_Add_Step(
        oiio rebuild_ocio_with_oiio
        COMMAND ${CMAKE_COMMAND} ../ocio ${ocio_cmake_args}
        COMMAND make install
        DEPENDEES install
        COMMENT "Rebuilding ocio against ocio!"
        WORKING_DIRECTORY
            ${CMAKE_CURRENT_BINARY_DIR}/ocio-${OCIO_VERSION}/src/ocio-build
    )
endif()

##  Localizing Qt libraries (if necessary) ...
if(QT4_FOUND)
    foreach(_library ${qt_libraries})
        get_filename_component(_ext ${_library} EXT)
        if("${_ext}" STREQUAL ".so")
            get_filename_component(_filename ${_library} NAME)
            ExternalProject_Add_Step(
                oiio copy_${_filename}
                COMMAND ${CMAKE_COMMAND} -E copy ${_library} ${_filename}
                COMMAND
                    ${CMAKE_COMMAND} -E create_symlink ${_filename}
                        ${_filename}.${QT_VERSION_MAJOR}
                COMMENT "Copying ${_library} to Alembic installation ..."
                DEPENDEES install
                WORKING_DIRECTORY ${EXTERNAL_ROOT}/lib
            )
        endif()
    endforeach(_library)
endif()

install(
    DIRECTORY ${BUILD_ROOT}/
    DESTINATION .
    USE_SOURCE_PERMISSIONS
    PATTERN "${EXTERNAL_DIR}" EXCLUDE
)
install(
    DIRECTORY ${EXTERNAL_ROOT}/lib/
    DESTINATION ${EXTERNAL_DIR}
    USE_SOURCE_PERMISSIONS
    FILES_MATCHING PATTERN "*.so*"
    PATTERN "pkgconfig" EXCLUDE
    PATTERN "openjpeg-*" EXCLUDE
)

set(patchelf_binary ${prereq_root}/bin/patchelf)

configure_file(in/update_runpaths.cmake.in update_runpaths.cmake @ONLY)
configure_file(in/clean_external_libs.bash.in clean_external_libs.bash @ONLY)
install(SCRIPT ${CMAKE_CURRENT_BINARY_DIR}/update_runpaths.cmake)

set(executables "")
if(QT4_FOUND)
    list(APPEND executables iv)
endif()

if(WRAP_EXECUTABLES)
    list(APPEND executables
        iconvert idiff igrep iinfo maketx ociobakelut ociocheck ocioconvert
        ociodisplay ociolutimage oiiotool
    )

    list(LENGTH executables _nexec)
    if(_nexec GREATER 0)
        list(SORT executables)
        message("WRAPPING EXECUTABLES: ${executables}")
        set(wrapper_path "${CMAKE_CURRENT_BINARY_DIR}/wrappers")
        file(MAKE_DIRECTORY ${CURRENT_BINARY_DIR}/wrappers)
        foreach(_executable ${executables})
            configure_file(
                in/wrap_executable.cmake.in wrap_${_executable}.cmake @ONLY)
            configure_file(
                in/generic_wrapper.in ${wrapper_path}/${_executable} @ONLY)
            install(SCRIPT ${CMAKE_CURRENT_BINARY_DIR}/wrap_${_executable}.cmake)
        endforeach(_executable)
    endif()
endif()

if(INSTALL_SOURCE)
    if(${default_source_root})
        install(FILES ${tarballs} DESTINATION ${SOURCE_DIR})
    else()
        install(FILES ${tarballs} DESTINATION ${SOURCE_ROOT})
    endif()
endif()

set(package_name "OIIO")
if(${BUILD_OCIO})
    set(package_name "OIIO-OCIO")
endif()

set(CPACK_PACKAGE_NAME ${package_name})
set(CPACK_PACKAGE_VENDOR "Hoafus Arabius Enterprises")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "OIIO ${OIIO_VERSION} packaged for easy installation.")
set(CPACK_PACKAGE_VERSION "${OIIO_VERSION}")
set(CPACK_PACKAGE_VERSION_MAJOR "${OIIO_MAJOR_VERSION}")
set(CPACK_PACKAGE_VERSION_MINOR "${OIIO_MINOR_VERSION}")
set(CPACK_PACKAGE_VERSION_PATCH "${OIIO_PATCH_VERSION}")
set(CPACK_PACKAGE_INSTALL_DIRECTORY "payload")
set(CPACK_GENERATOR STGZ)

include(CPack)


# Caffe as Dynamic Library (Visual Studio 2019 + CUDA 11 + cuDNN 8)

1. [What To Expect](#what-to-expect)
2. [Required Steps](#required-steps)
3. [Edits in Current Code (Optional Section)](#edits-in-current-code-optional-section)



## What To Expect
This code is only meant to generate the DLL/lib/include files that OpenPose needs on Windows. We might do changes or remove functionality in order to make it compile or to make it simpler.
- Disabled dependencies: Level DB, OpenCV, and Python dependencies.
- Deconvolution layer was commented out to avoid compiler errors on cuDNN 8.



## Required Steps
### Prerequisites
Rename path as short as possible (sometimes error for paths too long). Alternative: Disable 255-character limit of Windows (e.g., installing the latest Python.)

### Compiling Caffe
1. Run `.\scripts\build_win.cmd` (do not run it from inside the scripts folder).
2. If we wanted OpenCV (we don't):
    1. Stop it after the pre-requisites have been downloaded, and remove the `caffe\build` folder.
    2. Replace:
        1. `caffe3rdparty\dependencies\libraries_v140_x64_py27_1.1.0\libraries\x64` by desired OpenCV `x64`.
        2. `caffe3rdparty\dependencies\libraries_v140_x64_py27_1.1.0\libraries\include\opencv*` by desired OpenCV `include`.



### Extra Steps for VS 2019
1. After initial error:
    1. Manually substitute the new Boost (and optionally OpenCV) for the old ones.
        1. From `caffe3rdparty\dependencies\libraries_v140_x64_py35_1.1.0\libraries/include`
            1. Boost: Remove `include/boost_1.61/` and simply paste the new Boost as `include/boost-1_61/boost`.
            2. OpenCV: Simply remove it (or optionally add your desired version).
        2. Boost: From `caffe3rdparty\dependencies\libraries_v140_x64_py35_1.1.0\libraries/lib`, update Boost lib/dlls.
        3. Remove `caffe3rdparty\dependencies\libraries_v140_x64_py35_1.1.0\libraries/x64` (or optionally add your desired OpenCV version).
    2. Only if USE_LEVELDB enabled: `caffe3rdparty\dependencies\libraries_v140_x64_py35_1.1.0\libraries\cmake\leveldb-targets-release.cmake`: Replace
```
  IMPORTED_LINK_INTERFACE_LIBRARIES_RELEASE "${CMAKE_CURRENT_LIST_DIR}/../lib/boost_date_time-vc140-mt-1_61.lib;${CMAKE_CURRENT_LIST_DIR}/../lib/boost_filesystem-vc140-mt-1_61.lib;${CMAKE_CURRENT_LIST_DIR}/../lib/boost_system-vc140-mt-1_61.lib"
```
    by:
```
  IMPORTED_LINK_INTERFACE_LIBRARIES_RELEASE "${CMAKE_CURRENT_LIST_DIR}/../lib/boost_date_time-vc142-mt-x64-1_74.lib;${CMAKE_CURRENT_LIST_DIR}/../lib/boost_filesystem-vc142-mt-x64-1_74.lib;${CMAKE_CURRENT_LIST_DIR}/../lib/boost_system-vc142-mt-x64-1_74.lib"
```
    3. Only if USE_LEVELDB enabled: `caffe3rdparty\dependencies\libraries_v140_x64_py35_1.1.0\libraries\cmake\leveldb-targets-debug.cmake`: Replace
```
  IMPORTED_LINK_INTERFACE_LIBRARIES_DEBUG "${CMAKE_CURRENT_LIST_DIR}/../lib/boost_date_time-vc140-mt-gd-1_61.lib;${CMAKE_CURRENT_LIST_DIR}/../lib/boost_filesystem-vc140-mt-gd-1_61.lib;${CMAKE_CURRENT_LIST_DIR}/../lib/boost_system-vc140-mt-gd-1_61.lib"
```
    by:
```
  IMPORTED_LINK_INTERFACE_LIBRARIES_DEBUG "${CMAKE_CURRENT_LIST_DIR}/../lib/boost_date_time-vc142-mt-gd-x64-1_74.lib;${CMAKE_CURRENT_LIST_DIR}/../lib/boost_filesystem-vc142-mt-gd-x64-1_74.lib;${CMAKE_CURRENT_LIST_DIR}/../lib/boost_system-vc142-mt-gd-x64-1_74.lib"
```
    4. Only if OpenCV added: `caffe3rdparty\dependencies\libraries_v140_x64_py35_1.1.0\libraries\OpenCVConfig.cmake`: Inside `if(MSVC_VERSION EQUAL 1400)`, add:
```
  else(MSVC_VERSION EQUAL 1400) #if(MSVC_VERSION EQUAL 1914)
      set(OpenCV_RUNTIME vc15)
```
2. Remove build, re-run `.\scripts\build_win.cmd`. After new error:
    1. Modify `caffe\build\include\caffe\proto\caffe.pb.*`:
        1. Comment out the 2 lines that start with `static const DimCheckMode STRICT` (*.pb.h).
        2. Comment out the 2 lines that contain `::STRICT` (*.pb.cc).
3. Some final error about copy will occur. Simply re-run `build_win.cmd` and it will work.



### Moving Files
1. Copy `include\` as `include`.
2. Copy `build\install\bin` as `bin` (or `build\install\python\caffe`).
    1. Add `cudnn64_8.dll`.
3. Copy `build\caffe` & `build\include\caffe` as `include2\caffe`.
4. Copy from `build\lib\`: `caffe.lib` & `caffeproto.lib` as `lib`.
5. caffe3rdparty = caffe\..\caffe3rdparty\dependencies\libraries_v140_x64_py27_1.1.0:
    1. Copy `$caffe3rdparty\include` as `caffe3rdparty\include` (remove OpenCV/LevelDB folders).
    2. Copy `$caffe3rdparty\lib` as `caffe3rdparty\lib` (remove DLLs and unnecessary libs like OpenCV/LevelDB and all `lib*.lib`/`*static*.lib` files).
    3. Move `boost` out of `boost-1_61`.
6. Repeat the whole process with `Release` --> `Debug` in `build_win.cmd`.





## Edits in Current Code (Optional Section)
These are the changes that the current code already have. This is just for your information, out of curiosity.

### Edits Already Done To Original Code
1. Edit `caffe\CMake\Dependencies.cmake`:
```
# if(USE_OPENCV)
#   find_package(OpenCV QUIET COMPONENTS core highgui imgproc imgcodecs)
#   if(NOT OpenCV_FOUND) # if not OpenCV 3.x, then imgcodecs are not found
#     find_package(OpenCV REQUIRED COMPONENTS core highgui imgproc)
#   endif()
#   list(APPEND Caffe_INCLUDE_DIRS PUBLIC ${OpenCV_INCLUDE_DIRS})
#   list(APPEND Caffe_LINKER_LIBS PUBLIC ${OpenCV_LIBS})
#   message(STATUS "OpenCV found (${OpenCV_CONFIG_PATH})")
#   list(APPEND Caffe_DEFINITIONS PUBLIC -DUSE_OPENCV)
# endif()
# find_package(OpenCV COMPONENTS world)
# if(NOT OpenCV_FOUND)
#   message(FATAL_ERROR "OpenCV not found!")
# endif()
# message(STATUS "OpenCV found (${OpenCV_CONFIG_PATH})")
# message(STATUS "  OpenCV_INCLUDE_DIRS = ${OpenCV_INCLUDE_DIRS}")
# message(STATUS "  OpenCV_LIBS = ${OpenCV_LIBS}")
```
2. Edit scripts\build_win.cmd
    1. Add on line 3:
```
set CUDA_ARCH_NAME=All
set CMAKE_BUILD_SHARED_LIBS=1
```
3. Edit cmake\Cuda.cmake
    1. Edit line 7:
```
# set(Caffe_known_gpu_archs "20 21(20) 30 35 50 60 61")
# set(Caffe_known_gpu_archs "20 21(20) 30 35 50 52 60 61")
set(Caffe_known_gpu_archs "30 35 37 50 52 53 60 61 62 70 72 75")
```
    2. Comment lines 64-68:
```
#   set(__archs_name_default "All")
#   if(NOT CMAKE_CROSSCOMPILING)
#     list(APPEND __archs_names "Auto")
#     set(__archs_name_default "Auto")
#   endif()
```
    3. Comment lines 75-79:
```
#   # verify CUDA_ARCH_NAME value
#   if(NOT ";${__archs_names};" MATCHES ";${CUDA_ARCH_NAME};")
#     string(REPLACE ";" ", " __archs_names "${__archs_names}")
#     message(FATAL_ERROR "Only ${__archs_names} architeture names are supported.")
#   endif()
```
4. Edit cmake\WindowsDownloadPrebuiltDependencies.cmake
    1. Edit line 21:
```
#   set(CAFFE_DEPENDENCIES_ROOT_DIR ${USERPROFILE_DIR}/.caffe/dependencies CACHE PATH "Prebuild depdendencies root directory")
  set(CAFFE_DEPENDENCIES_ROOT_DIR ./0_temporary_downloaded/dependencies CACHE PATH "Prebuild depdendencies root directory")
```

### Additional Edits for VS 2019
1. Edit `scripts\build_win.cmd`:
    1. MSVC_VERSION=14 --> 16 (x2)
    2. BUILD_PYTHON=1 --> 0 (x2)
    3. BUILD_PYTHON_LAYER=1 --> 0 (x2)
    4. After `if %WITH_NINJA% EQU 0 (`, add:
```
    if "%MSVC_VERSION%"=="16" (
        set CMAKE_GENERATOR=Visual Studio 16 2019
    )
    if "%MSVC_VERSION%"=="15" (
        set CMAKE_GENERATOR=Visual Studio 15 2017 Win64
    )
```
    5. Replace `set batch_file=!VS%MSVC_VERSION%0COMNTOOLS!..\..\VC\vcvarsall.bat` by:
```
:: set batch_file=!VS%MSVC_VERSION%0COMNTOOLS!..\..\VC\vcvarsall.bat
set batch_file=D:\Programs\Microsoft Visual Studio\2019\Enterprise\VC\Auxiliary\Build\vcvars64.bat
echo batch_file = !batch_file!
```
2. Edit `caffe\cmake\Dependencies.cmake`: In `find_package(Boost 1.54 REQUIRED COMPONENTS system thread filesystem)`, replace by 1.74 (or used version):
```
find_package(Boost 1.74.0 REQUIRED COMPONENTS system thread filesystem date_time)
# find_package(Boost 1.54 REQUIRED COMPONENTS system thread filesystem)
```
3. Edit `caffe\cmake\WindowsDownloadPrebuiltDependencies.cmake`: Manually replace `${MSVC_VERSION}` by `1900` and `${_pyver}` by `35`.
4. Edit `caffe\cmake\TargetResolvePrerequesites.cmake`: Comment out `get_filename_component(_dir ${OpenCV_LIB_PATH} DIRECTORY)` and (next line) `list(APPEND _directories ${_dir}/bin)`.
5. Edit `caffe\CMakeLists.txt`: caffe_option(USE_OPENCV "Build with OpenCV support" ON) --> OFF

### Additional Edits for CUDA 11 / cuDNN 8
1. Edit `cmake\Cuda.cmake`:
    1. Edit line 7:
```
# Known NVIDIA GPU achitectures Caffe can be compiled for.
# This list will be used for CUDA_ARCH_NAME = All option
# set(Caffe_known_gpu_archs "30 35 50 52 60 61")
# set(Caffe_known_gpu_archs "20 21(20) 30 35 50 60 61")
# Fermi (3.2 <= CUDA <= 8)
# set(FERMI "20 21(20)")
# Kepler (CUDA >= 5)
set(KEPLER "35 37") # set(KEPLER "30 35 37") # This crashes with CUDA 10
# Maxwell (CUDA >= 6)
set(MAXWELL "50 52 53")
# Pascal (CUDA >= 8)
set(PASCAL "60 61 62")
# Volta (CUDA >= 9)
set(VOLTA "70 72") # set(VOLTA "70 71 72") # This crashes with CUDA 10
# Turing (CUDA >= 10)
set(TURING "75")
# Ampere (CUDA >= 11)
set(AMPERE "80 86")
if (UNIX AND NOT APPLE)
  set(Caffe_known_gpu_archs "${KEPLER} ${MAXWELL} ${PASCAL} ${VOLTA} ${TURING} ${AMPERE}")
  # set(Caffe_known_gpu_archs "${FERMI} ${KEPLER} ${MAXWELL} ${PASCAL} ${VOLTA} ${TURING}")
  # set(Caffe_known_gpu_archs "20 21(20) 30 35 50 52 60 61")
elseif (WIN32)
  set(Caffe_known_gpu_archs "${KEPLER} ${MAXWELL} ${PASCAL} ${VOLTA} ${TURING} ${AMPERE}")
endif ()
```
2. Edit `caffe\src\caffe\layers\cudnn_conv_layer.cpp`:
    - Just copy the new one from the default OpenPose caffe.
3. Edit `caffe\cmake\Dependencies.cmake`, and comment out USE_LEVELDB:
```
# ---[ LevelDB
if(USE_LEVELDB)
  find_package(LevelDB REQUIRED)
  list(APPEND Caffe_INCLUDE_DIRS PUBLIC ${LevelDB_INCLUDES})
  list(APPEND Caffe_LINKER_LIBS PUBLIC ${LevelDB_LIBRARIES})
  list(APPEND Caffe_DEFINITIONS PUBLIC -DUSE_LEVELDB)
endif()

# ---[ Snappy
if(USE_LEVELDB)
  find_package(Snappy REQUIRED)
  list(APPEND Caffe_INCLUDE_DIRS PRIVATE ${Snappy_INCLUDE_DIR})
  list(APPEND Caffe_LINKER_LIBS PRIVATE ${Snappy_LIBRARIES})
endif()
```
by
```
# # ---[ LevelDB
# if(USE_LEVELDB)
#   find_package(LevelDB REQUIRED)
#   list(APPEND Caffe_INCLUDE_DIRS PUBLIC ${LevelDB_INCLUDES})
#   list(APPEND Caffe_LINKER_LIBS PUBLIC ${LevelDB_LIBRARIES})
#   list(APPEND Caffe_DEFINITIONS PUBLIC -DUSE_LEVELDB)
# endif()

# # ---[ Snappy
# if(USE_LEVELDB)
#   find_package(Snappy REQUIRED)
#   list(APPEND Caffe_INCLUDE_DIRS PRIVATE ${Snappy_INCLUDE_DIR})
#   list(APPEND Caffe_LINKER_LIBS PRIVATE ${Snappy_LIBRARIES})
# endif()
```

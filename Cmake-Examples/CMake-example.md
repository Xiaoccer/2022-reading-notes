[toc]

# CMake-example

## 生成可执行文件

* add_executable(hello_cmake main.cpp)，第一参数是可执行文件名字，第二个参数是编译源文件（.cpp）

  ```
  cmake_minimum_required(VERSION 2.6)
  project (hello_cmake)
  add_executable(${PROJECT_NAME} main.cpp)
  ```

* 一般步骤

  ```
  $ mkdir build
  
  $ cd build
  
  $ cmake ..
  
  $ make / make VERBOSE=1 #（输出详细信息）
  
  $ make clean
  ```

## 创建变量

```
# Create a sources variable with a link to all cpp files to compile
set(SOURCES
    src/Hello.cpp
    src/main.cpp
)

add_executable(${PROJECT_NAME} ${SOURCES})
```

* 还可以用GLOB命令来获取源文件，**推荐用法**

```
file(GLOB SOURCES "src/*.cpp")
```

## 包含头文件

* add these directories to the compiler with the -I flag e.g. `-I/directory/path`

```
target_include_directories(target
    PRIVATE
        ${PROJECT_SOURCE_DIR}/include
)
```

## 静态库

* This will be used to create a static library with the name **libhello_library.a** with the sources in the add_library call.
* STATIC关键字
* 注意hello_library和hello_binary

```
# 生成hello_library
add_library(hello_library STATIC
    src/Hello.cpp
)

# 生成hello_binary
add_executable(hello_binary
    src/main.cpp
)

# 将hello_library链接到hello_binary
target_link_libraries(hello_binary
    PRIVATE
        hello_library
)

# 相当于
/usr/bin/c++ CMakeFiles/hello_binary.dir/src/main.cpp.o -o hello_binary -rdynamic libhello_library.a
```

## SCOPES

- PRIVATE - the directory is added to this target’s include directories
- INTERFACE - the directory is added to the include directories for any targets that link this library.
- PUBLIC - As above, it is included in this library and also **any targets that link this library.**

```
# 传播目录。
# 注意这里写的是hello_library，因为是public关键字，所以当链接这个library的所有target（如hello_binary），都会同样包含该目录。
target_include_directories(hello_library
    PUBLIC
        ${PROJECT_SOURCE_DIR}/include
)
```

* 为防止头文件名冲突，应该为不同模块的头文件各建一个目录，使用时如下：

```
#include "static/Hello.h"
```

## 动态库

```
# 生成动态库 .so文件
add_library(hello_library SHARED
    src/Hello.cpp
)

# 别名使用场景？？
add_library(hello::library ALIAS hello_library)

# 生成可执行文件
add_executable(hello_binary
    src/main.cpp
)

# 链接动态库
target_link_libraries(hello_binary
    PRIVATE
        hello::library
)

# 相当于
/usr/bin/c++ CMakeFiles/hello_binary.dir/src/main.cpp.o -o hello_binary -rdynamic libhello_library.so -Wl,-rpath,/home/matrim/workspace/cmake-examples/01-basic/D-shared-library/build
```

## 安装

* 将执行文件安装到系统中

```
# 安装执行文件
install (TARGETS cmake_examples_inst_bin
    DESTINATION bin)
    
# 安装库
install (TARGETS cmake_examples_inst
    LIBRARY DESTINATION lib)

# 安装目录
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/
    DESTINATION include)
    
# 安装配置
install (FILES cmake-examples.conf
    DESTINATION etc)
    
## 默认安装路径
${CMAKE_INSTALL_PREFIX} = /usr/local/

## 修改安装路径
make install DESTDIR=/tmp/stage = ${DESTDIR}/${CMAKE_INSTALL_PREFIX}

if( CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT )
  message(STATUS "Setting default CMAKE_INSTALL_PREFIX path to ${CMAKE_BINARY_DIR}/install")
  set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/install" CACHE STRING "The path to use for make install" FORCE)
endif()

## 取消安装
sudo xargs rm < install_manifest.txt

```

## 构建类型

Cmake内置类型

- Release - Adds the `-O3 -DNDEBUG` flags to the compiler
- Debug - Adds the `-g` flag
- MinSizeRel - Adds `-Os -DNDEBUG`
- RelWithDebInfo - Adds `-O2 -g -DNDEBUG` flags

使用

```
cmake .. -DCMAKE_BUILD_TYPE=Release
```

设置默认类型

```
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message("Setting build type to 'RelWithDebInfo' as none was specified.")
  set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
    "MinSizeRel" "RelWithDebInfo")
endif()
```

## 编译Flags

两种方式：

- using target_compile_definitions() function.(Per-Target Flags)（推荐）
- using the CMAKE_C_FLAGS and CMAKE_CXX_FLAGS variables.( globally for all targets)

```
#Per-Target Flags

# This will cause the compiler to add the definition -DEX3 when compiling the target.
target_compile_definitions(cmake_examples_compile_flags
    PRIVATE EX3
)

```

```
# Default Flags

# C++
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DEX2" CACHE STRING "Set C++ Compiler Flags" FORCE)

# C
Setting C compiler flags using CMAKE_C_FLAGS

# Linker
Setting linker flags using CMAKE_LINKER_FLAGS

# 或者
cmake .. -DCMAKE_CXX_FLAGS="-DEX3"
```

The values `CACHE STRING "Set C++ Compiler Flags" FORCE` from the above command are used to force this variable to be set in the CMakeCache.txt file

## 第三方库

从CMAKE_MODULE_PATH查找库

```
# find a boost install with the libraries filesystem and system
find_package(Boost 1.46.1 REQUIRED COMPONENTS filesystem system)
```

The arguments are:

- Boost - Name of the library. This is part of used to find the module file FindBoost.cmake
- 1.46.1 - The minimum version of boost to find
- REQUIRED - Tells the module that this is required and to fail if it cannot be found
- COMPONENTS - The list of components to find in the library.

检查是否找到：XXX_FOUND变量

```
if(Boost_FOUND)
    message ("boost found")
    include_directories(${Boost_INCLUDE_DIRS})
else()
    message (FATAL_ERROR "Cannot find Boost")
endif()
```

链接方式：

* 第一种，cmake3.5以上，通过Alias来导入

```
  # 导入filesystem会自动添加boost和system
  target_link_libraries( third_party_include
      PRIVATE
          Boost::filesystem
  )
  
Boost::boost for header only libraries

Boost::system for the boost system library.

Boost::filesystem for filesystem library.
```

* 第二种，不用Alias：
  * xxx_INCLUDE_DIRS - A variable pointing to the include directory for the library.
  * xxx_LIBRARY - A variable pointing to the library path.

```
# Include the boost headers
target_include_directories( third_party_include
    PRIVATE ${Boost_INCLUDE_DIRS}
)

# link against the boost libraries
target_link_libraries( third_party_include
    PRIVATE
    ${Boost_SYSTEM_LIBRARY}
    ${Boost_FILESYSTEM_LIBRARY}
)
```

## 编译器选择-Clang

Flags:

- CMAKE_C_COMPILER - The program used to compile c code.
- CMAKE_CXX_COMPILER - The program used to compile c++ code.
- CMAKE_LINKER - The program used to link your binary.

设置：

```
cmake .. -DCMAKE_C_COMPILER=clang-3.6 -DCMAKE_CXX_COMPILER=clang++-3.6
```

## 构建选择-ninja

CMake [generators](https://cmake.org/cmake/help/v3.0/manual/cmake-generators.7.html) are responsible for writing the input files (e.g. Makefiles) for the underlying build system。用cmake --help来查看支持的generators。

设置：

```
cmake .. -G Ninja

ninja -v 构建
```

## C++版本选择（通用方法）

Cmake支持在CHECK_CXX_COMPILER_FLAG函数中，通过传入的flag来编译一个程序，然后将结果存在传入的变量中，如（COMPILER_SUPPORTS_CXX11），可利用该函数选择C++版本。

```
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)

```

The line include(CheckCXXCompilerFlag) tells CMake to include this function to make it available for use.

根据不同编译结果，设置不同C++版本（设置CMAKE_CXX_FLAGS）

```
# check results and add flag
if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)#
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()
```

## C++版本选择（CMake v3.1）

设置[CMAKE_CXX_STANDARD](https://cmake.org/cmake/help/v3.1/variable/CMAKE_CXX_STANDARD.html) 

```
 # set the C++ standard to C++ 11
 set(CMAKE_CXX_STANDARD 11)
```

The `CMAKE_CXX_STANDARD` variable falls back to the closest appropriate standard without a failure. For example, if you request `-std=gnu11` you may end up with `-std=gnu0x`.

This can result in an unexpected failure at compile time.

## C++版本选择（CMake v3.1）

用[target_compile_features](https://cmake.org/cmake/help/v3.1/command/target_compile_features.html) function

```
# set the C++ standard to the appropriate standard for using auto
target_compile_features(hello_cpp11 PUBLIC cxx_auto_type)

message("List of compile features: ${CMAKE_CXX_COMPILE_FEATURES}")
```

## 多模块

当用project()函数时，自带``${PROJECT_NAME} PROJECT_SOURCE_DIR PROJECT_BINARY_DIR`` 

* 最上层的CMakelists.txt

```
project(subprojects)

add_subdirectory(sublibrary1)
add_subdirectory(sublibrary2)
add_subdirectory(subbinary)

# ${sublibrary1_SOURCE_DIR}
# ${sublibrary2_SOURCE_DIR}
```

* sublibrary1:
  * 设置了public的target_include_directories，说明链接该库时，会包含该目录。

```
# Set the project name
project (sublibrary1)

# Add a library with the above sources
add_library(${PROJECT_NAME} src/sublib1.cpp)
add_library(sub::lib1 ALIAS ${PROJECT_NAME})

target_include_directories( ${PROJECT_NAME}
    PUBLIC ${PROJECT_SOURCE_DIR}/include
)
```

* sublibrary2
  * 若只包含头文件，可用INTERFACE关键字，省去build。
  * The INTERFACE scope is use to make target requirements that are used in any Libraries that link this target but not in the compilation of the target itself.

```
# Set the project name
project (sublibrary2)

add_library(${PROJECT_NAME} INTERFACE)
add_library(sub::lib2 ALIAS ${PROJECT_NAME})

target_include_directories(${PROJECT_NAME}
    INTERFACE
        ${PROJECT_SOURCE_DIR}/include
)
```

* subbinary

```
project(subbinary)

# Create the executable
add_executable(${PROJECT_NAME} main.cpp)

# Link the static library from subproject1 using its alias sub::lib1
# Link the header only library from subproject2 using its alias sub::lib2
# This will cause the include directories for that target to be added to this project
target_link_libraries(${PROJECT_NAME}
    sub::lib1
    sub::lib2
)
```

## CMake产生Protobuf

根据proto文件产生protobuf源码。

找到protobuf的模块后，有这些变量能用：

- `PROTOBUF_FOUND` - If Protocol Buffers is installed

- `PROTOBUF_INCLUDE_DIRS` - The protobuf header files

- `PROTOBUF_LIBRARIES` - The protobuf library

```
 PROTOBUF_GENERATE_CPP(PROTO_SRCS PROTO_HDRS AddressBook.proto)
```

  The arguments are:

  - PROTO_SRCS - Name of the variable that will store the .pb.cc files.
  - PROTO_HDRS- Name of the variable that will store the .pb.h files.
  - AddressBook.proto - The .proto file to generate code from.

```
cmake_minimum_required(VERSION 3.5)

# Set the project name
project (protobuf_example)

# find the protobuf compiler and libraries
find_package(Protobuf REQUIRED)

# check if protobuf was found
if(PROTOBUF_FOUND)
    message ("protobuf found")
else()
    message (FATAL_ERROR "Cannot find Protobuf")
endif()

# Generate the .h and .cxx files
PROTOBUF_GENERATE_CPP(PROTO_SRCS PROTO_HDRS AddressBook.proto)

# Print path to generated files
message ("PROTO_SRCS = ${PROTO_SRCS}")
message ("PROTO_HDRS = ${PROTO_HDRS}")

# Add an executable
add_executable(protobuf_example
    main.cpp
    ${PROTO_SRCS})

target_include_directories(protobuf_example
    PUBLIC
    ${PROTOBUF_INCLUDE_DIRS}
    ${CMAKE_CURRENT_BINARY_DIR}
)

# link the exe against the libraries
target_link_libraries(protobuf_example
    PUBLIC
    ${PROTOBUF_LIBRARIES}
)
```

## 静态检查-clang-analyzer

```
$ scan-build-3.6 cmake ..
$ scan-build-3.6 make 或 scan-build-3.6 -o ./scanbuildout make // scanbuildout是输出文件名

# 设置不同编译器
 scan-build-3.6 --use-cc=clang-3.6 --use-c++=clang++-3.6 -o ./scanbuildout/ make
```

## 静态检查-clang-format（有需要再看）

## 静态检查-cppcheck（有需要再看）

## Google-test

大概结构

```
$ tree
.
├── 3rd_party
│   └── google-test
│       ├── CMakeLists.txt
│       └── CMakeLists.txt.in
├── CMakeLists.txt
├── Reverse.h
├── Reverse.cpp
├── Palindrome.h
├── Palindrome.cpp
├── unit_tests.cpp
```

顶层CMakelists.txt

* 将每个class编译成library，将测试用例文件编译成可执行文件，链接library和GTEST的一些库。

* ```
  enable_testing()
  
  # 创建一个test执行测试用例
  add_test(test_all unit_tests)
  ```

```
cmake_minimum_required(VERSION 3.5)

# Set the project name
project (google_test_example)

# Add an library for the example classes
add_library(example_google_test
    Reverse.cpp
    Palindrome.cpp
)


#############################################
# Unit tests

add_subdirectory(3rd_party/google-test)

# enable CTest testing
enable_testing()

# Add a testing executable
add_executable(unit_tests unit_tests.cpp)

target_link_libraries(unit_tests
    example_google_test
    GTest::GTest
    GTest::Main
)

add_test(test_all unit_tests)
```

CMakeLists.txt.in，下载googletest库，然后需要再有个CMakeLists.txt来安装googletest

```
cmake_minimum_required(VERSION 3.0)

project(googletest-download NONE)

include(ExternalProject)

# Version bfc0ffc8a698072c794ae7299db9cb6866f4c0bc happens to be master when I set this up.
# To prevent an issue with accidentally installing GTest / GMock with your project you should use a
# commit after 9469fb687d040b60c8749b7617fee4e77c7f6409
# Note: This is after the release of v1.8
ExternalProject_Add(googletest
  URL               https://github.com/google/googletest/archive/bfc0ffc8a698072c794ae7299db9cb6866f4c0bc.tar.gz
  SOURCE_DIR        "${CMAKE_CURRENT_BINARY_DIR}/googletest-src"
  BINARY_DIR        "${CMAKE_CURRENT_BINARY_DIR}/googletest-build"
  CONFIGURE_COMMAND ""
  BUILD_COMMAND     ""
  INSTALL_COMMAND   ""
  TEST_COMMAND      ""
)
```

## 生成安装包（有需要再看）

## 包管理（有需要再看）

* find_package() 系统安装的包
* 第三方库只有源码文件(cpp hpp)
* 导入外部库，需下载，参考googletest

## 学习资料

* [cmake-examples](https://github.com/ttroy50/cmake-examples)

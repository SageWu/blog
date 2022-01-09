# CMake

## 变量

* PROJECT_NAME：项目名称。

### 系统

* CMAKE_SYSTEM_NAME：构建系统名称，默认和CMAKE_HOST_SYSTEM_NAME一样。
* CMAKE_HOST_SYSTEM_NAME：宿主系统名称。

### 目录

* PROJECT_SOURCE_DIR：项目源码根目录。
* CMAKE_BINARY_DIR：构建树根目录，in-source构建时，和PROJECT_SOURCE_DIR一样。
* CMAKE_CURRENT_SOURCE_DIR：cmake当前处理的源码目录。
* CMAKE_MODULE_PATH：模块查找目录。

### 构建目标

* CMAKE_CXX_STANDARD：c++标准。
* CMAKE_C_STANDARD：c标准。
* CMAKE_POSITION_INDEPENDENT_CODE
* CMAKE_SKIP_BUILD_RPATH
* CMAKE_BUILD_WITH_INSTALL_RPATH：使用install路径作为rpth。
* CMAKE_INSTALL_RPATH_USE_LINK_PATH
* CMAKE_INSTALL_RPATH
* CMAKE_INSTALL_PREFIX：安装目录。

### 编译

* CMAKE_TOOLCHAIN_FILE：初始化构建工具链配置的cmake文件路径。
* CMAKE_C_COMPILER：c编译器路径。
* CMAKE_CXX_COMPILER：cpp编译器路径。
* CMAKE_LINKER：链接器路径。
* CMAKE_CXX_COMPILER_AR
* CMAKE_CXX_COMPILER_RANLIB
* CMAKE_CXX_LINK_FLAGS

## 命令

### cmake_minimum_required

cmake最低版本要求。

```cmake
cmake_minimum_required(VERSION cmake_version)
```

### project

创建新项目。

```cmake
project(name VERSION major.minor)
```

### add_executable

新增可执行构建目标。

```cmake
add_executable(name sources)
```

### include_directories

添加预处理器头文件搜索目录。

```cmake
include_directories(dir)
```

### add_definitions

添加源码编译时定义。

```cmake
add_definitions(-DNAME)
```

### target_include_directories

头文件搜索目录。

```cmake
target_include_directories(target_name <INTERFACE|PUBLIC|PRIVATE> directories)
```

### target_sources

构建目标添加源文件。

```cmake
target_sources(target_name <INTERFACE|PUBLIC|PRIVATE> source)
```

### target_link_directories

链接目标搜索目录。

```cmake
target_link_directories(target_name <INTERFACE|PUBLIC|PRIVATE> directory)
```

### target_link_libraries

构建目标链接其他库。

```cmake
target_link_libraries(target_name libraries)
```

### target_compile_definitions

构建目标添加编译时定义。

```cmake
target_compile_definitions(target_name <scope> definition)
```

### set_target_properties

构建目标添加属性，影响构建结果。

```cmake
set_target_properties(
  target_name
  PROPERTIES key value
)
```

### add_subdirectory

新增子项目，检索子目录下的CMakeLists.txt。

```cmake
add_subdirectory(dir)
```

### add_library

新增库文件，可用于链接。

```cmake
add_library(name sources)
```

### add_dependencies

一级构建目标相互添加依赖。

```cmake
add_dependencies(target_name dependency)
```

### include

包含其他CMakeLists.txt。

```cmake
include(path)
```

### option

选项，也即自定义变量。

```cmake
option(name description default_value)
```

### set

设置变量。

```cmake
set(variable_name value)
```

### configure_file

对文件进行配置，替换变量。

```cmake
configure_file(input output)
```

```cpp
#define NAME @VARIABLE_NAME@

#cmakedefine VARIABLE_NAME
```

### find_package

查找外部项目，并加载其配置。

```cmake
find_package(Git)

set(CMAKE_MODULE_PATH dir)
# 使用指定目录下的FindGit.cmake查找外部项目
find_package(Git MODULE <QUIET|REQUIRED>)
```

### find_library

查找库，并得到该库的路径。

```cmake
find_library(result libname NAMES orther_names DOC description)
```

### execute_process

执行命令。

```cmake
execute_process(
  COMMAND command
  WORKING_DIRECTORY dir
  RESULT_VARIABLE variable
)
```

### message

打印消息。

```cmake
message(level message)
```

### list

列表相关操作。

```cmake
list(APPEND list_name value)
```

### file

文件相关处理。

### install

将程序安装至特定位置，默认全局。
可使用cpack实现更简单的安装流程。

```cmake
install(TARGET target_name DESTINATION dir)
install(FILES file DESTINATION dir)
```

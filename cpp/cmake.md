# CMake

## 变量

* PROJECT_NAME：项目名称。

### 系统

* CMAKE_SYSTEM_NAME：构建目标系统名称，默认和CMAKE_HOST_SYSTEM_NAME一样。
* CMAKE_HOST_SYSTEM_NAME：宿主系统名称。

### 目录

* PROJECT_SOURCE_DIR：和调用project相关。
* CMAKE_BINARY_DIR
  
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

* CMAKE_C_COMPILER
* CMAKE_CXX_COMPILER
* CMAKE_LINKER
* CMAKE_CXX_COMPILER_AR
* CMAKE_CXX_COMPILER_RANLIB
* CMAKE_CXX_LINK_FLAGS

## 命令

* cmake_minimum_required：版本检查。
* set：设置变量。
* message：输出消息。
* include：包含其他cmake文件。
* option：选项。
* add_definitions：定义宏。
* project：创建项目。
* add_subdirectory：添加子项目。
* include_directories：添加头文件目录。
* aux_source_directory：检索目录添加源文件。
* add_library
* target_include_directories
* target_compile_definitions
* set_target_properties
* target_link_libraries
* add_dependencies
* install：安装命令。
* file：文件处理。
* list：列表处理。

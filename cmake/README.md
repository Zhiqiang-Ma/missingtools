### 1，推荐的目录机构

~~~cmake
project
	CMakeLists.txt
	build
	bin
		main.cpp
	src
		lib_one
			CMakeLists.txt
			hello.h
			hello.cpp
		lib_two
			CMakeLists.txt
			world.h
			world.cpp
~~~

在工程project目录下共有 src， bin，build三个文件件和一个主工程的cmake文件。

src为工程的源码文件夹，包括声明h文件和定义cpp文件。

在bin下面的main文件中调用。

build 中为编译之后可执行文件

#### 1.1，主文件cmake

主工程的CMake内容为

~~~cmake
CMake_minimum_required(VERSION 3.0)
project(project_name)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread -g")
set(CMAKE_BUILD_TYPE Debug)
add_subdirectory(${PROJECT_SOURCE_DIR}/src/student)
~~~

* CMAKE_CXX_FLAGS

  为C++ 编译标志，pthread 为使用线程函数，g 为产生可调试的执行文件

* add_subdirectory

  用于添加子目录中cmake 所在的路径

#### 1.2，子目录cmake

src文件下面的子目录中cmake文件内容

~~~cmake
set(SOURCES
student.cpp
${PROJECT_SOURCE_DIR}/bin/main_student.cpp
)
add_executable(student ${SOURCES} ${PROJECT_SOURCE_DIR}/bin)
include_directories("${PROJECT_SOURCE_DIR}/src/student")
~~~

* set 设置变量的值

* add_executable 

  生成可执行文件（可执行文件的名称，待编译的文件，生成可执行文件的位置）

* include_directories

  添加头文件，可以直接在在main 文件中写/src/student 中的 .h 文件

  ~~~c++
  include "student.h"
  include "/src/studentstudent.h" # 替换
  ~~~

**PS 另外的一种：**

首先建立目录结构如下：

~~~
|project
|---CMakeLists.txt
|---include
	|---|hello.h
|---src
	|---hello.cpp
	|---main.cpp
~~~

文件夹include是头文件，src文件夹位源文件。

顶层的CMakeLists.txt 的内容如下：

~~~cmake
CMake_minimum_required(VERSION 3.0) // cmake 的版本
project(PRO_ver1)					// 工程名称
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wal ")  // 编译选项设置
set(CMAKE_BUILD_TYPE Debug)
// 设置变量，除头文件外，讲需要编译的文件定义为一个变量
set(SOURCES
    src/tabletennis.cpp
    src/main.cpp
)
// 包含头文件路径
include_directories("${PROJECT_SOURCE_DIR}/include")
// 生成可执行文件
add_executable(tabletennis ${SOURCES})
~~~

使用外部编译方式进行编译

~~~shell
mkdir build
cd build
cmake ..
make
~~~

### 2，常用环境变量

* 当前的工程目录，project

~~~cmake
PROJECT_SOURCE_DIR
~~~

* rpath 动态库搜索路径

~~~cmake
set(CMAKE_INSTALL_RPATH "/path/to/dynamic/libraries")
~~~

另外，还可以使用`CMAKE_BUILD_WITH_INSTALL_RPATH`变量来指示CMake在构建过程中使用`CMAKE_INSTALL_RPATH`的值作为构建时的运行时库搜索路径。

~~~cmake
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
~~~

这样，在构建过程中将使用`CMAKE_INSTALL_RPATH`设置的路径作为运行时库搜索路径。

* CMAKE_BUILD_TYEP 

cmake 中的编译选项，默认为空，相当于debug 模式。可以有Release， Debug 等模式。 

标准模式：

~~~cmake
if(NOT CMAKE_BUILD_TYEP)
	set(CMAKE_BUILD_TYEP Release)
endif()
~~~

* CMAKE_CXX_FLAGS

编译设置，一般

~~~cmake
if(NOT MSVC)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14 -pthread -fPIC")
endif()
~~~

* CMAKE_CXX_FLAGS_DEBUG

~~~makefile
set(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -g")
~~~

* CMAKE_CXX_STANDARD

设置C++ 的编译标准。

~~~cmake
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON) # 检测是否满足标准，否则报错
~~~

* BUILD_SHARED_LIBS 

设置生成库的类型

~~~cmake
if(NOT DEFINE BUILD_SHARED_LIBS)
	set(BUILD_SHARED_LIBS  on)
endif()
~~~

* CMAKE_VERBOSE_MAKEFILE

~~~
This variable is a cache entry initialized (to FALSE) by the project() command. Users may enable the option in their local build tree to get more verbose output from Makefile builds and show each command line as it is launched.
~~~

* CMAKE_MODULE_PATH

这个变量用来定义自己的cmake模块所在的路径。该变量默认为空，需要自己定义。

如果工程比较复杂，有可能会自己编写一些cmake模块，这些cmake模块是随工程发布的，为了让cmake在处理CMakeLists.txt时找到这些模块，你需要通过SET指令将cmake模块路径设置一下。比如，

~~~
src/
|---CMakeList.txt
|---cmake
	|---gflags.cmake
	|---glog.cmake
	|---libtorch.cmake
|---build
~~~

使用set指定自定义模块的文件位置， cmake 文件夹中包括需要安装模块的CMakeList.txt 文件，注意使用.cmake文件结尾

~~~cmake
SET(CMAKE_MODULE_PATH,${PROJECT_SOURCE_DIR}/cmake)
~~~

这时候就可以通过INCLUDE指令来调用自己的模块了。

~~~cmake
option(TORCH "whether to build with Torch" ON)
if(TORCH)
	include(libtorch)
endif
~~~

* CMAKE_CURRENT_SOURCE_DIR

一般是 build 的上一层目录，即跟主CMakeList.txt 是在同一级目录

### 3，CMake 函数

#### 3.1，option

option 命令用于定义一个选项（全局变量），该选项可以在 CMake 配置时由用户设置

~~~cmake
option(<option_variable> "help string" [initial_value])
~~~

其中，`<option_variable>` 是选项的名称，`"help string"` 是选项的帮助文本，`[initial_value]` 是选项的初始值（可选）

~~~cmake
option(MY_OPTION "Enable mylib" ON)
~~~

可以使用-D` 选项来设置选项的值。例如，以下命令将 `MY_OPTION` 选项的值设置为 `ON

~~~cmake
cmake -DMY_OPTION=ON /path/to/source
~~~

可以结合if 进行判断

~~~cmake
if (MY_OPTION)
	add_library(mylib STATIC mylib.cpp)
	target_link_libraries(mylib PUBLIC ${MY_LIBRARIES})
endif
~~~

#### 3.2，设置编译选项

* 1,，set命令修改CMAKE_CXX_FLAGS或CMAKE_C_FLAGS

~~~cmake
set(CMAKE_CXX_FLAGS "-Wall -Werror -Wstrict-prototypes -Wmissing-prototypes)
~~~

另外，set 可以设置变量。注意：set 变量的时候没有中间的逗号 

~~~makefile
set(mysql++_BINARY_DIR ${thirdparts}/mysql++-src CACHE PATH "mysql++ build directory")
~~~

以上命令中的

CACHE  表示，该变量放在cmake 的缓存中，在构建过程中保持不变

PATH  表示，是一个路径

* 2，add_compile_options

CMake中用于添加编译选项的命令。该命令可以用于为特定的目标或整个项目添加编译选项

~~~cmake
add_compile_options(-Wall -O2)
add_compile_options(-Wall -Werror -Wstrict-prototypes -Wmissing-prototypes)
~~~

上述示例将添加`-Wall`和`-O2`编译选项

如果要为特定的目标添加编译选项，可以将`add_compile_options`命令放在`target_compile_options`命令中

~~~cmake
target_compile_options(your_target_name PRIVATE -Wall -O2)
~~~

* 3，add_definitions

~~~cmake
add_definitions("-Wall -Werror -Wstrict-prototypes -Wmissing-prototypes)
~~~

总结：

使用这三种方式在有的情况下效果是一样的，但请注意它们还是有区别的：
add_compile_options命令和add_definitions添加的编译选项是针对所有编译器的(包括c和c++编译器)，而set命令设置CMAKE_C_FLAGS或CMAKE_CXX_FLAGS变量则是分别只针对c和c++编译器的。

#### 3.3，message 

~~~cmake
SET(USER_KEY, "Hello World")
MESSAGE(STATUS "this var key = ${USER_KEY}")
~~~

#### 3.4，target_compile_definitions

target_compile_definitions 命令用于向target 添加编译定义

~~~makefile
target_compile_definitions(cblas PRIVATE -DBUILD_SHARED_LIBS)
~~~

#### 3.5，add_definitions

将 -D 定义标志添加到源文件的编译中。

将定义添加到当前目录中的目标的编译器命令行，无论是在调用此命令之前还是之后添加，以及之后添加的子目录中的目标。此命令可用于添加任何标志，但它旨在添加预处理器定义。

在文件中使用

~~~cmake
add_definitions(-DMYSQLPP_MYSQL_HEADERS_BURIED)
~~~

在cpp 文件中使用

~~~C++
#if defined(MYSQLPP_MYSQL_HEADERS_BURIED)
#       include <mysql/mysql.h>
#else
#       include <mysql.h>
#endif
~~~

注意与set和option 的不同。

#### 3.8，include

作用：include 指令用来载入并运行来自于文件或模块的 CMake 代码。

用于在 CMakeLists.txt 文件中下载、编译和安装外部依赖库。

~~~cmake
include(FetchContent)
FetchContent_Declare(
	<name>
	[GIT_REPOSITORY <url>]
	[URL <url>]
	[URL_HASH <hash>]
	[SOURCE_DIR <dir>]
	)
FetchContent_MakeAvailable(<name>)
~~~

FetchContent_Declare  子命令用于声明要下载的库的信息，<name> 是要下载的库的名称。FetchContent_MakeAvailable 子命令用于下载、编译和安装指定的库，`<name>` 是要下载的库的名称。

#### 3.9，include_directories 

相当于g++ 中的 -I 参数

将指定目录添加到编译器的头文件搜索路径之下，指定的目录被解释成当前源码路径的相对路径。

~~~cmake
include_directories ([AFTER|BEFORE] [SYSTEM] dir1 [dir2 ...])
~~~

举例

~~~cmake
include_directories(sub) 
include_directories(sub2) #默认将sub2添加到列表最后
include_directories(BEFORE sub3) #可以临时改变行为，添加到列表最前面
~~~

#### 3.10，aux_source_directory 

* 基本使用

~~~cmake
aux_source_directory(<dir> <variable>)
~~~

将 \<dir> 下的源文件都保存在 变量 \<variable> 中。比如有如下的目录

~~~cmake
CMakeList.txt
kenlm
	|--kenlm.cpp
	|--kenlm.h
~~~

在CMakeList.txt 中的语句

~~~cmake
aux_source_directory(kenlm DIR_LIST)
MESSAGE(STATUS "${DIR_LIST}")
~~~

* 指定文件加入变量中

将目录下的 .cpp 和 .c 文件加入，变量中

~~~cmake
aux_source_directory(kenlm/*.cpp src/*.c DIR_LIST)
~~~

* 除了某文件外，加入变量

使用 set 和EXCLUDE 的组合

~~~cmake
set(DIR_LIST "")
aux_source_directory(kenlm SOURCES EXCLUDE src/main.cpp)
~~~

#### 3.12，file 添加源文件

~~~cmake
file(GLOB source CONFIGURE_DEPENDS *.cpp *.h)
~~~

CONFIGURE_DEPENDS 实时的更新文件列表

这样就会把所有的 .cpp  和 .h 文件都添加到变量 source 中

思考： 如何的指定目录

#### 3.13，add_library

该指令的主要作用就是将指定的源文件生成链接文件，然后添加到工程中去

~~~cmake
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [source1] [source2] [...])
~~~

* \<name>表示库文件的名字，该库文件会根据命令里列出的源文件来创建。

* STATIC、SHARED和MODULE的作用是指定生成的库文件的类型。

  STATIC库是目标文件的归档文件，在链接其它目标的时候使用。

  SHARED库会被动态链接（动态链接库），在运行时会被加载。

  MODULE库是一种不会被链接到其它目标中的插件，但是可能会在运行时使用dlopen-系列的函数。

​		默认状态下，库文件将会在于源文件目录树的构建目录树的位置被创建，该命令也会在这里被调用。

* source1 source2 分别表示各个源文件

#### 3.14，target_include_directories

target_include_directories() 命令有三种不同的路径传递关系，分别是 PRIVATE、PUBLIC 和 INTERFACE。这些关键字决定了头文件路径如何被传播到其他目标。

* PUBLIC 

头文件路径仅对当前目标可见。不会传递给其他链接到此目标的目标

~~~~cmake
target_include_directories(mylib PUBLIC include/path)
~~~~

任何链接到 mylib 的其他目标也会看到 include/path目录。

* PRIVATE 

头文件路径仅对当前目标可见。不会传递给其他链接到此目标的目标。

~~~cmake
target_include_directories(mylib PRIVATE include/path)
~~~

include/path 目录仅对 mylib 目标可见

* INTERFACE 在interface后面引入的库不会被链接到你的target中，只会导出符号。

头文件路径对当前目标不可见。仅传递给其他链接到此目标的目标。

~~~cmake
target_include_directories(mylib INTERFACE include/path)
~~~

include/path 目录对 mylib 不可见，但对链接到 mylib 的任何其他目标可见。

#### 3.15，target_link_libraries 

该指令的作用为: 将目标文件与库文件进行链接

~~~
target_link_libraries(<target> [item1] [item2] [...]
                      [[debug|optimized|general] <item>] ...)
~~~

* \<target>是指通过 add_executable() 和 add_library() 指令生成已经创建的目标文件。
* [item]表示库文件没有后缀的名字。

默认情况下，库依赖项是传递的。当这个目标链接到另一个目标时，链接到这个目标的库也会出现在另一个目标的连接线上。

#### 3.16，link_directories

用于添加目录使链接器能在其查找库， **相当于在gcc 命令行编译中的 -L 参数**

~~~cmake
link_directories([AFTER|BEFORE] directory1 [directory2 ...])
~~~

用于安装库后，指定库搜索的目录。

注意：该命令为全局的变量，会给之后的所有编译操作，加上对应的头文件。

#### 3.17，add_subdirectory 

**添加一个子目录并构建该子目录**

~~~cmake
add_subdirectory (source_dir [binary_dir] [EXCLUDE_FROM_ALL])
~~~

* source_dir

**必选参数**。该参数指定一个子目录，子目录下应该包含`CMakeLists.txt`文件和代码文件。子目录可以是相对路径也可以是绝对路径，如果是相对路径，则是相对当前目录的一个相对路径

* binary_dir

**可选参数**。该参数指定一个目录，用于存放输出文件。可以是相对路径也可以是绝对路径，如果是相对路径，则是相对当前输出目录的一个相对路径。如果该参数没有指定，则默认的输出目录使用`source_dir`

* EXCLUDE_FROM_ALL

**可选参数**。当指定了该参数，则子目录下的目标不会被父目录下的目标文件包含进去，父目录的`CMakeLists.txt`不会构建子目录的目标文件，必须在子目录下显式去构建。例外情况：当父目录的目标依赖于子目录的目标，则子目录的目标仍然会被构建出来以满足依赖关系（例如使用了target_link_libraries）

**注意：**

目录结构如下：

~~~cmake
cpp/
|---3rdpart
|------CMakeList.txt
|---main
|------CMakeList.txt
|------sub1
|---------CMakeLIst.txt
~~~

* 添加主工程中的目录使用默认设置即可

~~~cmake
#只需要传入相对主目录的相对路径`sub1`
add_subdirectory(sub1)
~~~

* 添加需要依赖外部目录(即不是主目录的子目录)

  就需要指定绝对路径，指定用于存储输出文件的binary_dir参数。否则报如下错误

~~~cmake
add_subdirectory not given a binary directory but the given source directory "xxx/thirdlib" is not a subdirectory of "xxx/main".  When specifying an out-of-tree source a binary directory must be explicitly specified.
~~~

正确方式为：

~~~cmake
#CMAKE_CURRENT_SOURCE_DIR上当CMake目录
add_subdirectory(../3rdparty ${CMAKE_CURRENT_SOURCE_DIR})
~~~

参考：

~~~
https://www.jianshu.com/p/d5d3f13d9a96
~~~

#### 3.18，intall 安装文件

命令用于安装编译生成的文件到指定位置。比如在linux系统下若LIBRARY的安装路径指定为lib, 即为 /usr/local/lib。所以要安装们可以这样写。

~~~cmake
set(CMAKE_INSTALL_PREFIX "usr/local/thirdparts")
install(TARGETS MyLib
        EXPORT MyLibTargets 
        LIBRARY DESTINATION lib  # 动态库安装路径
        ARCHIVE DESTINATION lib  # 静态库安装路径
        RUNTIME DESTINATION bin  # 可执行文件安装路径
        PUBLIC_HEADER DESTINATION include  # 头文件安装路径
        )
~~~

根目录默认为CMAKE_INSTALL_PREFIX，可以试用`set`方法进行指定，如果使用默认值的话，Unix系统的默认值为 /usr/local

#### 3.19，find_package

find_package 本质上就是一个**搜包的命令**，通过一些特定的规则找到`<package_name>Config.cmake`包配置文件，通过执行该配置文件，从而定义了一系列的变量，通过这些变量就可以准确定位到 库的头文件和库文件，完成编译。

以Zlib 库为例子进行说明：

* 设置ZLIB库路径

~~~cmake
set(ZLIB_ROOT /path/to/zlib)
~~~

* 查找ZLIB库

~~~cmake
find_package(ZLIB REQUIRED)
~~~

* 检查库是否存在

~~~cmake
if(NOT ZLIB_FOUND)
    message(FATAL_ERROR "ZLIB library not found")
endif()
~~~

* 添加ZLIB库的头文件和链接库

~~~cmake
include_directories(${ZLIB_INCLUDE_DIRS})
target_link_libraries(your_target_name ${ZLIB_LIBRARIES})
~~~

#### 3.20，FetchContent

* FetchContent_Declare

管理第三方的库文件，会建立三个子目录

~~~cmake
name-src		: 存放远吗
name-build 		：编译目录
namd-subbuild	：存放的cmake 文件
~~~

使用例子：

~~~cmake
FetchContent_Declare(kenlm
			URL ${thirdparts}/kenlm-master.tar.gz
			URL_HASH SHA256 = xxx
			)
FetchContent_MakeAvailable(kenlm)					# 使得相应的头文件可用
include_directories(${kenlm_SOURCE_DIR}/lm) 		# 包含相应的头文件
~~~

* FetchContent_MakeAvailable

FetchContent_MakeAvailable 命令会检查之前是否已经调用了 FetchContent_Declare 来声明外部项目。如果已经声明，它会检查这些项目是否已经下载。

如果项目尚未下载，它会执行下载操作。

下载完成后，它会将外部项目添加到当前的构建中，这通常意味着它会调用该项目的CMakeLists.txt 文件，从而使得外部项目的目标（如库和可执行文件）可以在当前项目中使用

为什么有的库，把生成的可自行文件放到了 bin 下面，相应的头文件放在了 include 中，而有的库不是那？

  1. 在cmake文件写入  include(FetchContent) 
  2. 使用FetchContent_Declare(三方库) 获取项目。可以是一个URL也可以是一个Git仓库。
  3. 使用FetchContent_MakeAvailable(三方库) 获取我们需要库,然后引入项目。
  4. 使用 target_link_libraries(项目名PRIVATE 三方库::三方库)

#### 3.21，ExternalProject_Add

安装第三方依赖包，并将项目进行独立的管理。

* 配置安装的库

以下为安装gflag 的例子

~~~cmake
include(ExternalProject)

# 设置相应的环境变量
set(GFLAG_ROOT          ${CMAKE_BINARY_DIR}/thirdparty/gflag-2.2.2)
set(GFLAG_LIB_DIR       ${GFLAG_ROOT}/lib)
set(GFLAG_INCLUDE_DIR   ${GFLAG_ROOT}/include)
 
set(GFLAG_URL           https://github.com/gflags/gflags/archive/v2.2.2.zip)
set(GFLAG_CONFIGURE     cd ${GFLAG_ROOT}/src/gflag-2.2.2 && cmake -D CMAKE_INSTALL_PREFIX=${GFLAG_ROOT} .)
set(GFLAG_MAKE          cd ${GFLAG_ROOT}/src/gflag-2.2.2 && make)
set(GFLAG_INSTALL       cd ${GFLAG_ROOT}/src/gflag-2.2.2 && make install)
 
ExternalProject_Add(gflag-2.2.2
        URL                   ${GFLAG_URL}
        DOWNLOAD_NAME         gflag-2.2.2.zip
        PREFIX                ${GFLAG_ROOT}
        CONFIGURE_COMMAND     ${GFLAG_CONFIGURE}
        BUILD_COMMAND         ${GFLAG_MAKE}
        INSTALL_COMMAND       ${GFLAG_INSTALL}
)
~~~

可以通过 CONFIGURE_COMMAND 来运行 configure 脚本，从而自定义安装命令。

常见的默认目录：

~~~cmake
TMP_DIR      = <prefix>/tmp
STAMP_DIR    = <prefix>/src/<name>-stamp
DOWNLOAD_DIR = <prefix>/src
SOURCE_DIR   = <prefix>/src/<name>
BINARY_DIR   = <prefix>/src/<name>-build
INSTALL_DIR  = <prefix>
LOG_DIR      = <STAMP_DIR>
~~~

参数解释

```cmake
SOURCE_DIR <dir>
```

源目录,下载的内容将被解压到该目录中,或者对于非URL下载方法,版本库应该在该目录中被签出、克隆等。如果没有指定下载方式,则必须指向外部项目已经被解压或克隆/签出的现有目录。

Note

如果指定了下载方法,源目录中的任何现有内容都可能被删除。只有URL下载方法在启动下载前会检查该目录是否缺失或为空,如果不为空,则以一个错误停止。所有其他的下载方法都会默默地丢弃源目录中以前的任何内容。

```
BINARY_DIR <dir>
```

指定构建目录位置。如果启用了 `BUILD_IN_SOURCE` ,则忽略此选项。

```
INSTALL_DIR <dir>
```

要放置在 `<INSTALL_DIR>` 占位符中的安装前缀。这实际上并没有配置外部项目以安装到给定的前缀。这必须通过将适当的参数传递给外部项目配置步骤来完成，例如使用 `<INSTALL_DIR>` 。

* 使用安装的库

进一步的引入库的头文件和so文件

~~~cmake
set(MYSQL_INCLUDE_DIR ${mysql++_SOURCE_DIR}/lib)
set(MYSQL_LIBRARY ${mysql++_SOURCE_DIR})
~~~

将属性绑定到库上

~~~cmake
add_library(mysqlpp SHARED IMPORTED)
set_target_properties(mysqlpp PROPERTIES IMPORTED_LOCATION ${MYSQL_LIBRARY})
target_include_directories(mysqlpp INTERFACE ${MYSQL_INCLUDE_DIR})
~~~

MPORTED 参数的含义：

IMPORTED 参数则表明这个库并不是由当前 CMake 构建系统所生成的，而是从外部获取的。也就是说，CMake 不会对 mysqlpp 库进行任何编译步骤，因为假设它已经在系统中存在了因此，你需要提供足够的信息让 CMake 知道如何找到这个已有的库，这通常包括库的路径以及对应的导入库。

以上方式的另一种解法

~~~cmake
set_target_properties(mysqlpp PROPERTIES
    IMPORTED_LOCATION /path/to/libmysqlpp.so
     IMPORTED_IMPLIB /path/to/mysqlpp.lib
     )
~~~

#### 3.22，get_filename_component

get_filename_component 是 CMake 中的一个函数，用于获取文件路径的各个部分

~~~~cmake
get_filename_component(<VAR> <FileName> <COMP>)
~~~~

其中，`<VAR>` 是要设置的变量名，`<FileName>` 是文件路径，`<COMP>` 是要获取的路径部分，可以是以下值：

* ABSOLUTE：获取文件的绝对路径；
* DIRECTORY：获取文件所在的目录；
* FILENAME：获取文件名和扩展名；
* NAME：获取文件名，不包括扩展名；
* EXT：获取文件的扩展名。

需要注意的是，get_filename_component 函数只能获取文件路径的各个部分，不能判断文件是否存在。如果需要判断文件是否存在，可以使用  file(EXISTS\ <file\>) 命令。

#### 3.23，for_each

foreach  命令用于遍历一个列表，并对每个元素执行一组命令

~~~cmake
foreach(<loop_variable> <items>)
	<commands>
endforeach()
~~~

其中，<loop_variable> 是循环变量，\<items> 是要遍历的列表，可以是一个列表变量，也可以是一个由空格分隔的字符串，\<commands> 是要执行的命令

* 使用例子：

~~~cmake
set (FST_BINFS
	fstaddselfloops
	fstdeterminizestar
	fstisstochastic)
foreach(name IN LISTS FST_BINS)
	add_executable(${name} fstbin/${name}.cc)
	target_link_libraries(${name} PUBLIC kaldi-fstext glog)
endforeach()
~~~

以上使用set 建立一个list 集合，然后使用foreach 生成对应的可执行文件。

#### 3.24，set_property

设置属性

#### 3.25，set_target_porpertys

findcmake 文件 config.cmake

删除 cmakecash.txt文件

ccmake 图形化编辑器

安装 zlib 

~~~shell
apt-get install zlib1g-dev
apt-get install libbz2-dev
apt-get install liblzma-dev
~~~

#### 3.26，ifneq

~~~cmake
ifneq ($(BOARD_HAVE_BLUETOOTH_BCM),)
~~~

比较两个参数是否相同，第二个值为空，表示如果BOARD_HAVE_BLUETOOTH_BCM的值不为空，就可以进行下面的编译。

#### 3.27，function 自定义函数

~~~cmake
function(my_function arg1 arg2)
  # 函数体
  message("Hello from my_function")
  message("arg1: ${arg1}")
  message("arg2: ${arg2}")
endfunction()
my_function("Hello" "World")
~~~

#### 3.28，list 使用

用于添加列表内容

~~~cmake
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
~~~

#### 3.26，手动的添加库文件



### 4， Autotools 构建

* Makefile.in

Makefile.in 是一个模板文件，通常用于自动化构建工具如 autoconf 和 automake。这些工具是 GNU build system 的一部分，也被称为 Autotools。Makefile.in 文件包含了可以被 configure 脚本处理的占位符。当你运行 configure 脚本时，它会根据系统的特定情况和用户提供的配置选项，将 Makefile.in 文件转换成实际的 Makefile

工具链: Makefile.in 是 Autotools 工具链的一部分，而 CMakeLists.txt 是 CMake 工具的配置文件。

配置过程: 使用 Makefile.in 时，必须先运行 configure 脚本来生成 Makefile。而 CMake 直接处理 CMakeLists.txt 来生成构建文件。

语法和功能: CMakeLists.txt 使用 CMake 特有的命令和宏，而 Makefile.in 使用的是 Makefile 语法，并且可能包含 Autotools 宏。

* CMakeLists.txt

CMakeLists.txt 是 CMake 构建系统使用的配置文件。CMake 是一个跨平台的构建工具，它使用 CMakeLists.txt 文件来定义项目的构建过程。

总结：

CMakeLists.txt 是 CMake 的配置文件，而 Makefile.in 是 Autotools 使用的模板文件，属于不同的构建系统。

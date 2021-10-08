<h1 align = center>CMake FAST LEARNING</h1>
<h4 align = right>update 2021.10.8</h4>

查阅Cmake官方太慢， 还是按照自己的笔记来吧….

> ### 目录
>
> 1. 介绍
> 2. 基础函数
> 3. 进阶
> 4. 指定输出路径
> 5. CUDA 编译
> 6. 参考资料





> <h3 align=center>介绍</h3>

CMake是一个比Make工具更高级的编译配置工具，是一个跨平台的、开源的构建系统（BuildSystem）。CMake允许开发者编写一种平台无关的CMakeList.txt文件来定制整个编译流程， 然后再根据目标用户的平台进一步生成所需的本地化Makefile和工程文件



在Linux平台下使用CMake生成Makefile并编译的流程如下：
A、编写CMake配置文件CMakeLists.txt
B、执行命令cmake PATH生成Makefile，PATH是CMakeLists.txt所在的目录。
C、使用make命令进行编译。





### 重点

重点在于你所需要执行的文件需要哪些库， 源文件 来组成 则是编译的重点 !





> <h3 align=center>基础函数</h3>

------



**#** 可用来注释

#### project()

给工程取名字







#### include_directories 让CMake找到我的头文件

把当前目录下的添加Include 文件夹 to the build

```cmake
include_directories([AFTER|BEFORE] [SYSTEM] dir1 [dir2 ...])
```

Ex 

`include_directories(${CMAKE_CURRENT_LIST_DIR}/include)`





#### aux_source_directory 让CMake找到我的源文件

命令：`aux_source_directory(<dir> <variable>)`

作用：查找dir路径下的所有源文件，保存到variable变量中.

EX.

`aux_source_directory(./src ${hello_src})`

把`./src` 目前下所有源文件放到`hello_src`这个变量

之后就可以用`add_executable(hello ${hello_src})`, 意思就是用hello_src这个变量来建构hello这个可执行程序





#### link_directories() 让CMake 找到我的库文件

`link_directories(directory1 directory2 …)`

Ex.

`link_directories(${CMAKE_CURRENT_LIST_DIR}/lib)`



#### add_subdirectory() 添加外部项目文件夹

一般情况下，我们的项目各个子项目都在一个总的项目根目录下，但有的时候，我们需要使用外部的文件夹，怎么办呢？`add_subdirectory`命令，可以将指定的文件夹加到build任务列表中

```
add_subdirectory(source_dir [binary_dir]
                 [EXCLUDE_FROM_ALL])
```

Ex 假设s1为project主目录， sub_haha, sub_hello, top这三个为子目录

```
s1
1 ----sub_haha
				---haha.c
				---CMakeLists.txt
2 ----sub_hello
				---hello.h
				---hello.c
				---CMakeLists.txt
3 ----top
				---main.c
				---CMakeLists.txt
4 ----CMakeLists.txt
```

s1下的CMakeLists.txt编写为

```cmake
1 cmake_minimum_required(VERSION 2.8)
2 add_subdirectory(sub_haha sub_haha)
3 add_subdirectory(sub_hello sub_hello)
4 add_subdirectory(top top)
```

#### add_library() 新建库

```
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [source1] [source2 ...])
```

- <name> ： 表示要设定的库名称
- [STATIC | SHARED | MODULE] : 表示该库设定为 静态库 or 动态库 或着 Module库
- [source1]…: 源文件

Ex. onnx-tensorRT 外挂

```cmake
#将外挂源文件都保存到PLUGIN_SOURCES变量中
set(PLUGIN_SOURCES
  FancyActivation.cu
  ResizeNearest.cu
  Split.cu
  dcn_v2_im2col_cuda.cu
  InstanceNormalization.cpp
  DCNv2.cpp
  plugin.cpp
  )
  
  #条件式判断
  if(${CMAKE_VERSION} VERSION_LESS ${CMAKE_VERSION_THRESHOLD})
    CUDA_INCLUDE_DIRECTORIES(${CUDNN_INCLUDE_DIR} ${TENSORRT_INCLUDE_DIR})
    CUDA_ADD_LIBRARY(nvonnxparser_plugin STATIC ${PLUGIN_SOURCES})
  else() #add_library 设置为静态库(.a), 将前面设置在变量中的源文件加入
    include_directories(${CUDNN_INCLUDE_DIR} ${TENSORRT_INCLUDE_DIR})
    add_library(nvonnxparser_plugin STATIC ${PLUGIN_SOURCES})
  endif()
    target_include_directories(nvonnxparser_plugin PUBLIC ${CUDA_INCLUDE_DIRS} ${ONNX_INCLUDE_DIRS} ${TENSORRT_INCLUDE_DIR} ${CUDNN_INCLUDE_DIR})
    target_link_libraries(nvonnxparser_plugin ${TENSORRT_LIBRARY} cuda cudart cublas)
  
  
```







#### find_library() 查找库

https://www.cnblogs.com/coderfenghc/archive/2012/07/14/2591135.html

```cmake
find_library (
          <VAR>
          name | NAMES name1 [name2 ...] [NAMES_PER_DIR]
          [HINTS path1 [path2 ... ENV var]]
          [PATHS path1 [path2 ... ENV var]]
          [PATH_SUFFIXES suffix1 [suffix2 ...]]
          [DOC "cache documentation string"]
          [NO_DEFAULT_PATH]
          [NO_PACKAGE_ROOT_PATH]
          [NO_CMAKE_PATH]
          [NO_CMAKE_ENVIRONMENT_PATH]
          [NO_SYSTEM_ENVIRONMENT_PATH]
          [NO_CMAKE_SYSTEM_PATH]
          [CMAKE_FIND_ROOT_PATH_BOTH |
           ONLY_CMAKE_FIND_ROOT_PATH |
           NO_CMAKE_FIND_ROOT_PATH]
         )
```

主要用来找库（library)

- VAR : variable用来存储找到的库
- Name : 设定一个或者多个这个库的名称
- HINTS path ：除了default设定的目录之外， 指定目录来寻找, 也可以指定var
- PATH_SUFFIXES ：选项指定了每个搜索路径下待搜索的子路径

Ex. 假设需要连接某个动态库叫做libnvonnxparser.so, 并且从lib/arrch64-linux-gnu路径下找， 最终将找到的库保存到TENSORRT_LIBRARY中

```cmake
find_library(TENSORRT_LIBRARY_ONNXPARSER nvonnxparser
        HINTS  ${TENSORRT_ROOT} ${TENSORRT_BUILD} ${CUDA_TOOLKIT_ROOT_DIR}
        PATH_SUFFIXES  lib/aarch64-linux-gnu)
```


#### find_package() 查找包 : 较find_library（） 强大
调用该函数时候， 会先从cmake原生支持的module中寻找

ps.CMake原生支持许多module的寻找，通过cmake –help-modul-list，可以查看，相关原生.cmake文件存放在/usr/share/cmake/Modules
```cmake
find_package(<PackageName> [version] [EXACT] [QUIET]
             [REQUIRED] [[COMPONENTS] [components...]]
             [OPTIONAL_COMPONENTS components...]
             [CONFIG|NO_MODULE]
             [NO_POLICY_SCOPE]
             [NAMES name1 [name2 ...]]
             [CONFIGS config1 [config2 ...]]
             [HINTS path1 [path2 ... ]]
             [PATHS path1 [path2 ... ]]
             [PATH_SUFFIXES suffix1 [suffix2 ...]]
             [NO_DEFAULT_PATH]
             [NO_PACKAGE_ROOT_PATH]
             [NO_CMAKE_PATH]
             [NO_CMAKE_ENVIRONMENT_PATH]
             [NO_SYSTEM_ENVIRONMENT_PATH]
             [NO_CMAKE_PACKAGE_REGISTRY]
             [NO_CMAKE_BUILDS_PATH] # Deprecated; does nothing.
             [NO_CMAKE_SYSTEM_PATH]
             [NO_CMAKE_SYSTEM_PACKAGE_REGISTRY]
             [CMAKE_FIND_ROOT_PATH_BOTH |
              ONLY_CMAKE_FIND_ROOT_PATH |
              NO_CMAKE_FIND_ROOT_PATH])

```
主要用来寻找包：
- REQUIRED 指定必须要此库， 如指定之后未找到这个库会报错
- PATHS 可以用来指定寻找包的路径

找到该package之后，会自动生成以下变量
```
${"PACKAGENAME"_FOUND}
${"PACKAGENAME"_INCLUDE_DIRS}
${"PACKAGENAME"_LIBRARIES or LIBS}
${"PACKAGENAME"_DEFINITIONS}
```
Ex.
```cmake
find_packages(OpenCV REQUIRED PATHS /usr/local/)
```




#### add_executable() 告诉CMake我的构建目标（生成可执行文件）

`add_executable(<name> [WIN32] [MACOSX_BUNDLE] [EXCLUDE_FROM_ALL] source1 [source2 …])`

Ex

`add_executable(${PROJECT_NAME} ${hello_src})` : 主要是使用hello_src中的源文件来生成可执行文件

这个情况是src下的源文件刚好为main时候， 如果main文件不在src下而在外头， 则应该使用

`add_executable(${PROJECT_NAME} main.cpp ${hello_src})` 意思就是将main.cpp利用hello_src变量中的源文件 建构成PROJECT_NAME 的可执行文件



#### target_include_directories 添加include路径 文件夹到target

```cmake
target_include_directories(<target> [SYSTEM] [BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```

- <target> 目标文件必须是已经被add_excutable or add_library 创建过的， 并且不能被IMPORTED修饰
- <INTERFACE|PUBLIC|PRIVATE> : 关键字INTERFACE，PUBLIC和PRIVATE指定它后面参数的作用域。PRIVATE和PUBLIC项会填充targe目标文件的INCLUDE_DIRECTORIES属性。PUBLIC和INTERFACE项会填充target目标文件的INTERFACE_INCLUDE_DIRECTORIES属性。随后的参数指定包含目录。



#### target_link_libraries 告诉CMakeMake 目标要链接哪一个库文件

`arget_link_libraries(<target> [item1 [item2 [...]]] [[debug|optimized|general] <item>] …)`

Ex.

`target_link_libraries(${PROJECT_NAME} util)` : 主要是名字叫${PROJECT_NAME} 这个target需要链接util这个库



EX.

```cmake
#先将importer源文件都存储到IMPORTER_SOURCES这个变量
set(IMPORTER_SOURCES
  NvOnnxParser.cpp
  ModelImporter.cpp
  builtin_op_importers.cpp
  onnx2trt_utils.cpp
  ShapedWeights.cpp
  OnnxAttrs.cpp
)

add_library(nvonnxparser SHARED ${IMPORTER_SOURCES}) #建立动态库
target_include_directories(nvonnxparser PUBLIC ${CUDA_INCLUDE_DIRS} ${ONNX_INCLUDE_DIRS} ${TENSORRT_INCLUDE_DIR} ${CUDNN_INCLUDE_DIR}) #库作为目标， 作用域为Public
target_link_libraries(nvonnxparser PUBLIC onnx_proto nvonnxparser_plugin ${PROTOBUF_LIBRARY} ${CUDNN_LIBRARY} ${TENSORRT_LIBRARY}) 
#目标nvonnxparser连接onnx_proto nvonnxparser_plugin 等库
```





#### 传递FLAGS给C++编译器

如果想告诉CMake 在生成Makefile里告诉编译器启用C++11

```cmake
set(CMAKE_CXX_COMPILER      "clang++" )         # 显示指定使用的C++编译器

set(CMAKE_CXX_FLAGS   "-std=c++11")             # c++11
set(CMAKE_CXX_FLAGS   "-g")                     # 调试信息
set(CMAKE_CXX_FLAGS   "-Wall")                  # 开启所有警告

set(CMAKE_CXX_FLAGS_DEBUG   "-O0" )             # 调试包不优化
set(CMAKE_CXX_FLAGS_RELEASE "-O2 -DNDEBUG " )   # release包优化
```

- `CMAKE_CXX_FLAGS` 是CMake传给C++编译器的编译选项，通过设置这个值就好比 `g++ -std=c++11 -g -Wall`
- `CMAKE_CXX_FLAGS_DEBUG` 是除了`CMAKE_CXX_FLAGS`外，在Debug配置下，额外的参数
- `CMAKE_CXX_FLAGS_RELEASE` 同理，是除了`CMAKE_CXX_FLAGS`外，在Release配置下，额外的参数



### 开始建构

#### 范例

```c++
/Users/sunyongjian1/codes/local_codes/cmake_test tree
.
├── CMakeLists.txt
├── include
│   └── util.h
├── lib
│   └── libutil.a
└── src
    └── main.cpp
```

三个文件夹: include, lib, src分别存放包含文件，库文件，源文件；一个CMakeLists.txt脚本。下面我的任务是编写这个脚本，使得工程包含util.h头文件，编译main.cpp, 链接libutil.a, 最终生成一个可执行文件hello.



将最一开始

```cmake
cmake_minimum_required ( VERSION 3.0)

project(hello)

include_directories(${CMAKE_CURRENT_LIST_DIR}/include)

link_directories(${CMAKE_CURRENT_LIST_DIR}/lib)

aux_source_directory(${CMAKE_CURRENT_LIST_DIR}/src ${hello_src})

add_executable(${PROJECT_NAME} ${hello_src})

target_link_libraries(${PROJECT_NAME} util)

set(CMAKE_CXX_COMPILER      "clang++" )         # 显示指定使用的C++编译器

set(CMAKE_CXX_FLAGS   "-std=c++11")             # c++11
set(CMAKE_CXX_FLAGS   "-g")                     # 调试信息
set(CMAKE_CXX_FLAGS   "-Wall")                  # 开启所有警告

set(CMAKE_CXX_FLAGS_DEBUG   "-O0" )             # 调试包不优化
set(CMAKE_CXX_FLAGS_RELEASE "-O2 -DNDEBUG " )   # release包优化
```



#### build 方法

在CMakeLists.txt所在目录，新建build目录, 例如

```shell
#假设当前的目录与CMakeLists.txt 一样

mkdir build
cd build
cmake ../ 
#接着开始生成Makefile

make #开始build

#接着会生成一堆文件在build中就是编译结束的

```

------

> <h3 align=center id="">进阶</h3>

#### file()

**GLOB**收集文件

`file(GLOB <variable>[LIST_DIRECTORIES true|false] [RELATIVE <path>] [<globbing-expressions>...])`

GLOB命令主要是将所有匹配<globbing-expressions>, (这是可选的， 如果没写则无法匹配)的文件

Ex.

```
file(GLOB CPP_SRC src/*.cpp)
```

主要是将src资料下所有cpp文件存储到CPP_SRC变量中



#### list()

官方 https://cmake.org/cmake/help/v3.16/command/list.html?highlight=list

主要可分为read / search / modification / ordering作用

每种都有各自的操作

Ex.

```
list(APPEND <list> [<element> ...])
```





#### option() & add_definition()

两者通常搭配使用

```cmake
# Provides an option for the user to select as ON or OFF. If no initial <value> is provided, OFF is used. If <variable> is already set as a normal variable then the command does nothing
option(<variable> "<help_text>" [value])

# Add -D define flags to the compilation of source files.
add_definitions(-DFOO -DBAR ...)
```

添加编译参数

1. option增加选项, 然后add_definition增加编译参数

   ```cmake
   option (TEST_DEBUG "option for debug" OFF)
   
   if (TEST_DEBUG)
   add_definition(-DTEST_DEBUG)
   endif()
   ```

2. 在Cmake编译make的时候就可以设置参数

   ```cmake
   cmake -DTEST_DEBUG=ON
   ```

3. 原始码cpp中使用该定义

   ```c++
   #include "test.h"
   ifdef TEST_DEBUG
   ........
   endif
   ```



add_definition() 内的参数也可以直接为参数添加default值

Ex.

```cmake
if(USE_ARM64)
    add_definitions (-std=c++11 -O2 -fomit-frame-pointer -g -Wall)
    MESSAGE (STATUS "Build Option: -std=c++11 -O2 -fomit-frame-pointer -g -Wall")
else()
```

------

> <h3 align=center id="">输出指定目录</h3>
>
> 参考 [https://zh.wikibooks.org/zh-tw/CMake_%E5%85%A5%E9%96%80/%E8%BC%B8%E5%87%BA%E4%BD%8D%E7%BD%AE%E8%88%87%E5%AE%89%E8%A3%9D](https://zh.wikibooks.org/zh-tw/CMake_入門/輸出位置與安裝)
>
> 主要设定编译完成后的这种库的存放位置





**CMAKE_LIBRARY_OUTPUT_DIRECTORY**

```
CMAKE_LIBRARY_OUTPUT_DIRECTORY
```

可以存储 当动态库(.so)建立完成时 所存放的路径

Ex.

```cmake
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
```

这说明将生成的库存储在当前目录下的lib文件夹中， 可取代` LIBRARY_OUTPUT_DIRECTORY`

**CMAKE_ARCHIVE_OUTPUT_DIRECTORY**

可以以存储 当静态库(.a)建立完成时， 所存放的路径

```
CMAKE_ARCHIVE_OUTPUT_DIRECTORY
```

Ex.

```cmake
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
```



**CMAKE_RUNTIME_OUTPUT_DIRECTORY**

```cmake
CMAKE_RUNTIME_OUTPUT_DIRECTORY
```

用来存储 编译完成的执行档案， 例如Win下的exe， dll档案

```
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
```





------



<h3 align=center id="">CUDA 编译</h3>

#### CUDA编译简单范本

```cmake
# 按惯例，cmake的版本
CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
# 项目名称
PROJECT(AD-Census)
# cmake寻找cuda，这个要现在系统里面装好cuda，设置好cuda的环境参数啥的
FIND_PACKAGE(CUDA REQUIRED)
# C++和CUDA的编译参数，可选。
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
SET(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS};-gencode arch=compute_61,code=sm_61;-std=c++11;)
# 头文件路径，按需
INCLUDE_DIRECTORIES(
    ./containers)
# 库文件路径，按需
LINK_DIRECTORIES(/usr/lib
    /usr/local/lib)
# 主要就是这个，教cmake去找nvcc来编译这些东西
CUDA_ADD_EXECUTABLE(ad-census
    main.cu
    ./containers/device_memory.cpp
    ./containers/initialization.cpp
)
# 链接外部库，按需
TARGET_LINK_LIBRARIES(ad-census
    某个库的名字)
    
    
    
    	
```





#### CUDA 编译函数



**CUDA_INCLUDE_DIRS()**

包含CUDA平台的头文件， 会自动添加到`CUDA_ADD_EXECUTABLE` 以及 `CUDA_ADD_LIBRARY`中





**CUDA_ADD_EXECUTABLE（）**

指定要生成的执行档， 与add_executable区别是增加了可编译CUDA C file， 主要使用NVCC编译以及host compiler, 修改flags并不会影响那些被nvcc编译的档案， 注意flags应该在call `CUDA_ADD_EXECUTABLE`,
  ` CUDA_ADD_LIBRARY` or `CUDA_WRAP_SRCS` 之前就需要修改





------

<h3 align=center>参考资料</h3>

 [https://www.hahack.com/codes/cmake/](https://www.hahack.com/codes/cmake/)

 [https://zh.wikibooks.org/wiki/CMake_%E5%85%A5%E9%96%80](https://zh.wikibooks.org/wiki/CMake_入門)

 [https://blog.51cto.com/9291927/2115399](https://blog.51cto.com/9291927/2115399)

 [https://juejin.im/post/5b3ecfef6fb9a04f8c5ebab5](https://juejin.im/post/5b3ecfef6fb9a04f8c5ebab5)

[https://elloop.github.io/tools/2016-04-10/learning-cmake-2-commands](https://elloop.github.io/tools/2016-04-10/learning-cmake-2-commands)

CMake官方 https://cmake.org/cmake/help/latest/manual/cmake-commands.7.html

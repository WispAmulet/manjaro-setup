## C

1. 在 VScode 中安装 C/C++, CMake, CMake Tools 插件

2. C/C++ 插件默认使用的 formatter 报错

   ```
   ~/.vscode-oss/extensions/ms-vscode.cpptools-0.26.2-insiders/bin/../LLVM/bin/clang-format: error while loading shared libraries: libtinfo.so.5: cannot open shared object file: No such file or directory
   ```

   查询了一下仓库中没有 `libtinfo.so.5` 这个包，只有一个叫作 `ncurses` 的包提供了 `usr/lib/libtinfo.so` 和 `usr/lib/libtinfo.so.6`

   A: 创建一个 link，把调用的文件（但实际不存在，远程仓库也不提供）指向本地存在的文件

   ```sh
   # ln 命令用来创建一个 link，分为 hard links 与 symbolic links
   # {-s --symbolic} 表示创建的是 symbolic link
   # 第一个参数 /usr/lib/libtinfo.so.6 表示 target
   # 第二个参数表示在 /usr/lib/ 目录下创建了一个叫 libtinfo.so.5 的 link，指向  target
   sudo ln -s /usr/lib/libtinfo.so.6 /usr/lib/libtinfo.so.5

   # 可使用 unlink 命令删除 link
   unlink /usr/lib/libtinfo.so.5
   # 使用 ll 命令查看时会有箭头表示 link 的指向，类似 libtinfo.so.5 -> /usr/lib/ libtinfo.so.6

   # 测试该命令时发现有时候会出现以下错误，比如以下目录结构：
   # -- test/
   #   |-- des/
   #   |   `-- b.sh
   #   |-- a.sh
   #
   pwd # > ~/test/des
   ln -s ~/test/a.sh ~/test/des/a.sh # 正常
   ln -s ../a.sh a.sh # 正常
   pwd # > ~/test
   ln -s a.sh ./des/a.sh # 报错 too many levels of symbolic links: a.sh
   # 结论：
   # 最好以绝对路径表示
   # 或者切换到需要创建的 link 应该存在的位置
   ```

3. 创建工程目录

   ```sh
   # -- hello-world/
   #   |-- src/
   #   |   `-- Main.cpp
   #   |-- CMakeLists.txt
   ```

   ```cpp
   // src/Main.cpp
   #include <iostream>

   int main()
   {
       std::cout << "Hello World!" << std::endl;
       std::cin.get();
   }
   ```

   TODO: 根据步骤 2 之后 formatter 可以使用（但是功能好弱的样子。。）

   不过仍然会提示

   ```
   Can't create IntelliSense client for ...
   Custom configuration provider 'CMake Tools' registered
   ```

   ```cmake
   # CMakeLists.txt
   cmake_minimum_required (VERSION 3.5)

   project (HelloWorld)

   set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Werror -std=c++14")
   set (source_dir "${PROJECT_SOURCE_DIR}/src/")

   file (GLOB source_files "${source_dir}/*.cpp")

   add_executable (${PROJECT_NAME} ${source_files})
   ```

   插件会根据此配置自动生成文件，默认在根目录下新建 `build` 文件夹中

   `F1 -> CMake: Build` 会在 `build` 文件夹中生成可执行文件

   ```sh
   ./build/HelloWorld
   # > Hello World!
   ```

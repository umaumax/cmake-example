# cmake example

* 下記のディレクトリでbuild可能
* `./`
* `./hoge`: `libhoge.a`
* `./fuga`: `libfuga.a`
* `./main`: `hoge`,`fuga`に依存

## NOTE
### 文字列の比較は`STREQUAL`
```
if("str" STREQUAL "str")
endif()
```

### buildable child project
* `CMAKE_SOURCE_DIR`ではなく`CMAKE_CURRENT_SOURCE_DIR`を利用するように(`CMAKE_SOURCE_DIR`が変化するため)

### cmake rootかどうか
```
if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_SOURCE_DIR})
endif()
```

* [c \- How to use add\_dependency with CMake to non\-subdirectories? \- Stack Overflow]( https://stackoverflow.com/questions/44538006/how-to-use-add-dependency-with-cmake-to-non-subdirectories )

### PUBLIC/PRIVATE/INTERFACE
* [CMake: target\_link\_libraries\(PUBLIC/PRIVATE/INTERFACE\) の実践的な解説 \- Qiita]( https://qiita.com/chapuni/items/7ddffb46fa327c5bef43 )
  * use `PRVATE`

### add_subdirectory()
* [CMakeで大きめのプロジェクトを構成するためのメモ \- Qiita]( https://qiita.com/shibukawa/items/af7ff2e1178fbb3ab6de )

> 1. 基本的に今いるフォルダよりも上のフォルダを参照するのはイレギュラー(という方針)
> 2. イレギュラーな使い方をする場合は、読み込むフォルダだけではなく、作業フォルダも第二引数で指定する必要がある
> 3. 基本的に、1つのライブラリに対して、それに依存する親がたくさんいても1度だけ呼び出す。複数回呼び出すとエラーになる

### cpackとは?
TODO

### cmakeで作成されたmakeのinstallのファイル確認方法

[\[cmake\-developers\] Getting a list of files to install and their destination]( https://cmake.org/pipermail/cmake-developers/2013-September/019868.html )

> 1) use CPack
> 2) use make install to tmp and deploy

2番の簡単な方法
```
cmake -DCMAKE_INSTALL_PREFIX=$(mktemp -d install) ..
```

純粋なmakefileの場合には`make -n`で確認可能

### find_package()

[find\_packageの動作 \- Qiita]( https://qiita.com/osamu0329/items/bd3d1e50edf37c277fa9 )

* Moduleモード
  * file
    * Find<package>.cmake
  * dir
    * CMakeのインストール先 (e.g. /usr/local/share/cmake-2.8/Modules)
    * 環境変数 CMAKE_MODULE_PATH に設定したパス

* Configモード
  * file
    * <package>Config.cmake または <lower-case-package>-config.cmake
  * dir
    * `find_package()`の`HINTS`や`PATHS`

* [お手軽な xxx\-config\.cmake の作成方法 \- Qiita]( https://qiita.com/osamu0329/items/134de918c0ffa7f0557b )
  * ConfigモードのほうがHINTSでパスを指定できるのおすすめ

### あるcmakeが生成したファイルを他のcmakeで利用したい
* [\[CMake\] ソースファイルプロパティは同じディレクトリのターゲットにしか見えない \- Qiita]( https://qiita.com/kktk-KO/items/201cfd0ee744059f39ed )

`make`実行時にcommandが実行されることとなるが，依存関係となるため，下記の例では`bar`単体のcmakeは失敗することとなるので注意(コメント参照)

`foo/CMakeLists.txt`
```
add_custom_command(OUTPUT a.txt COMMAND touch a.txt)
add_custom_target(FOO_GENERATE_ATXT SOURCES a.txt) # このSOURCESは上記のOUTPUTに対応するcustom_commandを呼び出す
```

`bar/CMakeLists.txt`
```
add_custom_target(BAR_TARGET ALL DEPENDS FOO_GENERATE_ATXT) # 単体でbuildしたときに，make時にmake[2]: *** No rule to make target `../FOO_GENERATE_ATXT', needed by `CMakeFiles/BAR_TARGET'.  Stop.
# or
add_dependencies(bar FOO_GENERATE_ATXT) # 単体でbuildしたときに，cmake時にThe dependency target "FOO_GENERATE_ATXT" of target "bar" does not exist.
```

`bar/CMakeLists.txt`には下記を先頭に追加すれば，単体でbuildするときに依存先の解決ができる
```
if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_SOURCE_DIR})
  add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../foo
                   ${CMAKE_CURRENT_BINARY_DIR}/foo)
endif()
```

### ExternalProject_Add git --depth 1
```
GIT_SHALLOW true # --depth 1
```

### googletest
```
include(ExternalProject)
ExternalProject_Add(googletest
  GIT_REPOSITORY "https://github.com/google/googletest.git"
  GIT_TAG master
  GIT_SHALLOW true # --depth 1
  # NOTE: for local install
  BUILD_COMMAND cd ../googletest-build && ${CMAKE_COMMAND} -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/lib ../googletest
)
```

* しかし，googletestはinstallを利用しなくてもある程度簡単にファイルの指定が可能(下記参照)
* その場合には，下記の変更を加えること
```
# remove BUILD_COMMAND
INSTALL_COMMAND ""
```

### ExternalProject_Get_Property
* NOTE: 取得したい値と変数が1対1対応しているので注意
```
ExternalProject_Get_Property(googletest source_dir)
include_directories(${source_dir}/include)
```

```
ExternalProject_Get_Property(googletest binary_dir)
message("binary_dir:${binary_dir}")

add_library(gtest STATIC IMPORTED)
set_property(TARGET gtest PROPERTY IMPORTED_LOCATION ${binary_dir}/libgtest.a)
add_library(gtest_main STATIC IMPORTED)
set_property(TARGET gtest_main
             PROPERTY IMPORTED_LOCATION ${binary_dir}/libgtest_main.a)
add_library(gmock STATIC IMPORTED)
set_property(TARGET gmock PROPERTY IMPORTED_LOCATION ${binary_dir}/libgmock.a)
add_library(gmock_main STATIC IMPORTED)
set_property(TARGET gmock_main
             PROPERTY IMPORTED_LOCATION ${binary_dir}/libgmock_main.a)
```

### target名に対する出力ファイル名を変更する
```
SET_TARGET_PROPERTIES(${TARGET} PROPERTIES OUTPUT_NAME ${NEW_NAME})
```

### scope

* `find_package`は同じscopeで実行しないと意味がない
  * つまり，`function`で呼び出すのではなく，`macro`で呼び出せば所望の動作となる
  * add_subdirectory()の先は呼び出し元のスコープと同じだと考えられる

### include(xxx.cmake)で実装していた部分を関数にしたときに，動作が変わった

[How to set the global variable in a function for cmake? \- Stack Overflow]( https://stackoverflow.com/questions/10031953/how-to-set-the-global-variable-in-a-function-for-cmake )
```
set(GTEST_FOUND ${GTEST_FOUND} PARENT_SCOPE)
set(GTEST_INCLUDE_DIRS ${GTEST_INCLUDE_DIRS} PARENT_SCOPE)
set(GMOCK_INCLUDE_DIRS ${GMOCK_INCLUDE_DIRS} PARENT_SCOPE)
set(GTEST_LIBRARIES ${GTEST_LIBRARIES} PARENT_SCOPE)
```
もしくは，CACHEとして扱う
もしくは，`macro`を利用する
[c\+\+ \- cmake \- weird side effects when find\_package is called from function as opposed to global scope \- Stack Overflow]( https://stackoverflow.com/questions/36724768/cmake-weird-side-effects-when-find-package-is-called-from-function-as-opposed )

### cmakeでは""で変数を囲む場合と囲まない場合で挙動が大きく異なる
```
if(NOT ${GTEST_FOUND})
  # if(NOT)となるので，false
endif()
if(NOT "${GTEST_FOUND}")
  # if(NOT "")となるので，true
endif()

find_package(GTest)
if(NOT "${GTEST_FOUND}")
  # OK
endif()
if(NOT "${GTEST_FOUND}")
  # OK
endif()
```

### 文字列の中身が評価される
```
if("0")
  message("\"0\": true")
else()
  message("\"0\": false")
endif()
if("1")
  message("\"1\": true")
else()
  message("\"1\": false")
endif()
if("FALSE")
  message("\"FALSE\": true")
else()
  message("\"FALSE\": false")
endif()
if("TRUE")
  message("\"TRUE\": true")
else()
  message("\"TRUE\": false")
endif()
if(0)
  message("0: true")
else()
  message("0: false")
endif()
if(1)
  message("1: true")
else()
  message("1: false")
endif()
if(FALSE)
  message("FALSE: true")
else()
  message("FALSE: false")
endif()
if(TRUE)
  message("TRUE: true")
else()
  message("TRUE: false")
endif()
```

```
"0": false
"1": true
"FALSE": false
"TRUE": true
0: false
1: true
FALSE: false
TRUE: true
```

### gtest
* macでは`pthread`不要
```
find_package(GTest)
find_package(Threads)

target_link_libraries(${PROJECT_NAME} ${GTEST_LIBRARIES} ${GMOCK_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})
```

* ubuntuでは`pthread`必要
```
find_package(GTest)
target_link_libraries(${PROJECT_NAME} ${GTEST_LIBRARIES} ${GMOCK_LIBRARIES})
```

----

## FMI
* [CMake: 便利なコマンド・変数 \- Qiita]( https://qiita.com/mrk_21/items/5e7ca775b463a4141a58 )
* [CMakeスクリプトを作成する際のガイドライン \- Qiita]( https://qiita.com/shohirose/items/5b406f060cd5557814e9 )
  * refactorに役立つ
  * 非推奨/推奨な例が記述されているため，わかりやすい
* [CMake: target\_link\_libraries\(PUBLIC/PRIVATE/INTERFACE\) の実践的な解説 \- Qiita]( https://qiita.com/chapuni/items/7ddffb46fa327c5bef43 )

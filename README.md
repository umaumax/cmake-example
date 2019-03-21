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

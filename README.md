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


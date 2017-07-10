================
4.外部ライブラリ
================

4.1.想定するシナリオ
====================

　C のソースの中身に入っていく前に、ちょっと寄り道をします。PHP Extension を開発する必要性が生じるケースとして代表的な、外部ライブラリの利用をシナリオとして想定します。このために、別途ダミーの C ライブラリを用意しました。以下の手順でビルドしてください。

.. code-block:: bash
  :emphasize-lines: 1-4

  ~/php/ext/my_ext$ cd
  ~$ git clone https://github.com/hotta/my_lib.git
  ~$ cd my_lib
  ~/my_lib$ make

　これにより、~/my_lib 配下に libmy_lib.so という共有ライブラリファイルが作られます。mylib_test という名前のテスト用プログラムも用意していますので、試しに使ってみてください。

.. code-block:: bash
  :emphasize-lines: 1,7

  ~/my_lib$ ./mylib_test
  my_echo_int(123)
  my_echo_double(123.456000)
  my_echo_string("hello world")
  my_add_return_int(30, 40) = 70
  my_add_return_str("hello", "world") = "helloworld"
  ~/my_lib$ sudo tail -1 /var/log/messages
  Jun  7 11:45:14 php-reform MY_EXT: MY_LOGGER_TEST

　mylib_test コマンドは libmy_lib.so にある API 関数を一つずつ呼び出します。mylib_test のソース test.c は以下の通りです。

my_lib/test/test.c::

  #include  <stdio.h>
  #include  <malloc.h>
  #include  "my_lib.h"
  
  #define SYSLOG_MSG  "MY_LOGGER_TEST"
  
  int main(int argc, char **argv)
  {
    my_echo_int(123);
    my_echo_double(123.456);
    my_echo_string("hello world");
    printf("my_add_return_int(30, 40) = %d\n", my_add_return_int(30, 40));
  
    char *add;
    add = my_add_return_str("hello", "world");
    printf("my_add_return_str(\"hello\", \"world\") = \"%s\"\n", add);
    free(add);
  
    my_logger(SYSLOG_MSG);
    return  0;
  }

　``my_`` で始まる関数の実体は components ディレクトリの中にあります。C から呼ぶ場合は上記の通りですが、今回はこれらの関数を PHP から呼ぶための PHP Extension を作成してみます。

4.2.ライブラリの構成
====================

　C で書かれた外部ライブラリの機能を呼び出すためには、以下のことが前提となります。

* 参照したい共有ライブラリが、外部に関数名を公開（エクスポート）していること。
* 共有ライブラリが公開している関数のインターフェイス（呼び出し方法）が、C のヘッダファイルとして提供されていること。

　ライブラリが公開している関数シンボルは、objdump コマンドで確認できます。

.. code-block:: bash
  :emphasize-lines: 1

  ~/my_lib$ objdump -T libmy_lib.so
  
  libmy_lib.so:     ファイル形式 elf64-x86-64
  
  DYNAMIC SYMBOL TABLE:
  00000000000007b0 l    d  .init  0000000000000000              .init
  0000000000000000  w   D  *UND*  0000000000000000              _ITM_deregisterTMCloneTable
  0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 strcpy
  0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 strlen
  0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 printf
  0000000000000000  w   D  *UND*  0000000000000000              __gmon_start__
  0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 malloc
  0000000000000000  w   D  *UND*  0000000000000000              _Jv_RegisterClasses
  0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 vsyslog
  0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 openlog
  0000000000000000  w   D  *UND*  0000000000000000              _ITM_registerTMCloneTable
  0000000000000000  w   DF *UND*  0000000000000000  GLIBC_2.2.5 __cxa_finalize
  0000000000201058 g    D  .got.plt       0000000000000000  Base        _edata
  0000000000201060 g    D  .bss   0000000000000000  Base        _end
  0000000000000995 g    DF .text  0000000000000026  Base        my_echo_string
  0000000000000a80 g    DF .text  00000000000000cd  Base        my_logger
  00000000000009ed g    DF .text  0000000000000014  Base        my_add_return_int
  00000000000009bb g    DF .text  0000000000000032  Base        my_add_int
  0000000000000945 g    DF .text  0000000000000023  Base        my_echo_int
  0000000000201058 g    D  .bss   0000000000000000  Base        __bss_start
  00000000000007b0 g    DF .init  0000000000000000  Base        _init
  0000000000000b50 g    DF .fini  0000000000000000  Base        _fini
  0000000000000968 g    DF .text  000000000000002d  Base        my_echo_double
  0000000000000a01 g    DF .text  000000000000007f  Base        my_add_return_str

　第２カラムが ``g`` になっているのが公開されているグルーバルシンボルです。また第４カラムが ``.text`` になっているのは、このシンボルがプログラム部分にある（≒関数名である）ことを示します。test.c で呼び出している関数群が公開されているのがわかります。

　ちなみに第４カラムが ``*UND*`` (undefined) のものは、この（バイナリの）中では定義されていない外部シンボルへの参照を表しており、GLIBC に含まれる strcpy(3) や printf(3) といった関数が内部で使われ（外部参照され）ているのがわかります。

　なお、関数のインターフェイスを提示するヘッダファイルの中身は、以下のようになっています。

.. code-block:: bash
  :emphasize-lines: 1

  ~/my_lib$ cat my_lib.h
  extern  void  my_echo_int(int arg);
  extern  void  my_echo_double(double arg);
  extern  void  my_echo_string(char *arg);
  extern  void  my_add_int(int a, int b);
  extern  int   my_add_return_int(int a, int b);
  extern  char *my_add_return_str(char *a, char *b);
  extern  void  my_logger(const char *fmt, ...);

4.3.外部ライブラリへの依存を追加
================================

　PHP のソースツリーから見ると ~/my_lib/{my_lib.h,libmy_lib.so} は知らない存在なので、これをコンパイラやリンカに教えてやる必要があります。config.m4 を以下のように修正します。

.. code-block:: bash
  :emphasize-lines: 1

  ~/php/ext/my_ext$ diff /tmp/config.m4 config.m4
  43,44c43,45
  <   dnl # --with-my_ext -> add include path
  <   dnl PHP_ADD_INCLUDE($MY_EXT_DIR/include)
  ---
  >   MY_LIB_DIR=/home/vagrant/my_lib
  >   INCLUDE_DIR=$MY_LIB_DIR
  >   PHP_LIBDIR=$MY_LIB_DIR
  46,48c47,51
  <   dnl # --with-my_ext -> check for lib and symbol presence
  <   dnl LIBNAME=my_ext # you may want to change this
  <   dnl LIBSYMBOL=my_ext # you most likely want to change this
  ---
  >   AC_CHECK_HEADER([$INCLUDE_DIR/my_lib.h],
  >     [],
  >     [AC_MSG_ERROR(["$INCLUDE_DIR/my_lib.h" が見つかりません])]
  >   )
  >   PHP_ADD_INCLUDE($INCLUDE_DIR)
  50,60c53,70
  <   dnl PHP_CHECK_LIBRARY($LIBNAME,$LIBSYMBOL,
  <   dnl [
  <   dnl   PHP_ADD_LIBRARY_WITH_PATH($LIBNAME, $MY_EXT_DIR/$PHP_LIBDIR, MY_EXT_SHARED_LIBADD)
  <   dnl   AC_DEFINE(HAVE_MY_EXTLIB,1,[ ])
  <   dnl ],[
  <   dnl   AC_MSG_ERROR([wrong my_ext lib version or lib not found])
  <   dnl ],[
  <   dnl   -L$MY_EXT_DIR/$PHP_LIBDIR -lm
  <   dnl ])
  <   dnl
  <   dnl PHP_SUBST(MY_EXT_SHARED_LIBADD)
  ---
  >   # --with-my_ext -> add include path
  >   PHP_ADD_INCLUDE($MY_EXT_DIR/include)
  >
  >   # --with-my_ext -> check for lib and symbol presence
  >   LIBNAME=my_lib
  >   LIBSYMBOL=my_echo_int
  >
  >   PHP_CHECK_LIBRARY($LIBNAME,$LIBSYMBOL,
  >   [
  >     PHP_ADD_LIBRARY_WITH_PATH($LIBNAME, $MY_EXT_DIR/$PHP_LIBDIR, MY_EXT_SHARED_LIBADD)
  >     AC_DEFINE(HAVE_MY_EXTLIB,1,[ ])
  >   ],[
  >     AC_MSG_ERROR([libmy_lib.so が見つからないか、バージョンが誤っています])
  >   ],[
  >     -L$MY_EXT_DIR/$PHP_LIBDIR -lm
  >   ])
  >
  >   PHP_SUBST(MY_EXT_SHARED_LIBADD)

　前半の変更は my_lib.h を見つけるためです。後半の変更は、libmy_lib.so を見つけ、その中でさらに my_echo_int 関数の存在を確認しています。[1]_

　再度 phpize からやり直します。

.. code-block:: bash
  :emphasize-lines: 1,6,8

  ~/php/ext/my_ext$ phpize
  Configuring for:
  PHP Api Version:         20160303
  Zend Module Api No:      20160303
  Zend Extension Api No:   320160303
  ~/php/ext/my_ext$ ./configure --enable-my_ext
  （中略）
  configure: creating ./config.status
  config.status: creating config.h

　これで作成された config.h は、my_ext.c でインクルードして使用します。

.. [1] ここで使われているマクロ群は、PHP 公式マニュアルの `UNIX 用のビルドシステム: config.m4 <http://php.net/manual/ja/internals2.buildsys.configunix.php>`_ に一部記載があります。ここに記載のない AC_CHECK_HEADER などは、 `GNU Autoconf/Automake/Libtool（でびあんぐる監訳） <https://www.amazon.co.jp/Autoconf-Automake-Libtool-Gary-Vaughan/dp/4274064115>`_ が詳しいです。

==============
6.ソースの改造
==============

6.1.PHP の関数仕様
==================

　まず手始めに、外部ライブラリ libmy_lib.so に入っている my_echo_int() 関数への PHP インターフェイスを提供してみましょう。まず PHP 言語レベルで、関数の仕様を以下のように定めることとします。

.. php:function:: void my_echo_int( int $arg )
  コンソールに関数名と引数を表示します。

　`my_echo_int` は C の関数名と衝突しそうですが、実は問題ありません。関数エントリは PHP_FE マクロで定義しますが、:ref:`zend_function_entry` で述べたように、 ``PHP_FE(関数名)`` はプリプロセッサにより ``vif_関数名`` に展開されるため、名前の衝突が起こらないようになっています。


6.2.関数エントリの登録
======================

　まず、my_echo_int() の外部定義がある、 ``my_lib.h`` を ``my_ext.c`` からインクルードします。 次に、zend_function_entry に my_echo_int を追加します。現時点で、変更箇所は以下の通りです。

.. code-block:: bash
  :emphasize-lines: 1,10,18

  ~/php/ext/my_ext$ git diff my_ext.c
  diff --git a/my_ext.c b/my_ext.c
  index 864175d..eeb2e77 100644
  --- a/my_ext.c
  +++ b/my_ext.c
  @@ -26,6 +26,7 @@
   #include "php_ini.h"
   #include "ext/standard/info.h"
   #include "php_my_ext.h"
  +#include "my_lib.h"
  
   /* If you declare any globals in php_my_ext.h uncomment this:
   ZEND_DECLARE_MODULE_GLOBALS(my_ext)
  @@ -147,6 +148,7 @@ PHP_MINFO_FUNCTION(my_ext)
    */
   const zend_function_entry my_ext_functions[] = {
          PHP_FE(confirm_my_ext_compiled, NULL)     /* For testing, remove later. */
  +       PHP_FE(my_echo_int,             NULL)
          PHP_FE_END      /* Must be the last line in my_ext_functions[] */
   };
   /* }}} */


6.3.リフレクション情報の登録
============================

　PHP_FE マクロの第二引数は、 `Reflection API <http://php.net/manual/ja/book.reflection.php>`_ に対して関数の引数の情報を提供するのに使われます。指定しなくても動きますが、作成した Extension を外部に公開する場合は必要になりますので、以下の要領で指定してください。

.. code-block:: c
  :emphasize-lines: 3

   const zend_function_entry my_ext_functions[] = {
          PHP_FE(confirm_my_ext_compiled, NULL)
  +       PHP_FE(my_echo_int,             arginfo_my_echo_int)
          PHP_FE_END
   };

　 ``arginfo_`` プリフィックスは、引数定義を表すための慣習として付加します。続いて、`zend_function_entry` の定義より前に、以下の要領で ``arginfo_my_echo_int`` の定義を追加します。

::

  ZEND_BEGIN_ARG_INFO_EX(arginfo_my_echo_int, 0, 0, 1)  // ……（1）
  ZEND_ARG_INFO(0, int_arg1)                            // ……（2）
  ZEND_END_ARG_INFO()                                   // ……（3）

(1).ZEND_BEGIN_ARG_INFO_EX マクロ
---------------------------------

　以下の４つの引数を取ります。

.. list-table::
  :widths: 15 80
  :header-rows: 1

  * - 引数番号
    - 説明
  * - 1
    - 定義名（PHP_FE() マクロの第一引数として指定する文字列）
  * - 2
    - 未使用
  * - 3
    - 返却フラグ（1.参照を返す　0.受け取るだけ）
  * - 4
    - 必須の引数の数

(2).ZEND_ARG_INFO マクロ
------------------------

　引数の数の分だけ列挙します。それぞれは以下の２個の引数を受け取ります。


.. list-table::
  :widths: 15 80
  :header-rows: 1

  * - 引数番号
    - 説明
  * - 1
    - 1.参照渡し　0.値渡し
  * - 2
    - 仮引数名

(3).ZEND_END_ARG_INFO マクロ
----------------------------

　引数の終了を表します。

利用例
------

　  `mb_convert_encoding <http://php.net/manual/ja/function.mb-convert-encoding.php>`_ を例に取り、使い方を見てみましょう。この関数は以下のように定義されています。

.. php:function:: string mb_convert_encoding ( string $str , \
  string $to_encoding [, mixed $from_encoding = mb_internal_encoding() ] )
  文字列 strの文字エンコーディングを、 オプションで指定した \
  from_encoding から to_encoding に変換します。

　php/ext/mbstring/mbstring.c では、以下のように引数の定義がなされています。

.. code-block:: c

  ZEND_BEGIN_ARG_INFO_EX(arginfo_mb_convert_encoding, 0, 0, 2)
    ZEND_ARG_INFO(0, str)
    ZEND_ARG_INFO(0, to)
    ZEND_ARG_INFO(0, from)
  ZEND_END_ARG_INFO()
  ...
  PHP_FE(mb_convert_encoding,     arginfo_mb_convert_encoding)

　 PHP_FE および ZEND_ARG_INFO マクロで行った引数定義は、 `ReflectionFunction クラス <http://php.net/manual/ja/class.reflectionfunction.php>`_ を使うことで参照が可能です。

.. code-block:: bash
  :emphasize-lines: 1,7

  ~$ cat reflection_test.php
  <?php
  $reffunc = new ReflectionFunction('mb_convert_encoding');
  foreach ($reffunc->getParameters() as $arg) {
    print $arg . PHP_EOL;
  }
  ~$ php reflection_test.php
  Parameter #0 [ <required> $str ]
  Parameter #1 [ <required> $to ]
  Parameter #2 [ <optional> $from ]


6.4.関数本体の追加
==================

　zend_function_entry の宣言より前に、関数本体を以下のように追加します。

.. code-block:: bash
  :emphasize-lines: 1,10-16

  ~/php/ext/my_ext$ git diff my_ext.c
  diff --git a/my_ext.c b/my_ext.c
  index eeb2e77..16db606 100644
  --- a/my_ext.c
  +++ b/my_ext.c
  @@ -73,6 +73,13 @@ PHP_FUNCTION(confirm_my_ext_compiled)
      follow this convention for the convenience of others editing your code.
   */
  
  +/* {{{ proto void my_echo_int(int arg)
  +   コンソールに関数名と引数を表示します。
  +*/
  +PHP_FUNCTION(my_echo_int)
  +{
  +}
  +/* }}} */
  
   /* {{{ php_my_ext_init_globals
    */

　関数の前のプロトタイプ宣言っぽいコメントや、関数全体をコメントレベルで `{{{` ～ `}}}` で囲むスタイルは、PHP の標準に準じた記載です。詳細は :ref:`folding-hooks` を参照してください。なお、成果物をインターネット上で公開する場合はコメントは当然英語で入れるべきですが、本ドキュメントではわかりやすくするために日本語で記載します。

6.5.ビルドと環境設定
====================

　まだ関数の中身がありませんが、いったんこれでビルドしてみましょう。

.. code-block:: bash
  :emphasize-lines: 1

  ~/php/ext/my_ext$ make
  （中略）
  Build complete.
  Don\'t forget to run 'make test'.

　次に実行ですが、その前に、毎回 extension 指定をしなくてもいいように、php.ini に Extension の読み込み設定を入れておきしょう。ソースからビルドした場合の設定ファイルのパスは /usr/local/lib/php.ini です。デフォルトではこのファイルはありませんので新規で作成します。

.. code-block:: bash
  :emphasize-lines: 1,3

  ~/php/ext/my_ext$ sudo vi /usr/local/lib/php.ini
  （ファイルを新規で作成）
  ~/php/ext/my_ext$ cat /usr/local/lib/php.ini
  extension=/home/vagrant/php/ext/my_ext/modules/my_ext.so

　設定を追加したら、モジュールとして読み込まれていることを確認します。

.. code-block:: bash
  :emphasize-lines: 1

  ~/php/ext/my_ext$ php -m | grep my_ext
  my_ext

6.5.動作確認
============

　それでは実行してみましょう。

.. code-block:: bash
  :emphasize-lines: 1

  ~/php/ext/my_ext$ php -r "my_echo_int(100);"

　何も表示されませんが、エラーにもなりませんね。切り分けのために、わざと関数名を間違えてみましょう。

.. code-block:: bash
  :emphasize-lines: 1

  ~/php/ext/my_ext$ php -r "my_echo_int_not_exist(100);"
  Fatal error: Uncaught Error: Call to undefined function my_echo_int_not_exist() in Command line code:1
  Stack trace:
  #0 {main}
    thrown in Command line code on line 1

　今度は見慣れたエラーメッセージが表示されましたね。少なくとも、PHP は関数としては認識してくれているようです。

6.6.関数の中身の実装
====================

　それでは関数本体を実装しましょう。

.. code-block:: bash
  :emphasize-lines: 1,10-18

  ~/php/ext/my_ext$ git diff
  diff --git a/my_ext.c b/my_ext.c
  index 16db606..57b9a8c 100644
  --- a/my_ext.c
  +++ b/my_ext.c
  @@ -78,6 +78,16 @@ PHP_FUNCTION(confirm_my_ext_compiled)
   */
   PHP_FUNCTION(my_echo_int)
   {
  +  zend_long arg;
  +
  +  if (ZEND_NUM_ARGS() < 1 || 1 < ZEND_NUM_ARGS()) {
  +    WRONG_PARAM_COUNT;
  +  }
  +  if (zend_parse_parameters(ZEND_NUM_ARGS(), "l", &arg) == FAILURE) {
  +    return;
  +  }
  +  my_echo_int(arg);
   }
   /* }}} */

　これで再度ビルドして実行します。

.. code-block:: bash
  :emphasize-lines: 1-2

  ~/php/ext/my_ext$ make
  ~/php/ext/my_ext$ php -r "my_echo_int(100);"
  my_echo_int(100)

　これで C のライブラリに制御が渡って、無事に実行されました。

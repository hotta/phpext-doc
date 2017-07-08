==============
6.ソースの改造
==============

6.1.PHP の関数仕様
==================

　まず最初は、libmy_lib.so という外部ライブラリに入っている my_echo_int() 関数への PHP インターフェイスを提供してみましょう。C のソースを修正する前に、まず PHP 言語レベルで関数の仕様を以下のように定めることとします。

.. php:function:: void my_echo_int( int $arg )
  コンソールに関数名と引数を表示します。

　C の関数名と全く同じですが、問題ありません。個々の関数の実体は PHP_FUNCTION マクロで定義しますが、 ``PHP_FUNCTION(関数名)`` は最終的に ``vif_関数名`` に展開（後述）されるため、名前の衝突が起こらないようになっています。


6.2.関数エントリの登録
======================

　まず、my_echo_int() の外部定義がある、 ``my_lib.h`` を ``my_ext.c`` からインクルードします。 次に、zend_function_entry に my_echo_int を追加します。現時点で、変更箇所は以下の通りです。::

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

　PHP_FE マクロの第二引数は、 `Reflection API <http://php.net/manual/ja/book.reflection.php>`_ に対して関数の引数の情報を提供するのに使われます。指定しなくても動きますが、作成した Extension を外部に公開する場合は指定してください。詳細は `リフレクション情報を設定する <http://codezine.jp/article/detail/7385?p=3>`_ を参照してください。

6.3.関数本体の追加
==================

　zend_function_entry の宣言より前に、関数本体を以下のように追加します。::

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

　関数の前のプロトタイプ宣言っぽいコメントや、関数全体をコメントレベルで `{{{` ～ `}}}` で囲むスタイルは、PHP の標準に準じた記載です。詳細は :ref:`folding-hooks` を参照してください。なお、成果物をインターネット上で公開する場合は当然コメントは英語で入れるべきですが、本ドキュメントではわかりやすくするために日本語で記載します。

6.4.ビルドと環境設定
====================

　まだ関数の中身がありませんが、いったんこれでビルドしてみましょう。::

  ~/php/ext/my_ext$ make
  （中略）
  Build complete.
  Don't forget to run 'make test'.

　次に実行ですが、その前に、毎回 extension 指定をしなくてもいいように、php.ini に Extension の読み込み設定を入れておきしょう。ソースからビルドした場合の設定ファイルのパスは /usr/local/lib/php.ini で、デフォルトではこのファイルはありませんから新規で作成します。::

  ~/php/ext/my_ext$ sudo vi /usr/local/lib/php.ini
  （ファイルを新規で作成）
  ~/php/ext/my_ext$ cat /usr/local/lib/php.ini
  extension=/home/vagrant/php/ext/my_ext/modules/my_ext.so

　設定を追加したら、モジュールとして読み込まれていることを確認します。::

  ~/php/ext/my_ext$ php -m | grep my_ext
  my_ext

6.5.動作確認
============

　それでは実行してみましょう。::

  ~/php/ext/my_ext$ php -r "my_echo_int(100);"

　何も表示されませんが、エラーにもなりませんね。切り分けのために、わざと関数名を間違えてみましょう。::

  ~/php/ext/my_ext$ php -r "my_echo_int_not_exist(100);"
  Fatal error: Uncaught Error: Call to undefined function my_echo_int_not_exist() in Command line code:1
  Stack trace:
  #0 {main}
    thrown in Command line code on line 1

　ちゃんと見慣れたエラーメッセージが表示されましたね。少なくとも、PHP は関数としては認識してくれているようです。

6.6.関数の中身の実装
====================

　それでは関数本体を実装しましょう。::

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

　これで再度ビルドして実行します。::

  ~/php/ext/my_ext$ make
  ~/php/ext/my_ext$ php -r "my_echo_int(100);"
  my_echo_int(100)

　これで C のライブラリに制御が渡って、無事に実行されました。

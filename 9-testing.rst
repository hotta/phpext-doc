==================
9.テストの組み込み
==================

9.1.戻り値の返却
================

　次は戻り値を返す関数のサンプルとして、my_add_return_int() をやってみましょう。my_lib.h における C のプロトタイプ宣言は以下の通りです。

.. code-block:: bash
  :emphasize-lines: 1

  ~/my_lib$ grep my_add_return_int my_lib.h
  extern  int   my_add_return_int(int a, int b);

　関数の実体は以下の通りです。

.. code-block:: bash
  :emphasize-lines: 1

  ~/my_lib$ cat components/my_add_return_int.c
  #include <stdio.h>
  
  int my_add_return_int(int a, int b)
  {
    return a + b;
  }

　これに対応する、PHP レベルの仕様は以下のようになります。

.. php:function:: int my_add_return_int( int $a, int $b )
  $a と $b の和を返します。

　実装は以下のようになりました。

.. code-block:: bash
  :emphasize-lines: 1,10-26,34

  ~/php/ext/my_ext$ git diff my_ext.c
  diff --git a/my_ext.c b/my_ext.c
  index e0cec50..2581718 100644
  --- a/my_ext.c
  +++ b/my_ext.c
  @@ -90,6 +90,23 @@ PHP_FUNCTION(my_echo_int)
   }
   /* }}} */
  
  +/* {{{ proto int my_add_return_int( int a, int b )
  +   a と b の和を返します。
  +*/
  +PHP_FUNCTION(my_add_return_int)
  +{
  +  zend_long a, b;
  +
  +  if (ZEND_NUM_ARGS() < 2 || 2 < ZEND_NUM_ARGS()) {
  +    WRONG_PARAM_COUNT;
  +  }
  +  if (zend_parse_parameters(ZEND_NUM_ARGS(), "ll", &a, &b) == FAILURE) {
  +    return;
  +  }
  +  RETURN_LONG(my_add_return_int(a, b));
  +}
  +/* }}} */
  +
   /* {{{ php_my_ext_init_globals
    */
   /* Uncomment this function if you have INI entries
  @@ -165,6 +182,7 @@ PHP_MINFO_FUNCTION(my_ext)
   const zend_function_entry my_ext_functions[] = {
          PHP_FE(confirm_my_ext_compiled, NULL)           /* For testing, remove l
          PHP_FE(my_echo_int,             NULL)
  +       PHP_FE(my_add_return_int,       NULL)
          PHP_FE_END      /* Must be the last line in my_ext_functions[] */
   };
   /* }}} */

　ビルドして実行してみました。うまくいっているようです。

.. code-block:: bash
  :emphasize-lines: 1

  ~/php/ext/my_ext$ php -r "print my_add_return_int(3,4).PHP_EOL;"
  7


9.2.PHPUnitの導入
=================

　いくつかパラメーターのパターンを追加してみましょう。毎回コマンドラインからテストパターンを入力するのは効率が悪いので、 `PHPUnit <https://phpunit.de/manual/current/ja/installation.html>`_ によるユニットテストを行います。PHPUnit は、以下の手順で導入します。

.. code-block:: bash
  :emphasize-lines: 1-4

  ~$ wget https://phar.phpunit.de/phpunit-6.2.phar
  ~$ chmod +x phpunit-6.2.phar
  ~$ sudo mv phpunit-6.2.phar /usr/local/bin/phpunit
  ~$ phpunit --version
  PHPUnit 6.2.1 by Sebastian Bergmann and contributors.

9.3.最初のテストケース
======================

　コマンドラインから行った動作確認を、PHPUnit 経由で実行します。tests 配下に以下の PHP スクリプトを置きます。

.. code-block:: bash
  :emphasize-lines: 1-3

  ~$ cd ~/php/ext/my_ext
  ~/php/ext/my_ext$ vi tests/MyAddReturnIntTest.php
  ~/php/ext/my_ext$ cat tests/MyAddReturnIntTest.php
  <?php
  
  use PHPUnit\Framework\TestCase;
  
  class MyAddReturnIntTest extends TestCase
  {
    public function test3plus4returns7()
    {
      $ret = my_add_return_int(3, 4);
      $this->assertEquals(7, $ret);
    }
  }

　それでは実行してみます。

.. code-block:: bash
  :emphasize-lines: 1

  ~/php/ext/my_ext$ phpunit tests/MyAddReturnIntTest.php
  PHPUnit 6.2.1 by Sebastian Bergmann and contributors.
  
  .                                                                   1 / 1 (100%)
  
  Time: 120 ms, Memory: 10.00MB
  
  OK (1 test, 1 assertion)

　``7`` と返ってきた加算結果 ``$ret`` を比較したら等しかったので、テストは成功です。 ``assertEquals()`` は PHPUnit のアサーションのひとつです。詳細は `付録A アサーション - assertEquals() <https://phpunit.de/manual/current/ja/appendixes.assertions.html#appendixes.assertions.assertEquals>`_ を参照してください。


9.4.テストケースの追加
======================

　MyAddReturnIntTest クラスの中に、以下のようなパラメーターが１個しかないケースを追加しました。

.. code-block:: bash
  :emphasize-lines: 1,10-14

  ~/php/ext/my_ext$ git diff
  diff --git a/tests/MyAddReturnIntTest.php b/tests/MyAddReturnIntTest.php
  index e27db50..9210507 100644
  --- a/tests/MyAddReturnIntTest.php
  +++ b/tests/MyAddReturnIntTest.php
  @@ -9,4 +9,9 @@ class MyAddReturnIntTest extends TestCase
       $ret = my_add_return_int(3, 4);
       $this->assertEquals(7, $ret);
     }
  +
  +  public function testLackOfParameter()
  +  {
  +    $ret = my_add_return_int(3);
  +  }
   }

　これで実行してみますと、

.. code-block:: bash
  :emphasize-lines: 1


  ~/php/ext/my_ext$ phpunit tests/MyAddReturnIntTest.php
  PHPUnit 6.2.1 by Sebastian Bergmann and contributors.
  
  .E                                                                  2 / 2 (100%)
  
  Time: 121 ms, Memory: 10.00MB
  
  There was 1 error:
  
  1) MyAddReturnIntTest::testLackOfParameter
  Wrong parameter count for my_add_return_int()
  
  /usr/local/src/php-7.1.5/ext/my_ext/tests/MyAddReturnIntTest.php:15
  
  ERRORS!
  Tests: 2, Assertions: 1, Errors: 1.

　エラーになってしまいました。

9.5.エラーと例外の扱い
======================

　PHP におけるエラーと例外は、 `Standard PHP Library (SPL) <http://php.net/manual/ja/book.spl.php>`_ で規定されています。ソースツリーの中の実体は、php/ext/spl/spl.php にあります。
 
　PHPUnit でもエラーと例外は `標準でサポート <https://phpunit.de/manual/6.2/ja/writing-tests-for-phpunit.html#writing-tests-for-phpunit.exceptions>`_ されているのですが、手元で試した限りでは、引数誤りエラーを補足することはできませんでした。試した結果、以下の方法でうまくいくようになりました。

　まず、テスト前処理を行なうクラス setUpBeforeClass() を定義し、その中でエラーを捕捉して RuntimeException 例外を発生させるようにします。 ``setUpBeforeClass()`` などは `フィクスチャ <https://phpunit.de/manual/current/ja/fixtures.html>`_ と呼ばれるもので、マニュアルに詳細に記載があります。

.. code-block:: bash
  :emphasize-lines: 1


  ~/php/ext/my_ext$ cat tests/MyTestCase.php
  <?php
  
  use PHPUnit\Framework\TestCase;
  
  class MyTestCase extends TestCase
  {
    public static function setUpBeforeClass() {
      set_error_handler(function($errno, $errstr, $errfile, $errline) {
        throw new RuntimeException($errstr . " on line "
          . $errline . " in file " . $errfile);
      });
      ini_set('log_errors', 1);
      ini_set('display_errors', 0);
    }
  }

　そして、実際のテストケースは、新たに作成した MyTestCase を継承するようにします。また、関数コールに対して RuntimeException 例外の発生を期待することを PHPunit に教えます。コメントの中で '@' 記号で指示するメタデータは `アノテーション <https://phpunit.de/manual/current/ja/appendixes.annotations.html>`_ と呼ばれます。

.. code-block:: bash
  :emphasize-lines: 1,9,10,12,13,21-23


  $ git diff
  diff --git a/tests/MyAddReturnIntTest.php b/tests/MyAddReturnIntTest.php
  index 243bf88..dd28dce 100644
  --- a/tests/MyAddReturnIntTest.php
  +++ b/tests/MyAddReturnIntTest.php
  @@ -1,8 +1,8 @@
   <?php
  
  -use PHPUnit\Framework\TestCase;
  +require_once 'MyTestCase.php';
  
  -class MyAddReturnIntTest extends TestCase
  +class MyAddReturnIntTest extends MyTestCase
   {
     public function test3plus4returns7()
     {
  @@ -10,6 +10,9 @@ class MyAddReturnIntTest extends TestCase
       $this->assertEquals(7, $ret);
     }
  
  +  /**
  +    * @expectedException RuntimeException
  +    */
     public function testLackOfParameter()
     {
       my_add_return_int(3);

　これで一応通るようになりました。

.. code-block:: bash
  :emphasize-lines: 1

  ~/php/ext/my_ext$ phpunit tests/MyAddReturnIntTest.php
  PHPUnit 6.2.1 by Sebastian Bergmann and contributors.
  
  ..                                                                  2 / 2 (100%)
  
  Time: 138 ms, Memory: 10.00MB
  
  OK (2 tests, 2 assertions)

　ただ、なんだかやっつけのような気もするので、本来のやり方（？）がわかったら追記します。

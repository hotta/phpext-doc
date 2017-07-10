.. _phpext-xdebug:

=============
付録D. Xdebug
=============

1. Xdebug とは
==============

　Xdebug は PHP 用のデバッグツールです。主な機能は以下の通りです。

* スタックの追跡。エラーが発生するまでの経過を詳細に表示する。関数に渡されたパラメーターも表示され、エラーの原因を探しやすくする
* var_dumpを整形して出力する。 `VarDumper <https://www.sitepoint.com/var_dump-introducing-symfony-vardumper/>`_ 同様、色分けした情報と構造化ビューを生成。スーパーグローバルのダンパーが可能
* コードのボトルネックを特定するプロファイラー。外部のツールでパフォーマンスのグラフをビジュアライズでき、 `Blackfire <https://www.sitepoint.com/an-in-depth-walkthrough-of-supercharging-apps-with-blackfire/>`_ のようなグラフが書ける
* 実行中のコードや、IDE、ブラウザーなどのエンドクライアントにリモートで Xdebug を接続するリモートデバッガー。コードにブレークポイントを設定して1行ずつアプリケーションを実行できる
* リクエスト中に実行されたコードの量を示すコードカバレッジ。ユニットテストで使う。テストでカバーしたコードの割合が分かる

（出典 `PHP開発者がいまさら聞けない、Xdebugの基礎の基礎 <https://www.webprofessional.jp/getting-know-love-xdebug/>`_ ）

2. インストール
===============

　:ref:`phpext-recommendation` で記載した ansible 環境を入れている場合、PHP をパッケージで入れるならは単に

.. code-block:: bash


  $ ansible-playbook /etc/ansible/jobs/xdebug.yml

　だけで入ります。ただ、本ドキュメントでは PHP 自体に手を入れる関係上、PHP はソースからビルドすることを推奨していますので、Xdebug についてもソースを git リポジトリから導入することにします。以下の手順でインストールします。

.. code-block:: bash

  $ git clone git://github.com/xdebug/xdebug.git
  $ cd xdebug
  $ phpize
  $ ./configure --enable-xdebug
  $ make

　これで xdebug のバイナリが PHP Extension の形でビルドできました。これを PHP バイナリで認識させるために、Extension 読み込みの設定を追加します。

.. code-block:: bash
  :emphasize-lines: 1,2,5

  $ sudo vi /usr/local/lib/php.ini
  $ cat /usr/local/lib/php.ini
  extension=/home/vagrant/php/ext/my_ext/modules/my_ext.so
  zend_extension=/home/vagrant/xdebug/modules/xdebug.so     ← この行を追加
  $ php -m | grep xdebug
  xdebug

3. 提供される関数
=================

　Xdebug Extension によって提供される（または機能が置き換わる）関数を以下に示します。詳細は Xdebug ドキュメント の `Related Functions <https://xdebug.org/docs/all_functions>`_ を参照してください。

3.1. var_dump()
---------------

.. php:function:: void var_dump ( mixed $var [, ... ]] )
  変数の詳細を表示する。

  この関数は Xdebug によって xdebug_var_dump にオーバーロードされます。詳細は :ref:`xdebug_var_dump` を参照してください。

3.2. xdebug_break()
-------------------

.. php:function:: bool xdebug_break()
  ``debug client`` に対してブレークポイントを発生させる。

  この関数は、あたかもこの行に対して通常のファイル／行レベルのブレークポイントを指定するように、指定した行にデバッガーのブレークポイントを設定します。

.. _xdebug_call_class:

3.3. xdebug_call_class()
------------------------

.. php:function:: string xdebug_call_class( [int $depth = 1] )

  コールされたクラスを返します。スタックフレームが見つからなければ NULL を、
  スタックフレームにクラスの情報がない場合は FALSE を返します。

  この関数は、現在のメソッドを定義したクラスの名前を返します。この呼び出しに対してクラスが関連付けられてない場合は FALSE を返します。


.. _xdebug_call_file:

3.4. xdebug_call_file()
------------------------

.. php:function:: string xdebug_call_file( [int $depth = 1] )

  この関数は、現在の関数／メソッドが実行された箇所からファイル名を返します。
  過去のスタックフレームから情報を取り出すために、オプションの `$depth` 引数を使用します。
  

.. _xdebug_call_function:

3.4. xdebug_call_function()
---------------------------

.. php:function:: string xdebug_call_function( [int $depth = 1] )

  コールしている関数／メソッドを返します。スタックフレームが見つからなければ
  NULL を、スタックフレームに関数／メソッドの情報がない場合は FALSE を返します。

  この関数は、現在の関数／メソッドの名前を返します。
  過去のスタックフレームから情報を取り出すために、オプションの `$depth` 引数を使用します。
  
3.5. xdebug_call_line()
-----------------------

.. php:function:: string xdebug_call_line( [int $depth = 1] )

3.6. xdebug_code_coverage_started()
-----------------------------------

.. php:function:: boolean xdebug_code_coverage_started()

3.7. xdebug_debug_zval()
------------------------

.. php:function:: void xdebug_debug_zval( [string varname [, ...]] )
  変数に関する情報を表示します。

この関数は、１つ以上の評価式について、その型や値、および refcount を含む情報を構造化して表示します。配列は値で再帰的に展開されます。この関数は PHP の `debug_zval_dump <http://php.net/debug-zval-dump>`_ とは異なった実装になっており、変数そのものが実際にその関数に渡されるケースでの問題を回避しています。Xdebug 版では、変数の検索は内部のシンボルテーブルを通して行い、すべてのプロパティへのアクセスはその関数に実際に変数を渡すことなく直接扱えるようになっているなど、よりよい実装になっています。その結果、この関数が返す zval 関連の情報は、PHP 版の関数よりさらに正確になっています。

Xdebug 2.3 以降は、単純変数名（以下の "a[2]" など）のようなものだけをサポートするようにしています。

利用例

.. code-block:: php

  <?php
      $a = array(1, 2, 3);
      $b =& $a;
      $c =& $a[2];
  
      xdebug_debug_zval('a');
      xdebug_debug_zval("a[2]");
  ?>

返される結果::

  a: (refcount=2, is_ref=1)=array (
    0 => (refcount=1, is_ref=0)=1, 
    1 => (refcount=1, is_ref=0)=2, 
    2 => (refcount=2, is_ref=1)=3)
  a[2]: (refcount=2, is_ref=1)=3


3.8. xdebug_debug_zval_stdout()
-------------------------------

.. php:function:: void xdebug_debug_zval_stdout( [string varname [, ...]] )

3.9.xdebug_disable()
--------------------

.. php:function:: void xdebug_disable()

3.10. xdebug_dump_superglobals()
--------------------------------

.. php:function:: void xdebug_dump_superglobals()

3.11. xdebug_enable()
---------------------

.. php:function:: void xdebug_enable()

3.12. xdebug_get_code_coverage()
--------------------------------

.. php:function:: array xdebug_get_code_coverage()

3.12. xdebug_get_collected_errors()
-----------------------------------

.. php:function:: string xdebug_get_collected_errors( [int clean] )

3.13. xdebug_get_declared_vars()
--------------------------------

.. php:function:: array xdebug_get_declared_vars()

3.14. xdebug_get_function_stack()
---------------------------------

.. php:function:: array xdebug_get_function_stack()

3.15. xdebug_get_headers()
--------------------------

.. php:function:: array xdebug_get_headers()

3.16. xdebug_get_monitored_functions()
--------------------------------------

.. php:function:: array xdebug_get_monitored_functions()

3.17. xdebug_get_profiler_filename()
------------------------------------

.. php:function:: string xdebug_get_profiler_filename()

3.18. xdebug_get_stack_depth()
------------------------------

.. php:function:: integer xdebug_get_stack_depth()

3.19. xdebug_get_tracefile_name()
---------------------------------

.. php:function:: string xdebug_get_tracefile_name()

3.20. xdebug_is_enabled()
-------------------------

.. php:function:: bool xdebug_is_enabled()

3.21. xdebug_memory_usage()
-------------------------------

.. php:function:: int xdebug_memory_usage()

3.22. int xdebug_peak_memory_usage()
------------------------------------

.. php:function:: xdebug_peak_memory_usage()

3.23. xdebug_print_function_stack()
-----------------------------------

.. php:function:: none xdebug_print_function_stack( [ string message [, int options ] ] )

3.24. xdebug_start_code_coverage()
----------------------------------

.. php:function:: void xdebug_start_code_coverage( [int options] )

3.25. xdebug_start_error_collection()
-------------------------------------

.. php:function:: void xdebug_start_error_collection()

3.26. xdebug_start_function_monitor()
-------------------------------------

.. php:function:: void xdebug_start_function_monitor( array $list_of_functions_to_monitor )

3.27. xdebug_start_trace()
--------------------------

.. php:function:: string xdebug_start_trace( [ string trace_file [, integer options] ] )

3.28. xdebug_stop_code_coverage()
---------------------------------

.. php:function:: void xdebug_stop_code_coverage( [int cleanup=true] )

3.29. xdebug_stop_error_collection()
------------------------------------

.. php:function:: void xdebug_stop_error_collection()

3.30. xdebug_stop_function_monitor()
------------------------------------

.. php:function:: void xdebug_stop_function_monitor()

3.31. xdebug_stop_trace()
-------------------------

.. php:function:: string xdebug_stop_trace()

3.32. xdebug_time_index()
-------------------------

.. php:function:: float xdebug_time_index()

.. _xdebug_var_dump:

3.33.xdebug_var_dump
--------------------

.. php:function:: void xdebug_var_dump ( mixed $var [, ... ]] )
  指定された変数に関する詳細情報を表示します。

この関数は、１つ以上の評価式について、その型や値を含む情報を構造化して表示します。配列は値で再帰的に展開されます。


利用例

.. code-block:: php

  <?php
  ini_set('xdebug.var_display_max_children', 3 );
  $c = new stdClass;
  $c->foo = 'bar';
  $c->file = fopen( '/etc/passwd', 'r' );
  var_dump(
      array(
          array(TRUE, 2, 3.14, 'foo'),
          'object' => $c
      )
  );
  ?>  

出力結果::

  array
    0 => 
      array
        0 => boolean true
        1 => int 2
        2 => float 3.14
        more elements...
    'object' => 
      object(stdClass)[1]
        public 'foo' => string 'bar' (length=3)
        public 'file' => resource(3, stream)

4. デバッガを通した利用
=======================

　Xdebug が最も多く使われるのがこのケースです。以前、Windows + NetBeans という環境から、CentOS 上の Apache 経由で CakePHP 2.x をトレースするケースにおけるチュートリアルを以前書きましたので、そのリンクを置いておきます。

`Xdebug+NetBeansによるCakePHPのデバッグ <https://net-newbie.com/cakephp/xdebug-netbeans/>`_

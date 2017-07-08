============
付録C.phpdbg
============

　phpdbg は PHP に特化したソースコードレベルのデバッガです。gdb と同様に、会話形式でコマンドにより動作を制御します。コマンドは、たとえば 'list' なら単に 'l' のように、特定可能な範囲で省略可能です。

　マニュアルは用意されていないようですが、強力な help コマンドが内蔵されています。またリモートコンソールモードという、他のホストからリモートで制御するモードもあります。

C-1.起動
========

コマンド書式
  phpdbg [オプション] [PHPスクリプトファイル名] [ -- arg1 arg2 ... ]

  phpdbg は、カレントディレクトリに .phpdbginit というファイルがあれば、それをスタートアップスクリプトとして読み込みます。gdb 等と異なり、ホームディレクトリ直下に置いても参照されないようです。

  .phpdbginit の書き方は、help phpdbginit で参照できます。

オプション
  help options で利用可能なオプションの一覧を表示できます。

C-2.コマンド一覧
================

　help コマンドで表示されるコマンドの一覧を以下に示します。詳細は `help コマンド名` で確認してください。

C-2-1.表示関連
--------------

.. list-table::
  :widths: 10 50
  :header-rows: 1

  * - コマンド名（略号）
    - 概要
  * - list(l)
    - PHP のソースを表示する
  * - info
    - デバッグセッションの情報を表示する
  * - print(p)
    - オペコードを表示する
  * - frame(f)
    - | 現在またはその祖先(呼び出し元）のスタックフレームを選択して、
      | そのスタックのサマリーを表示する。
      | その後実行を継続した場合、選択したスタックフレームは破棄される。
  * - generator(g)
    - アクティブなジェネレーターを選択するか、ジェネレーターフレームを選択する
  * - back(t)
    - 現在のバックトレースを表示する  
  * - help(h)
    - トピックのヘルプを表示する

list(l)
^^^^^^^

.. list-table::
  :header-rows: 1

  * - 利用例
    - 説明
  * - prompt> **list 10** | prompt> **l 10**
    - 現在行から 10 行表示する
  * - prompt> **list my_function** | prompt> **list f my_function**
    - 関数 my_function のソースを表示する
  * - prompt> **list func .mine** | prompt> **l f .mine**
    - | スコープ内のアクティブなクラスにある
      | メソッド mine のソースを表示する
  * - prompt> **list m my::method** | prompt> l **my::method**
    - my::method  のソースを表示する
  * - prompt> **list c myClass** | prompt> l c **myClass**
    - myClass  のソースを表示をする

info
^^^^

.. list-table::
  :header-rows: 1

  * - 利用例
    - 説明
  * - info
    - 実行コンテキストの情報
  * - prompt> **info break** | prompt> **info b**
    - 現在のブレークポイント一覧
  * - prompt> **info funcs** | prompt> **info f**
    - ロードされている関数の一覧

print(p)
^^^^^^^^

.. list-table::
  :header-rows: 1

  * - 利用例
    - 説明
  * - | prompt> **print class \my\class**
      | prompt> **p c \my\class**
    - \my\class にあるメソッドのオペコードを表示する
  * - | prompt> **print method \my\class::method**
      | prompt> **p m \my\class::method**
    - \my\class::method のオペコードを表示する
  * - | prompt> **print func .getSomething**
      | prompt> **p f .getSomething**
    - | 現在のアクティブスコープにある ::getSomething の
      | オペコードを表示する
  * - | prompt> **print func my_function**
      | prompt> **p f my_function**
    - グローバルの my_function 関数のオペコードを表示する
  * - | prompt> **print opline**
      | prompt> **p o**
    - | 現在の opline （オペコードレベルの現在行）の
      | オペコードを表示する
  * - | prompt> **print exec**
      | prompt> **p e**
    - 実行コンテキストのオペコードを表示する
  * - | prompt> **print stack**
      | prompt> **p s**
    - 現在のスタック上のオペコードを表示する

frame(f)
^^^^^^^^

.. list-table::
  :header-rows: 1

  * - 利用例
    - 説明
  * - | prompt> **frame 2**
      | prompt> **ev $count**
    - フレーム 2 に移動して、そのフレームにおける変数 $count の中身を表示する。


C-2-2.実行の開始と停止
----------------------

.. list-table::
  :widths: 10 50
  :header-rows: 1

  * - コマンド名（略号）
    - 概要
  * - exec(e)
    - 実行コンテキストをセットする
  * - stdin
    - 標準入力から実行するスクリプトを読み込む
  * - run(r)
    - 実行する
  * - step(s)
    - 次の行まで実行する
  * - continue(c)
    - 続けて実行する
  * - until(u)
    - 指定の場所まで実行する
  * - next(n)
    - 指定の場所まで実行し、その先頭行で実行を停止する
  * - finish(f)
    - 現在の実行フレームの最後まで実行を続ける
  * - leave(L)
    - 現在の実行フレームの最後まで実行を続け、その後実行を停止する
  * - break(b)
    - | 指定された箇所にブレークポイントを設定する
      | break 位置 --------- 指定の位置でブレークポイントを設定
      | break at(A) 条件 --- 指定の位置と条件でブレークポイントを設定
      | break del(d) 番号 -- ID指定でブレークポイントを削除
  * - watch(w)
    - 変数 $variable にウォッチポイントを設定する
  * - clear(C)
    - １つまたはすべてのブレークポイントをクリアする
  * - clean(X)
    - 実行環境を消去する

exec(e)
^^^^^^^

.. list-table::
  :header-rows: 1

  * - 利用例
    - 説明
  * - | prompt> **exec /tmp/script.php**
      | prompt> **e /tmp/script.php**
    - 実行対象コンテキストを /tmp/script.php にする


stdin
^^^^^

.. list-table::
  :header-rows: 1

  * - 利用例
    - 説明
  * - | prompt>  **stdin foo**
      | **<?php**
      | **echo "Hello, world!n";**
      | **foo**
    - 引数をデリミタとし、標準入力を読み込んで実行コンテキストとして評価する


run(r)
^^^^^^

.. list-table::
  :header-rows: 1

  * - 利用例
    - 説明
  * - prompt> **run** / prompt> **r**
    - 実行コンテキストがセットされている場合、実行する
  * - prompt> **r test < foo.txt**
    - $argv[1] == "test" 、foo.txt を STDIN として実行する

break(b)
^^^^^^^^

.. list-table::
  :header-rows: 1

  * - 利用例
    - 説明
  * - | prompt> **break test.php:100**
      | prompt> **b test.php:100**
    - test.php の 100 行目で実行を停止する
  * - | prompt>  **break 200**
      | prompt>  **b 200**
    - | 現在の PHP スクリプトファイル の 200 行目で停止する
  * - | prompt> **break \mynamespace\my_function**
      | prompt> **b \mynamespace\my_function**
    - \mynamespace\my_function のエントリで停止する
  * - | prompt> **break classX::method**
      | prompt> **b classX::method**
    - classX::method のエントリで停止する
  * - | prompt> **break 0x7ff68f570e08**
      | prompt> **b 0x7ff68f570e08**
    - opline のアドレス 0x7ff68f570e08 で停止する
  * - | prompt> **break my_function#14**
      | prompt> **b my_function#14**
    - 関数 my_function の opline #14 で停止する
  * - | prompt> **break \my\class::method#2**
      | prompt> **b \my\class::method#2**
    - メソッド \my\class::method の opline #2 で停止する
  * - | prompt> **break test.php:#3**
      | prompt> **b test.php:#3**
    - test.php の #3 で停止する
  * - | prompt> **break if $cnt > 10**
      | prompt> **b if $cnt > 10**
    - 条件 ($cnt > 10) の評価結果が真になったら停止する
  * - | prompt> **break at phpdbg::isGreat if $opt == 'S'**
      | prompt> **break @ phpdbg::isGreat if $opt == 'S'**
    - | 条件 ($opt == 'S') が真になったら phpdbg::isGreat 
      | のいずれかのオペコードで停止する
  * - | prompt> **break at test.php:20 if !isset($x)**
    - | 条件の評価結果が真になったら 
      | test.php の 20 行目で停止する
  * - | prompt> **break ZEND_ADD**
      | prompt> **b ZEND_ADD**
    - オペコード ZEND_ADD に出会ったら停止する
  * - | prompt> **break del 2**
      | prompt> **b ~ 2**
    - ブレークポイントの２番を削除する

watch(w)
^^^^^^^^

* 変数定義が有効な間、変数にウォッチポイントを設定する
* 引数が与えられない場合、現在アクティブなウォッチポイントの一覧を表示する

.. list-table:: 変数の書式
  :header-rows: 1

  * - 書式
    - 説明
  * - $var
    - $var という変数
  * - $var[]
    - $var の配列要素すべて
  * - $var->
    - $var のすべてのプロパティ
  * - $var->a
    - プロパティ $var->a
  * - $var[b]
    - 配列 $var の中の b というキーを持つ配列要素

.. list-table:: watch のサブコマンド
  :header-rows: 1

  * - タイプ
    - エイリアス
    - 目的
  * - array
    - a
    - | 配列またはオブジェクトにウォッチポイントを設定し、
      | エントリが追加／削除されるのを監視する
  * - recursive
    - r
    - | 変数を再帰的にウォッチし、配列またはオブジェクトに
      | エントリが追加された場合自動的にウォッチポイントを追加する
  * - delete
    - d
    - ウォッチポイントを削除する

　再帰的ウォッチポイントが削除された場合、その子供のウォッチポイントもすべて削除されます。

.. list-table:: watch の使用例
  :widths: 20 50
  :header-rows: 1

  * - 使用例
    - 説明
  * - prompt> **watch**
    - 現在アクティブなウォッチポイントの一覧を表示する
  * - | prompt> **watch $array**
      | prompt> **w $array**
    - $array に対してウォッチポイントを設定する
  * - | prompt> **watch recursive $obj->**
      | prompt> **w r $obj->**
    - $obj-> に対して再帰的にウォッチポイントを設定する
  * - | prompt> **watch delete $obj->a**
      | prompt> **w d $obj->a**
    - $obj->a のウォッチポイントを削除する

技術的な留意点
  この機能をデバッガ上で使用した場合、監視対象のアドレスを含むメモリページがヒットするたびにセグメンテーション違反(SEGV)が起こります。その後も実行を継続できる場合、phpdbg は書き込み保護を無効にするため、プログラムは継続実行できます。

  もし phpdbg が SEGV を処理できなかった場合、再度 SEGV がトリガーされ、phpdbg は異常終了します。

clear(C)
^^^^^^^^

　ブレークポイントを削除します。 ブレークポイントを削除すると、そのコードを割り込みなしにもう一度実行可能となります。

* break delete N を使うと、特定のブレークポイントを削除できます。
* すべてのブレークポイントがクリアされると、PHP スクリプトは正常終了するまで実行されます。

clean(X)
^^^^^^^^

　PHP でクラス、定数、関数を宣言できるのは１度だけです。デバッグ中に PHP をリコンパイルする際、エラーが起こることがあります。clean コマンドはコンパイル後のクラス、停止位、関数を保持している Zend 実行時テーブルをクリアし、関連するストレージをストレージプールに戻します。これによりリコンパイルを行えるようになります。

　これらのリソースプールを選択的にクリアすることはできません。完全にクリーンな状態に戻すことだけが可能です。


C-2-3.その他のコマンド
----------------------

.. list-table::
  :widths: 10 50
  :header-rows: 1

  * - コマンド名
    - 概要
  * - set
    - phpdbg の設定値をセットする
  * - source
    - phpdbginit スクリプトを実行する
  * - register
    - phpdbginit 関数をコマンドエイリアスとして登録する
  * - sh
    - シェルのコマンドを実行する
  * - ev
    - PHP のコードを評価(eval)する
  * - quit
    - phpdbg を抜ける


C-3.利用例
==========

　以下のような PHP スクリプトを用意しましたので、このファイルを使って使い方を見てみましょう。主に、zval から参照される gc.refcount の動きを追跡するのが目的です。

::

  $ cat simple-copy.php
  <?php
  $a = "new string";
  $b = $a;

　スクリプトファイル名を引数として起動します。

::

  $ phpdbg simple-copy.php
  [Welcome to phpdbg, the interactive PHP debugger, v0.5.0]
  To get help using phpdbg type "help" and press enter
  [Please report bugs to <http://bugs.php.net/report.php>]
  [Successful compilation of /home/vagrant/temp/simple-copy.php]
  prompt>

　起動時のメッセージでわかるように、'prompt>' が表示された時点ですでに PHP スクリプトのコンパイルは完了しています。phpdbg では PHP スクリプトの行単位、もしくはコンパイル後のオペコード単位でステップ実行できます。

::

  prompt> list 3
   00001: <?php
   00002: $a = "new string";
   00003: $b = $a;

　gdb と異なり、list には引数が必要です。help list で詳細を確認してください。



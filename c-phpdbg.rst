.. _phpext-phpdbg:

=============
付録C. phpdbg
=============

　phpdbg はコンソール上で使用する、PHP に特化したソースコードレベルのデバッガです。gdb と同様に、会話形式でコマンドにより動作を制御します。コマンドは、たとえば 'list' なら単に 'l' のように、特定可能な範囲で省略可能です。

　マニュアルは用意されていないようですが、強力な help コマンドが内蔵されています。またリモートコンソールモードという、他のホストからリモートで制御するモードもあります。

C-1.起動
========

コマンド書式
  **phpdbg [オプション] [PHPスクリプトファイル名] [ -- arg1 arg2 ... ]**

  phpdbg は、カレントディレクトリに .phpdbginit というファイルがあれば、それをスタートアップスクリプトとして読み込みます。gdb 等と異なり、ホームディレクトリ直下にあるものは参照されないようです。

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

  * - コマンド名（省略形）
    - 概要
  * - :ref:`phpdbg-list`
    - PHP のソースを表示する
  * - :ref:`phpdbg-info`
    - デバッグセッションの情報を表示する
  * - :ref:`phpdbg-print`
    - オペコードを表示する
  * - :ref:`phpdbg-frame`
    - | 現在またはその祖先(呼び出し元）のスタックフレームを選択して、
      | そのスタックのサマリーを表示する。
      | その後実行を継続した場合、選択したスタックフレームは破棄される。
  * - generator(g)
    - | アクティブなジェネレーターを選択するか、
      | ジェネレーターフレームを選択する
  * - back(t)
    - 現在のバックトレースを表示する  
  * - help(h)
    - トピックのヘルプを表示する

.. _phpdbg-list:

list(l)
^^^^^^^

.. list-table::
  :widths: 30 70
  :header-rows: 1

  * - 利用例
    - 説明
  * - | prompt> **list 10** 
      | prompt> **l 10**
    - 現在行から 10 行表示する
  * - | prompt> **list my_function** 
      | prompt> **list f my_function**
    - 関数 my_function のソースを表示する
  * - | prompt> **list func .mine** 
      | prompt> **l f .mine**
    - | スコープ内のアクティブなクラスにあるメソッド mine の
      | ソースを表示する
  * - | prompt> **list m my::method** 
      | prompt> l **my::method**
    - my::method  のソースを表示する
  * - | prompt> **list c myClass** 
      | prompt> l c **myClass**
    - myClass  のソースを表示をする

.. _phpdbg-info:

info
^^^^

.. list-table::
  :widths: 30 70
  :header-rows: 1

  * - 利用例
    - 説明
  * - info
    - 実行コンテキストの情報
  * - | prompt> **info break** 
      | prompt> **info b**
    - 現在のブレークポイント一覧
  * - | prompt> **info funcs** 
      | prompt> **info f**
    - ロードされている関数の一覧

.. _phpdbg-print:

print(p)
^^^^^^^^

.. list-table::
  :widths: 30 70
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

.. _phpdbg-frame:

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

  * - コマンド名（省略形）
    - 概要
  * - :ref:`phpdbg-exec`
    - 実行コンテキストをセットする
  * - :ref:`phpdbg-stdin`
    - 標準入力から実行するスクリプトを読み込む
  * - :ref:`phpdbg-run`
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
  * - :ref:`phpdbg-break`
    - | 指定された箇所にブレークポイントを設定する
      | break 位置 --------- 指定の位置でブレークポイントを設定
      | break at(A) 条件 --- 指定の位置と条件でブレークポイントを設定
      | break del(d) 番号 -- ID指定でブレークポイントを削除
  * - :ref:`phpdbg-watch`
    - 変数 $variable にウォッチポイントを設定する
  * - :ref:`phpdbg-clear`
    - １つまたはすべてのブレークポイントをクリアする
  * - :ref:`phpdbg-clean`
    - 実行環境を消去する

.. _phpdbg-exec:

exec(e)
^^^^^^^

.. list-table::
  :header-rows: 1

  * - 利用例
    - 説明
  * - | prompt> **exec /tmp/script.php**
      | prompt> **e /tmp/script.php**
    - 実行対象コンテキストを /tmp/script.php にする


.. _phpdbg-stdin:

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

.. _phpdbg-run:


run(r)
^^^^^^

.. list-table::
  :widths: 20 40
  :header-rows: 1

  * - 利用例
    - 説明
  * - prompt> **run** / prompt> **r**
    - 実行コンテキストがセットされている場合、実行する
  * - prompt> **r test < foo.txt**
    - $argv[1] == "test" 、foo.txt を STDIN として実行する

.. _phpdbg-break:

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

.. _phpdbg-watch:

watch(w)
^^^^^^^^

* 変数定義が有効な間、変数にウォッチポイントを設定する
* 引数が与えられない場合、現在アクティブなウォッチポイントの一覧を表示する

.. list-table:: 変数の書式
  :widths: 10 40
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
  :widths: 10 10 40
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

.. _phpdbg-clear:

clear(C)
^^^^^^^^

　ブレークポイントを削除します。 ブレークポイントを削除すると、そのコードを割り込みなしにもう一度実行可能となります。

* break delete N を使うと、特定のブレークポイントを削除できます。
* すべてのブレークポイントがクリアされると、PHP スクリプトは正常終了するまで実行されます。

.. _phpdbg-clean:

clean(X)
^^^^^^^^

　PHP でクラス、定数、関数を宣言できるのは１度だけです。デバッグ中に PHP をリコンパイルする際、エラーが起こることがあります。clean コマンドはコンパイル後のクラス、停止位、関数を保持している Zend 実行時テーブルをクリアし、関連するストレージをストレージプールに戻します。これによりリコンパイルを行えるようになります。

　これらのリソースプールを選択的にクリアすることはできません。完全にクリーンな状態に戻すことだけが可能です。


C-2-3.その他のコマンド
----------------------

.. list-table::
  :widths: 10 50
  :header-rows: 1

  * - コマンド名（省略形）
    - 概要
  * - :ref:`phpdbg-set`
    - phpdbg の設定値をセットする
  * - :ref:`phpdbg-source`
    - phpdbginit スクリプトを実行する
  * - :ref:`phpdbg-register`
    - phpdbginit 関数をコマンドエイリアスとして登録する
  * - :ref:`phpdbg-sh`
    - シェルのコマンドを実行する
  * - :ref:`phpdbg-ev`
    - PHP のコードを評価(eval)する
  * - quit(q)
    - phpdbg を抜ける

.. _phpdbg-set:

set(S)
^^^^^^^

.. list-table:: サブコマンド一覧
  :widths: 15 15 30
  :header-rows: 1

  * - サブコマンド
    - エイリアス
    - 用途／書式
  * - prompt
    - p
    - プロンプトを設定する
  * - color
    - c
    - set color  <element> <color>
  * - colors
    - C
    - set colors [<on|off>]
  * - oplog
    - O
    - set oplog [output]
  * - break
    - b
    - set break id <on|off>
  * - breaks
    - B
    - set breaks [<on|off>]
  * - quiet
    - q
    - set quiet [<on|off>]
  * - stepping
    - s
    - set stepping [<opcode|line>]
  * - refcount
    - r
    - set refcount [<on|off>]

　有効な color は none, white, red, green, yellow, blue, purple, cyan, black です。none 以外の色には後ろに -bold と -underline 修飾子を付加できます。

　color element は prompt, notice, error のいずれかです。

.. list-table:: set の使用例
  :widths: 20 50
  :header-rows: 1

  * - 使用例
    - 説明
  * - prompt> **S C on**
    - カラー表示制御を有効にします。
  * - | prompt>  **set p >**
      | prompt>  **set color prompt white-bold**
    - プロンプト文字列を白の太字の > にします。
  * - prompt>  **S c error red-bold**
    - エラー表示を赤の太字にします。
  * - prompt>  **S refcount on**
    - | ウォッチポイントにヒットした際、
      | refcount を表示するようにします。
  * - prompt>  S b 4 off
    - | ブレークポイント #4 を一時的に無効にします。
      | これは後に s b 4 on で再度有効にできます。

.. _phpdbg-source:

source(<)
^^^^^^^^^

　phpdbginit を実行します。デバッグセッションの中で phpdbginit を source するようにすれば時間の節約になる場合があります。

.. list-table:: source(<)の使用例
  :widths: 20 30
  :header-rows: 1

  * - 使用例
    - 説明
  * - | prompt>  **source /my/init**
      | prompt>  **< /my/init**
    - /my/init にある phpdbginit を実行します


.. _phpdbg-register:

register(R)
^^^^^^^^^^^

　グローバル関数を登録して phpdbg コンソール上でコマンドとして実行できるようにします。

.. list-table:: register(R)の使用例
  :widths: 20 30
  :header-rows: 1

  * - 使用例
    - 説明
  * - | prompt>  **register scandir**
      | prompt>  **R scandir**
    - phpdbg で使えるように scandir 関数を登録します。


.. _phpdbg-sh:

sh
^^

　シェルコマンドに直接アクセスし、ウィンドウやコンソールへの切り替えを省略できます。

.. list-table:: sh の使用例
  :widths: 20 30
  :header-rows: 1

  * - 使用例
    - 説明
  * - prompt>  **sh ls /usr/src/php-src**
    - ls /usr/src/php-src を実行し、その出力をコンソールに表示します。

.. _phpdbg-ev:

ev
^^

　ev コマンドは文字列式を受け取ってそれを評価し、結果を表示します。事前に :ref:`phpdbg-frame` コマンドで明示的に切り替えていない限り、実行中の最下層のフレームのコンテキストで評価が行われます。

.. list-table:: ev の使用例
  :widths: 20 30
  :header-rows: 1

  * - 使用例
    - 説明
  * - prompt>  **ev $variable**
    - | $variable が定義されていれば、コンソール上で
      | print_r($variable) を実行します。
  * - prompt>  **ev $variable = "Hello phpdbg :)"**
    - 現在のスコープで $variable をセットします。

* ev は、代入、関数コールその他の変更系ステートメントを含む、あらゆる有効な PHP 評価式を受け付けます。結果的に、実行中の環境が変更される可能性もあるので留意してください。ブレークポイントが設定されている PHP 関数をコールすることも可能です。
* ev は常に結果を表示しますので、コードの前に return を置かないようにしてください。

C-3.利用例
==========

　以下のような PHP スクリプトを用意しました。このファイルを使って、利用方法を見てみましょう。

.. code-block:: bash
  :emphasize-lines: 1

  $ cat simple-copy.php
  <?php
  $a = "new string";
  $b = $a;

　スクリプトファイル名を引数として起動します。

.. code-block:: bash
  :emphasize-lines: 1

  $ phpdbg simple-copy.php
  [Welcome to phpdbg, the interactive PHP debugger, v0.5.0]
  To get help using phpdbg type "help" and press enter
  [Please report bugs to <http://bugs.php.net/report.php>]
  [Successful compilation of /home/vagrant/temp/simple-copy.php]
  prompt>

　起動時のメッセージでわかるように、'prompt>' が表示された時点ですでに PHP スクリプトのコンパイルは完了しています。phpdbg では PHP スクリプトの行単位、もしくはコンパイル後のオペコード単位でステップ実行できます。

.. code-block:: bash
  :emphasize-lines: 1

  prompt> l 3
   00001: <?php
   00002: $a = "new string";
   00003: $b = $a;

　gdb と異なり、:ref:`phpdbg-list` には引数が必要です。

.. code-block:: bash
  :emphasize-lines: 1,3

  prompt> b if $a === "new string"
  [Conditional breakpoint #0 added $a === "new string"/0x7fd4d5673100]
  prompt> run
  [Conditional breakpoint #0: on $a === "new string" == true at /home/vagrant/temp/simple-copy.php:3, hits: 1]
  >00003: $b = $a;
   00004:

　ブレークポイントを条件で設定して実行。3 行目で停止しました。

.. code-block:: bash
  :emphasize-lines: 1,3

  prompt> p o
  [L2       0x7fabc1e75000 ASSIGN                  $a                   "new string"                              /home/vagrant/temp/simple-copy.php]
  prompt> p e
  [Context /home/vagrant/temp/simple-copy.php (3 ops)]
  L1-4 {main}() /home/vagrant/temp/simple-copy.php - 0x7fabc1e75000 + 3 ops
   L2    #0     ASSIGN                  $a                   "new string"         
   L3    #1     ASSIGN                  $b                   $a                   
   L4    #2     RETURN                  1                                 

　Zend Engine レベルのオペコードを表示してみました。

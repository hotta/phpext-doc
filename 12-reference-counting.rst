=================
12.参照カウント法
=================

12.1.はじめに
=============

　参照は、同じ値を複数のシンボル（変数名等）で共有するためのしくみです。PHP では、たとえば文字列変数の値が単に参照された場合、参照された側の zval が指しているオブジェクトの参照回数（リファレンス・カウンタ - gc.refcount) を 1 増やします。参照している変数がもはや使われなくなった場合、参照回数を 1 減らします。

　このような制御を「参照カウント法（リファレンス・カウントティング）」と呼んでいます。これにより、処理結果の値が変わらない場合、値の無駄なコピーを防いでメモリの利用効率や実行速度を稼いでいます。

　参照が返されるのは、以下の場合が挙げられます。

* 変数名に '&' を付けた場合
* 変数を引数として関数を呼び出した場合
* オブジェクトを new した場合
* オブジェクト内で使用する \$this

　以下に、refcount の動きを確認するためのテストプログラムを記載しますので参考にしてください。使用している `xdebug_debug_zval()` 関数は、 `Xdebug <https://xdebug.org/>`_ に含まれています。

テストプログラム::

  $ cat ref_test.php
  <?php
  echo 'PHP_VERSION = ' . PHP_VERSION . PHP_EOL;
  $a = "new string";
  echo 'TEST1: after $a = "new string"' . PHP_EOL;
  xdebug_debug_zval('a', 'b', 'c');
  $b = $a;
  echo 'TEST2: after $b = $a' . PHP_EOL;
  xdebug_debug_zval('a', 'b', 'c');
  $c = &$a;
  echo 'TEST3: after $c = &$a' . PHP_EOL;
  xdebug_debug_zval('a', 'b', 'c');
  $a = "another string";
  echo 'TEST4: after $a = "another string"' . PHP_EOL;
  xdebug_debug_zval('a', 'b', 'c');
  $c = "further string";
  echo 'TEST5: after $c = "further string"' . PHP_EOL;
  xdebug_debug_zval('a', 'b', 'c');
  echo 'TEST6: after dump($a, $b, $c)' . PHP_EOL;
  dump($a, $b, $c);
  
  function dump(&$x, &$y, &$z) {
    xdebug_debug_zval('x', 'y', 'z');
  }

実行結果::

  $ php ref_test.php
  PHP_VERSION = 7.1.5
  TEST1: after $a = "new string"
  a: (refcount=0, is_ref=0)='new string'
  b: (refcount=0, is_ref=0)=*uninitialized*
  c: (refcount=0, is_ref=0)=*uninitialized*
  TEST2: after $b = $a
  a: (refcount=0, is_ref=0)='new string'
  b: (refcount=0, is_ref=0)='new string'
  c: (refcount=0, is_ref=0)=*uninitialized*
  TEST3: after $c = &$a
  a: (refcount=2, is_ref=1)='new string'
  b: (refcount=0, is_ref=0)='new string'
  c: (refcount=2, is_ref=1)='new string'
  TEST4: after $a = "another string"
  a: (refcount=2, is_ref=1)='another string'
  b: (refcount=0, is_ref=0)='new string'
  c: (refcount=2, is_ref=1)='another string'
  TEST5: after $c = "further string"
  a: (refcount=2, is_ref=1)='further string'
  b: (refcount=0, is_ref=0)='new string'
  c: (refcount=2, is_ref=1)='further string'
  TEST6: after dump($a, $b, $c)
  x: (refcount=4, is_ref=1)='further string'
  y: (refcount=2, is_ref=1)='new string'
  z: (refcount=4, is_ref=1)='further string'

　参照に関するマクロについて、以下に記載します。

.. list-table:: 参照カウント関連マクロ
  :header-rows: 1

  * - No.
    - マクロ定義
    - 意味・用途
  * - 1
    - Z_REFCOUNT_P(pz)
    - (\*(pz)).value.counted)->gc.refcount を返す
  * - 2
    - GC_TYPE(p)
    - (p)->gc.u.v.type を返す
  * - 3
    - GC_FLAGS(p)
    - (p)->gc.u.v.flags を返す（文字列やオブジェクトで使われる）
  * - 4
    - GC_INFO(p)
    - (p)->gc.u.v.gc_info を返す(keeps GC root number (or 0) and color)
  * - 5
    - GC_TYPE_INFO(p)
    - (p)->gc.u.type_info を返す
  * - 6
    - | Z_GC_TYPE(zval) / 
      | Z_GC_TYPE_P(zval_p)
    - ((&zval).value.counted)->gc.u.v.type を返す
  * - 7 
    - | Z_GC_FLAGS(zval) / 
      | Z_GC_FLAGS_P(zval_p)
    - ((&zval).value.counted)->gc.u.v.flags を返す
  * - 8
    - | Z_GC_INFO(zval) / 
      | Z_GC_INFO_P(zval_p)
    - ((&zval).value.counted)->gc.u.v.gc_info を返す
  * - 9
    - | Z_GC_TYPE_INFO(zval) /
      | Z_GC_TYPE_INFO_P(zval_p)
    - ((&zval).value.counted)->gc.u.v.type_info を返す

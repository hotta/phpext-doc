====================
8.PHP_FUNCTIONの詳細
====================

8.1.PHP_FUNCTIONマクロ
======================

　my_echo_int は、最終的に以下のようになりました（再掲）。

.. code-block:: c

  /* {{{ proto void my_echo_int(int arg)
    コンソールに関数名と引数を表示します。
  */
  PHP_FUNCTION(my_echo_int)
  {
    zend_long arg;
  
    if (ZEND_NUM_ARGS() < 1 || 1 < ZEND_NUM_ARGS()) {
      WRONG_PARAM_COUNT;
    }
    if (zend_parse_parameters(ZEND_NUM_ARGS(), "l", &arg) == FAILURE) {
      return;
    }
    my_echo_int(arg);
  }

　PHP_FUNCTION マクロの定義は以下のようになっています（一部空白を編集）。

.. code-block:: bash
  :emphasize-lines: 1,3,5,7,9

  ~/php$ grep -rw 'define PHP_FUNCTION' .
  ./main/php.h:#define PHP_FUNCTION                   ZEND_FUNCTION
  ~/php$ grep -rw 'define ZEND_FUNCTION' .
  ./Zend/zend_API.h:#define ZEND_FUNCTION(name)       ZEND_NAMED_FUNCTION(ZEND_FN(name))
  ~/php$ grep -rw 'define ZEND_FN' .
  ./Zend/zend_API.h:#define ZEND_FN(name) zif_##name
  ~/php$ grep -rw 'define ZEND_NAMED_FUNCTION' .
  ./Zend/zend_API.h:#define ZEND_NAMED_FUNCTION(name) void name(INTERNAL_FUNCTION_PARAMETERS)
  ~/php$ grep -rw 'define INTERNAL_FUNCTION_PARAMETERS' .
  ./Zend/zend.h:#define INTERNAL_FUNCTION_PARAMETERS zend_execute_data *execute_data, zval *return_value

　``PHP_FUNCTION(my_echo_int)`` は、プリプロセッサにより最終的に

::

  void zif_my_echo_int(zend_execute_data *execute_data, zval *return_value)

と展開されることがわかりました。以下、これを ``my_echo_int() 疑似関数`` と呼ぶことにします。

8.2.引数の受け取り
==================

　疑似関数では、まず ``ZEND_NUM_ARGS()`` マクロを使って引数の数をチェックします。引数の数が誤っている場合、 ``WRONG_PARAM_COUNT`` マクロを使います。これは内部的に引数エラーを発生させて、そのまま上位にリターンします。これらのマクロは Zend/zend_API.h で定義されています。引数の数が合っていれば次に進みます。

　今回の疑似関数は、（PHP の）int 型の引数を１つ取ります。疑似関数で引数を受け取るには、まず引数に対応するローカル変数を宣言します。PHP の int 型に対応するのは zend_long 型です。

　次に zend_parse_parameters() で引数を受け取ります。この説明は次節で行います。

　後は C の世界ですので、C の外部関数 ``my_echo_int()`` にそのまま引数を渡してやるだけです。zend_long は Zend/zend_long.h に定義がありますが、事実上 int と同じなので、キャストなしにそのまま渡せます。

8.3.zend_parse_parameters()
===========================

　この関数のプロトタイプは Zend/zend_API.h で以下のように定義されています。

.. code-block:: c

  ZEND_API int zend_parse_parameters(int num_args, const char *type_spec, ...);

　num_args には ZEND_NUM_ARGS() を渡します。type_spec は、Zend Engine における型を表す文字を引数の順に並べた書式文字列です。３番目以降のパラメーターには、引数を受け取るための個々の変数へのポインタを列挙します。

8.3.1.書式文字列（型）
----------------------

　type_spec に指定可能な、書式文字列を以下に示します。指定文字が 'f, 'O', 'p', 's' のケースでは、２つの内部変数を使って値を受け取ります。

.. list-table:: 書式文字列（型）
  :widths: 10 40 40
  :header-rows: 1

  * - 指定文字
    - 引数の型
    - 代入対象の変数の型
  * - a
    - array
    - zval *
  * - A
    - array または object
    - zval *
  * - b
    - boolean
    - zend_bool
  * - C
    - class
    - zend_class_entry *
  * - d
    - double
    - double 
  * - f
    - function または method
    - zend_fcall_info \*, zend_fcall_info_cache \*
  * - h
    - array
    - HashTable *
  * - H
    - array または HASH_OF(object)
    - HashTable *
  * - l
    - long
    - zend_long
  * - L
    - long (LONG_MAX/LONG_MIN)
    - zend_long
  * - o
    - object
    - zval *
  * - O
    - object（クラス定義した時）
    - zval \*, zend_class_entry *
  * - p
    - string（NULLを含まない）
    - char \*, size_t
  * - P
    - 有効なパス
    - zend_string *
  * - r 
    - resource
    - char *
  * - s
    - string（NULLを含んでよい）
    - char \*, size_t 
  * - S
    - zend_string（NULLを含んでよい）
    - zend_string *
  * - z
    - mixed（実際は zval）
    - zval *

8.3.2.書式文字列（修飾文字）
----------------------------

　type_spec には、以下の修飾文字も指定可能です。

.. list-table:: 書式文字列（修飾文字）
  :widths: 10 90
  :header-rows: 1

  * - 修飾文字
    - 説明
  * - \*
    - 可変引数リスト（0 以上）
  * - \+
    - 可変引数リスト（1 以上）
  * - \|
    - | 残りのパラメーターは任意指定（省略可能）であることを示します。
      | これらが渡されない場合でもパース関数は特になにもしないので、
      | Extension が責任を持ってデフォルト値に初期化してやらなければなりません。
  * - /
    - これ以降のパラメーターに対して SEPARATE_ZVAL_IF_NOT_REF() を使います。
  * - !
    - | これ以降のパラメーターは、指定の型の他に NULL を指定できます。
      | NULL が渡されて、さらにこのような型に対する出力がポインターの場合、
      | 出力ポインターは（zend_null ではなく）ネイティブの NULL ポインターとなります。

8.3.3.書式文字列（注意事項）
----------------------------

* zend_bool* 型の追加の引数である 'b', 'l', 'd' は、順に bool*、zend_long*、double* 型の引数の後ろに置かなければなりません。
* PHP の NULL が渡されると、zend_bool には非ゼロ値が書かれます。

8.3.4.書式文字列の使用例
------------------------

　~/php/ext/ 配下から、いくつかの関数について引数の受け取り方の例を見てみましょう。

posix_kill
^^^^^^^^^^

.. php:function:: bool posix_kill ( int $pid , int $sig )
  プロセスにシグナルを送信するする

実装
  php/ext/posix/posix.c::

    PHP_FUNCTION(posix_kill)
    {
      zend_long pid, sig;
    
      if (zend_parse_parameters(ZEND_NUM_ARGS(), "ll", &pid, &sig) == FAILURE) {
        RETURN_FALSE;
      }

posix_mknod
^^^^^^^^^^^^

.. php:function:: bool posix_mknod ( string $pathname , int $mode [, int $major = 0 [, int $minor = 0 ]] )
  スペシャルファイルあるいは通常のファイルを作成する (POSIX.1)

実装
  php/ext/posix/posix.c::

    PHP_FUNCTION(posix_mknod)
    {
        char *path;
        size_t path_len;
        zend_long mode;
        zend_long major = 0, minor = 0;
        int result;
        dev_t php_dev;
    
        php_dev = 0;
    
        if (zend_parse_parameters(ZEND_NUM_ARGS(), "pl|ll", &path, &path_len,
                &mode, &major, &minor) == FAILURE) {
            RETURN_FALSE;
        }

curl_multi_info_read
^^^^^^^^^^^^^^^^^^^^

.. php:function:: array curl_multi_info_read ( resource $mh [, int &$msgs_in_queue = NULL ] )
  現在の転送についての情報を表示する

実装
  php/ext/curl/multi.c::

    PHP_FUNCTION(curl_multi_info_read)
    {
        zval      *z_mh;
        php_curlm *mh;
        CURLMsg   *tmp_msg;
        int        queued_msgs;
        zval      *zmsgs_in_queue = NULL;
    
        if (zend_parse_parameters(ZEND_NUM_ARGS(), "r|z/", &z_mh, &zmsgs_in_queue) == FAILURE) {
            return;
        }
        
SQLite3::openBlob
^^^^^^^^^^^^^^^^^

.. php:function:: public resource SQLite3::open ( string $table , string $column \
  , int $rowid [, string $dbname = "main" ] )
  ストリームリソースをオープンして BLOB を読み込む

実装
  php/ext/sqlite3/sqlite3.c::

    PHP_METHOD(sqlite3, openBlob)
    {
        php_sqlite3_db_object *db_obj;
        zval *object = getThis();
        char *table, *column, *dbname = "main";
        size_t table_len, column_len, dbname_len;
        zend_long rowid, flags = 0;
        sqlite3_blob *blob = NULL;
        php_stream_sqlite3_data *sqlite3_stream;
        php_stream *stream;
    
        db_obj = Z_SQLITE3_DB_P(object);
    
        SQLITE3_CHECK_INITIALIZED(db_obj, db_obj->initialised, SQLite3)
    
        if (zend_parse_parameters(ZEND_NUM_ARGS(), "ssl|s", 
          &table, &table_len, &column, &column_len, &rowid, &dbname, 
          &dbname_len) == FAILURE) {
            return;
        }

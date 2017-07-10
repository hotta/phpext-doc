==============
3.ひな形の作成
==============

3.1.ext ディレクトリの構成
==========================

　php のソースツリーの中の ext ディレクトリ配下には、PHP に配布物に標準で同梱されている Extension があります。

.. code-block:: bash
  :emphasize-lines: 1,2

  ~/php$ cd ext
  ~/php/ext$ ls -F
  bcmath/             gd/         oci8/          posix/       sysvsem/
  bz2/                gettext/    odbc/          pspell/      sysvshm/
  calendar/           gmp/        opcache/       readline/    tidy/
  com_dotnet/         hash/       openssl/       recode/      tokenizer/
  ctype/              iconv/      pcntl/         reflection/  wddx/
  curl/               imap/       pcre/          session/     xml/
  date/               interbase/  pdo/           shmop/       xmlreader/
  dba/                intl/       pdo_dblib/     simplexml/   xmlrpc/
  dom/                json/       pdo_firebird/  skeleton/    xmlwriter/
  enchant/            ldap/       pdo_mysql/     snmp/        xsl/
  exif/               libxml/     pdo_oci/       soap/        zip/
  ext_skel*           mbstring/   pdo_odbc/      sockets/     zlib/
  ext_skel_win32.php  mcrypt/     pdo_pgsql/     spl/
  fileinfo/           my_ext/     pdo_sqlite/    sqlite3/
  filter/             mysqli/     pgsql/         standard/
  ftp/                mysqlnd/    phar/          sysvmsg/

　今回、configure 時に何も Extension の指定をしなかったので、生成された php バイナリに組み込まれているのはこの中の standard 配下など一部のものだけとなります。組み込まれているモジュールは、 ``php -m`` で確認できます。

.. code-block:: bash
  :emphasize-lines: 1

  ~/php/ext$ php -m
  [PHP Modules]
  Core
  ctype
  date
  dom
  fileinfo
  filter
  hash
  iconv
  json
  libxml
  pcre
  PDO
  pdo_sqlite
  Phar
  posix
  Reflection
  session
  SimpleXML
  SPL
  sqlite3
  standard
  tokenizer
  xml
  xmlreader
  xmlwriter
  
  [Zend Modules]

　新たに作成する Extension の名前は、これらのモジュール名や ext 配下のディレクトリ名と重複しないように決定する必要があります。ここでは、新しい Extension の名前を "my_ext" とします。

3.2.ひな形の作成
================

　./ext_skel コマンドを実行し、現在のシステム構成をベースにして、新しい Extension ``my_ext`` のためのスケルトン（ひな形）を作成します。

.. code-block:: bash
  :emphasize-lines: 1

  ~/php/ext$ ./ext_skel --extname=my_ext --skel=./skeleton
  reating directory my_ext
  Creating basic files: config.m4 config.w32 .gitignore my_ext.c php_my_ext.h CREDITS EXPERIMENTAL tests/001.phpt my_ext.php [done].
  
  To use your new extension, you will have to execute the following steps:
  
  1.  $ cd ..
  2.  $ vi ext/my_ext/config.m4
  3.  $ ./buildconf
  4.  $ ./configure --[with|enable]-my_ext
  5.  $ make
  6.  $ ./sapi/cli/php -f ext/my_ext/my_ext.php
  7.  $ vi ext/my_ext/my_ext.c
  8.  $ make
  
  Repeat steps 3-6 until you are satisfied with ext/my_ext/config.m4 and step 6 confirms that your module is compiled into PHP. Then, start writing code and repeat the last two steps as often as necessary.

　ext/my_ext ディレクトリが作られ、その中にひな形のコードが出力されます。

.. code-block:: bash
  :emphasize-lines: 1,2

  ~/php/ext$ cd my_ext/
  ~/php/ext/my_ext$ ls
  CREDITS       config.m4   my_ext.c    php_my_ext.h
  EXPERIMENTAL  config.w32  my_ext.php  tests/

　特に重要なファイルは以下の通りです。

config.m4
  my_ext 専用の configure コマンドを作るための元になるスクリプトです。

my_ext.c
  Extension の本体となる C のソースファイルです。

my_ext.php
  Extension の基本動作を確認するための PHP スクリプトです。

php_my_ext.h
  Extension のビルドに必要な C のヘッダファイルです。

tests
  Extension のテストをするためのディレクトリです。この中にテストコードを入れていきます。PHP の標準では \*.phpt 形式のテストを採用していますが、今回は PHPUnit を使います。

3.3.config.m4 の修正
====================

　config.m4 は UNIX 用の伝統的なマクロ・プリプロセッサである M4 の文法で書かれています。 ``dnl`` で始まる行はコメントです。まずは、ビルドするために必要最小限の部分のコメントを外して有効にします。これ以降、変更分は diff の出力として記載します。 ``'<'`` が変更前、 ``'>'`` が変更後の内容を表します。

.. code-block:: bash
  :emphasize-lines: 1,2

  ~/php/ext/my_ext$ vi config.m4
  ~/php/ext/my_ext$ git diff config.m4
  16,18c16,18
  < dnl PHP_ARG_ENABLE(my_ext, whether to enable my_ext support,
  < dnl Make sure that the comment is aligned:
  < dnl [  --enable-my_ext           Enable my_ext support])
  ---
  > PHP_ARG_ENABLE(my_ext, whether to enable my_ext support,
  > Make sure that the comment is aligned:
  > [  --enable-my_ext           Enable my_ext support])

　これにより、PHP_ARG_ENABLE マクロと ``--enable-my_ext`` ビルドオプションが有効になりました。このように、単に Extension を有効にしたい場合（特にパラメーター指定が不要の場合）は ``--enable-XXX`` オプションを使います。ライブラリの場所等、何らかのパラメーターを指定できるようにしたい場合は ``--with-XXX`` オプションを使います。

3.4.はじめてのビルド
====================

　phpize （autoconf のラッパー）により config.m4 から configure コマンドを生成します。

.. code-block:: bash
  :emphasize-lines: 1,6

  ~/php/ext/my_ext$ phpize
  Configuring for:
  PHP Api Version:         20160303
  Zend Module Api No:      20160303
  Zend Extension Api No:   320160303
  ~/php/ext/my_ext$ ls
  CREDITS          autom4te.cache  config.sub    ltmain.sh      php_my_ext.h
  EXPERIMENTAL     build           config.w32    missing        run-tests.php
  Makefile.global  config.guess    configure     mkinstalldirs  tests
  acinclude.m4     config.h.in     configure.in  my_ext.c
  aclocal.m4       config.m4       install-sh    my_ext.php

　生成された configure コマンドは、my_ext 専用です。``--enable-my_ext`` オプションが有効になっていることを確認後、ビルドしてみます。

.. code-block:: bash
  :emphasize-lines: 1,3,6,10

  ~/php/ext/my_ext$ ./configure --help | grep my_ext
    --enable-my_ext           Enable my_ext support
  ~/php/ext/my_ext$ ./configure --enable-my_ext
  （中略）
  config.status: creating config.h
  ~/php/ext/my_ext$ make
  （中略）
  Build complete.
  Don\'t forget to run 'make test'.
  ~/php/ext/my_ext$ ls modules/
  my_ext.la  my_ext.so

　ビルドに成功したら、modules 配下に my_ext.so が作られます。

3.5.はじめての実行
==================

　ext_skel により作成されたサンプルスクリプト my_ext.php を使って、Extension が正しく作られたかどうかを確認します。まだシステムグローバルでは my_ext.so を認識できていないので、コマンドライン引数で Extension の共有ライブラリファイルを指定して起動します。

.. code-block:: bash
  :emphasize-lines: 1

  ~/php/ext/my_ext$ php -d extension=modules/my_ext.so my_ext.php
  Functions available in the test extension:
  confirm_my_ext_compiled
  
  Congratulations! You have successfully modified ext/my_ext/config.m4. Module my_ext is now compiled into PHP.

　正常に実行されたようです。my_ext.php の中身は以下のようになっています。

.. code-block:: bash
  :emphasize-lines: 1

  ~/php/ext/my_ext$ cat -n my_ext.php
     1  <?php
     2  $br = (php_sapi_name() == "cli")? "":"<br>";
     3
     4  if(!extension_loaded('my_ext')) {
     5          dl('my_ext.' . PHP_SHLIB_SUFFIX);
     6  }
     7  $module = 'my_ext';
     8  $functions = get_extension_funcs($module);
     9  echo "Functions available in the test extension:$br\n";
    10  foreach($functions as $func) {
    11      echo $func."$br\n";
    12  }
    13  echo "$br\n";
    14  $function = 'confirm_' . $module . '_compiled';
    15  if (extension_loaded($module)) {
    16          $str = $function($module);
    17  } else {
    18          $str = "Module $module is not compiled into PHP";
    19  }
    20  echo "$str\n";
    21  ?>

　実行結果と見比べてみましょう。出力の２行目の ``confirm_my_ext_compiled`` は、11 行目の echo $func の出力結果です。またこれと同じ文字列を 14 行目で生成して $function に代入し、16 行目でこの変数を介して動的に ``confirm_my_ext_compiled('my_ext')`` を呼び出しています。 ``Congraturations! ...`` は confirm_my_ext_compiled() 関数の中で出力されているようです。この関数の実体は、my_php.c の中で以下のように定義されています。

.. code-block:: bash
  :emphasize-lines: 1

  ~/php/ext/my_ext$ sed -n '54,68p' my_ext.c
  PHP_FUNCTION(confirm_my_ext_compiled)
  {
          char *arg = NULL;
          size_t arg_len, len;
          zend_string *strg;
  
          if (zend_parse_parameters(ZEND_NUM_ARGS(), "s", &arg, &arg_len) == FAILURE) {
                  return;
          }
  
          strg = strpprintf(0, "Congratulations! You have successfully modified ext/%.78s/config.m4. Module %.78s is now compiled into PHP.", "my_ext", arg);
  
          RETURN_STR(strg);
  }
  /* }}} */


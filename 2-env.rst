================
2.開発環境の準備
================

2.1.推奨環境
============

　筆者は `Vagrant <http://qiita.com/ozawan/items/160728f7c6b10c73b97e>`_ + `VirtualBox <https://www.virtualbox.org/>`_ の環境の上に CentOS 7.3 の VM を建てて、その上で開発をしています。必要な環境を構築するためのツールや手順は `ansible <http://qiita.com/t_nakayama0714/items/fe55ee56d6446f67113c>`_ で管理しています。推奨環境を構築するには `https://github.com/hotta/ansible-centos7 <https://github.com/hotta/ansible-centos7>`_ を参照して環境を構築し、リファレンスとして紹介している laravel.yml の代わりに::

  $ ansible-playbook /etc/ansible/jobs/php-ext.yml

を実行してください。laravel.yml を流してしまうと依存関係で PHP 等も入ってしまい、開発の際に混乱することがあるのでお勧めしません。

　上記以外の環境を使いたい場合、上記リポジトリの `roles/php-ext/tasks/main.yml <https://github.com/hotta/ansible-centos7/blob/master/roles/php-ext/tasks/main.yml>`_ や、これが依存する `roles/base/tasks/mandatory.yml <https://github.com/hotta/ansible-centos7/blob/master/roles/base/tasks/mandatory.yml>`_ に必要なコンポーネントが列挙してありますので、適宜読み替えてください。

　以下、vagrant ユーザーでログインして作業していることを前提としています。

2.2.PHPのインストール
=====================

　PHP は tarball から以下の要領でインストールします。::

  ~$ wget -O php-7.1.5.tar.bz2 http://jp2.php.net/get/php-7.1.5.tar.bz2/from/this/mirror
  ~$ tar xjf php-7.1.5.tar.bz2 
  ~$ sudo mv php-7.1.5 /usr/local/src
  ~$ ln -fs /usr/local/src/php-7.1.5 php
  ~$ cd php
  ~/php$ ./configure --enable-debug --with-openssl --enable-mbstring --readline-devel
  （中略）
  Thank you for using PHP.
  ...
  config.status: executing default commands
  ~/php$ make
  （中略）
  Build complete.
  Don't forget to run 'make test'.
  ~/php$ sudo make install
  ~/php$ php -v
  PHP 7.1.5 (cli) (built: Jun  7 2017 09:13:29) ( NTS DEBUG )
  Copyright (c) 1997-2017 The PHP Group
  Zend Engine v3.1.0, Copyright (c) 1998-2017 Zend Technologies

　PHP のビルド手順について、少し説明します。これから開発する Extension のビルドも、これと似たような操作を行います。

configure
  システムの差異を吸収して、このシステムでビルドするための Makefile を作成します。 ``--enable-debug`` オプションは、Extension 開発には必須です。これを付けることで、Zend Engine がメモリリークを検出して報告できるようになります。メモリリークを検出すると、PHP スクリプトの実行時に以下のような出力がなされます。::

    [Fri May 26 13:43:46 2017]  Script:  '/home/vagrant/temp/b.php' /home/vagrant/php/ext/my_ext/my_ext.c(326) :  Freeing 0x00007f7dd6402fc0 (25 bytes), script=/home/vagrant/temp/b.php
    [Fri May 26 13:43:46 2017]  Script:  '/home/vagrant/temp/b.php' /home/vagrant/php/ext/my_ext/my_ext.c(256) :  Freeing 0x00007f7dd6472028 (8 bytes), script=/home/vagrant/temp/b.php
    === Total 2 memory leaks detected ===

  　 ``--with-openssl`` / ``--enable-mbstring`` / ``--with-readline`` は PHP Extension とは直接関係ありませんが、後からテストやデバッグの際に必要になったので追加してあります。

  　./configure でエラーになる場合は、必要な依存パッケージが入っていること、また PHP のパッケージ版が入ったりしていないことを確認してください。

make
  Makefile を読み込んでコンパイルとリンクを行います。これらの作業はまとめて「ビルド」と呼ばれます。今後頻繁に使用することになる php コマンドは、インストール前には以下のところにあります。::

    ~/php$ find . -name php
    sapi/cli/php

sudo make install
  ビルド結果をシステムにインストール（適切な場所に設置）します。sapi 配下の実行ファイルは /usr/local/bin に置かれます。このディレクトリは一般的にパスが通っているので、ここにあるファイルは他のユーザーからも実行可能となります。


2.3.成果物について
==================

　/usr/local/bin 置かれたファイルは以下の通りです。::

    root:/usr/local/bin# ls -l
    total 56452
    lrwxrwxrwx 1 root root        9 Jun  7 09:32 phar -> phar.phar
    -rwxr-xr-x 1 root root    53490 Jun  7 09:32 phar.phar
    -rwxr-xr-x 1 root root 19051768 Jun  7 09:32 php
    -rwxr-xr-x 1 root root 18954640 Jun  7 09:32 php-cgi
    -rwxr-xr-x 1 root root     2245 Jun  7 09:32 php-config
    -rwxr-xr-x 1 root root 19723064 Jun  7 09:32 phpdbg
    -rwxr-xr-x 1 root root     4551 Jun  7 09:32 phpize

　これらについて、簡単にご紹介します。

phar
  PHAR (PHP Archiver) 形式のファイルを管理したり、phar 形式のインストーラーを実行します。今回は使用しません。詳細については `Phar アーカイブの使用法 <http://php.net/manual/ja/phar.using.php>`_ を参照してください。

php-config / phpize
  これらは共に、Extension 開発に必要となるシェルスクリプトです。

php / php-cgi / phpdbg
  これらは PHP 本体（実行用バイナリ）です。PHP にはさまざまな起動方法がありますが、それらが SAPI (Server API) というレイヤーで抽象化され、それぞれに対応した PHP バイナリが生成されます。たとえば configure の ``--with-apxs`` オプションは、Apache 2.0 Handler モジュール用の共有ライブラリを作成します（今回は指定していないので出力されていません）。

php
  CLI（コマンドライン）版と呼ばれるもので、開発の際は主にこれを使います。

php-cgi
  Web サーバーから CGI / FastCGI 経由で実行する場合に使用される実行ファイルです。

phpdbg
  PHP スクリプトをコマンドラインからステップ実行したり、もっと細かい Zend VM のオペコード単位で実行したりするための SAPI モジュールです。興味のある方は `PHP による hello world 入門 <http://tech.respect-pal.jp/php-helloworld/>`_ が詳しいです。



1日目: プロジェクトを始める
===========================

.. include:: common/original.rst.inc

Jobeet とは
-----------

| `Jobeet` はオープンソースのジョブボードのソフトウェアです。一日づつチュートリアルをこなし、最新の Web テクノロジーである Symfony 2.3.21 を学ぶことを手助けします。
| (まだ知らない人のために、SymfonyはPHPのためのフレームワークです。)
| それぞれのチャプター/日は、約 1 時間ででき、開始から終了まで、本物のウェブサイトをコーディングすることによって Symfony を学ぶことが出来ます。
| 毎日、新しい機能がアプリケーションに追加され、新しい Symfony の機能性だけでなく、 Symfony の Web 開発におけるグッドプラクティスを紹介し、この開発の利点を学びます。
| 今日は初日となり、コードは書きません。その代わり開発環境のセットアップを行います。


開発環境のセットアップ
----------------------

| 初めに、コンピューターがweb開発環境として適しているか確認する必要があります。
| VMware Player の仮想マシンにインストールされている Ubuntu 12.04 LTS Server を使用します。最低でも、 Web サーバ（例えば Apache ）、データベースエンジン（ MySQL ）と PHP 5.3.3 以降が必要です。

1. Webサーバー Apache をインストールします。:

   .. code-block:: bash

       $ sudo apt-get install apache2

   次に、 Apache mod-rewrite を有効にします。

   .. code-block:: bash

       $ sudo a2enmod rewrite

2. MySQLサーバをインストールします。

   .. code-block:: bash

       $ sudo apt-get install mysql-server mysql-client

3. サーバースクリプト言語、 PHP をインストールします。

   .. code-block:: bash

       $ sudo apt-get install php5 libapache2-mod-php5 php5-mysql

4. 国際化拡張機能をインストールします。

   .. code-block:: bash

       $ sudo apt-get install php5-intl

5. ここで、Apache サービスを再起動する必要があります。

   .. code-block:: bash

       $ sudo service apache2 restart

Symfony 2.3.2 のダウンロードとインストール
------------------------------------------

| まず最初にすることは、新しいプロジェクトをインストールする場所を Web サーバ上のディレクトリを準備することです。
| それを Jobeet ( /var/www/jobeet ) と呼びましょう。

.. code-block:: bash

    $ mkdir /var/www/jobeet

| ディレクトリを用意しましたが、それに何を入れましょう？
| http://symfony.com/download に進み、ベンダーのない Symfony Standard 2.3.2 を選択し、それをダウンロードします。
| 準備ディレクトリ Jobeet に symfony のディレクトリ内のファイルを解凍します。

Vendors の更新
~~~~~~~~~~~~~~

| この時点で、独自アプリケーション開発をはじめる、フル機能の Symfony のプロジェクトをダウンロードしました。
| Symfony のプロジェクトは、多くの外部ライブラリに依存します。
| それらは、 Composer と呼ばれるライブラリを経由して、プロジェクトの ``vendor/`` ディレクトリにダウンロードされます。
| Composer は PHP の依存管理ライブラリで、 Symfony 2.3.2 スタンダード版をダウンロードするために使用することができます。
| Jobeet のディレクトリ上に Composer をダウンロードすることではじめます。

.. code-block:: bash

    $ curl -s https://getcomposer.org/installer | php

curl 拡張をインストールしていない場合は、このコマンドを使用してインストールすることができます。

.. code-block:: bash

    $ sudo apt-get install curl

次に、必要なすべてのベンダー·ライブラリのダウンロードを開始するには、次のコマンドを入力します。

.. code-block:: bash

    $ php composer.phar install

Web サーバーの設定
------------------

| 良い Web の作法は、 Web ルートディレクトリの下にスタイルシート、 JavaScript と画像のように、Web ブラウザがアクセスする必要のあるファイルだけを置くことです。
| デフォルトでは、 Symfony プロジェクトの web/ サブディレクトリの下にこれらのファイルを保存することをお勧めします。
| 新しいプロジェクトのために Apache を設定するには、バーチャルホストを作成します。
| そのためには、使用しているターミナルで次のコマンドをタイプします。

.. code-block:: bash

    $ sudo nano /etc/apache2/sites-available/jobeet.local

これで、 jobeet.local という名前のファイルが作成されます。以下をそのファイルの中に追記し、`Control – O` を選択し、それを保存します。その後、 `Control – X` でエディタを終了します。

/etc/apache2/sites-available/jobeet.local

.. code-block:: apache

    <VirtualHost *:80>
        ServerName jobeet.local
        DocumentRoot /var/www/jobeet/web
        DirectoryIndex app.php
        ErrorLog /var/log/apache2/jobeet-error.log
        CustomLog /var/log/apache2/jobeet-access.log combined
        <Directory "/var/www/jobeet/web">
            AllowOverride All
            Allow from All
         </Directory>
     </VirtualHost>

| Apache の設定で使用されるドメイン名 jobeet.local は、ローカルで宣言する必要があります。
| Linux システムで実行する場合、``/etc/hosts`` ファイルで行う必要があります。
| Windowsで実行している場合、このファイルは ``C:\Windows\System32\drivers\etc\`` ディレクトリにあります。次の行を追加します。

.. code-block:: bash

    127.0.0.1 jobeet.local

.. tip::

   リモート·サーバーで作業している場合には、127.0.0.1をWebサーバー·マシンのIPで置き換えてください。

| この仮想ホストを動かしたい場合は、新しく作成された仮想ホストを有効にして Apache を再起動する必要があります。
| そのため、ターミナルに移って次のようにタイプします。

.. code-block:: bash

    $ sudo a2ensite jobeet.local
    $ sudo service apache2 restart

| Web サーバーと PHP が Symfony を使用できるよう正常に設定されているかどうか確認する視覚的なサーバー設定テスターが Symfony に付属しています。
| 設定を確認するには、次のURLを使用します。

http://jobeet.local/config.php

.. image:: /images/sf2-config-1.png

ローカルホストからこれを実行しない場合、web/config.php ファイルを開いてローカルホストの外部からのアクセスを制限する行をコメントにします。

web/config.php

.. code-block:: php

   if (!isset($_SERVER['HTTP_HOST'])) {
      exit('This script cannot be run from the CLI. Run it from a browser.');
   }
   /*
   if (!in_array(@$_SERVER['REMOTE_ADDR'], array(
      '127.0.0.1',
      '::1',
   ))) {
      header('HTTP/1.0 403 Forbidden');
      exit('This script is only accessible from localhost.');
   }
   */
   // ...

web/app_dev.php に対しても同じことを行います。

web/app_dev.php

.. code-block:: php

   use Symfony\Component\HttpFoundation\Request;
   use Symfony\Component\Debug\Debug;

   // If you don't want to setup permissions the proper way, just uncomment the following PHP line
   // read http://symfony.com/doc/current/book/installation.html#configuration-and-setup for more information
   //umask(0000);

   // This check prevents access to debug front controllers that are deployed by accident to production servers.
   // Feel free to remove this, extend it, or make something more sophisticated.
   /*
   if (isset($_SERVER['HTTP_CLIENT_IP'])
       || isset($_SERVER['HTTP_X_FORWARDED_FOR'])
       || !in_array(@$_SERVER['REMOTE_ADDR'], array('127.0.0.1', 'fe80::1', '::1'))
   ) {
       header('HTTP/1.0 403 Forbidden');
       exit('You are not allowed to access this file. Check '.basename(__FILE__).' for more information.');
   }
   */

   $loader = require_once __DIR__.'/../app/bootstrap.php.cache';
   Debug::enable();

   require_once __DIR__.'/../app/AppKernel.php';

   // ...

おそらく、config.php を開くことですべての種類の必要条件を取得できるでしょう。以下は、それらすべての ``warnings`` が出ないようにする作業リストです。

1. app/cache と、app/logs のパーミッションを変更します。

   .. code-block:: bash

      sudo chmod -R 777 app/cache
      sudo chmod -R 777 app/logs
      sudo setfacl -dR -m u::rwX app/cache app/logs

   まだ ACL を持っていない場合はインストールします。

   .. code-block:: bash

      sudo apt-get install acl

2. php.iniで date.timezone を設定します

   /etc/php5/apache2/php.ini

   .. code-block:: ini

      date.timezone = Europe/Bucharest

   .. code-block:: bash

      sudo nano /etc/php5/apache2/php.ini

   [date] セクションの date.timezone を見つけて、タイムゾーンを設定します。その後、";" を削除し、行の先頭に配置します。

3. 同じ php.ini ファイルで short_open_tag をオフに設定します。

   /etc/php5/apache2/php.ini

   .. code-block::

      short_open_tag
        Default Value: Off

4. PHP アクセラレータ（APC推奨）をインストールし有効にします。

   .. code-block:: bash

      sudo apt-get install php-apc
      sudo service apache2 restart

   Apache を再起動した後、 ブラウザを開き http://jobeet.local/app_dev.php とタイプします。次のページが表示されます。

   .. image:: /images/Day-1-SF_welcome.jpg

Symfony2 のコンソール
---------------------

| Symfony2 には、さまざまなタスクに使用するコンソールコンポーネントのツールが付属しています。
| コマンドプロンプトで入力できることをリスト表示するには以下のように入力します。

.. code-block:: bash

    $ php app/console list

Application Bundle の作成
-------------------------

bundleとは正確には何か
~~~~~~~~~~~~~~~~~~~~~~

| 他のソフトウェアのプラグインと似ていますが、それよりも良いです。
| 主な違いは、すべてが Symfony 2.3.2 のバンドルということです。コアフレームワークの機能とアプリケーションのために書かれたコードの両方を含みます。
| バンドルは、単一の機能を実装した構造化されたディレクトリ内のファイルのセットです。

.. note::

   バンドルはオートロード（ app/autoload.php ）される限りどこにでも設置できます。

.. note::

   ここに詳細を読むことができます。
   バンドルシステム http://symfony.com/doc/current/book/page_creation.html#the-bundle-system


基本的 bundle スケルトンの作成
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Symfony のバンドルジェネレーターを開始するには、次のコマンドを実行します。

.. code-block:: bash

    $ php app/console generate:bundle --namespace=Ibw/JobeetBundle

ジェネレーターは、バンドルを生成する前にいくつか質問をします。ここでの質問と回答（ 1 つを除くすべてがデフォルトの答え）は次のとおりです。

.. code-block:: bash

    Bundle name [IbwJobeetBundle]: IbwJobeetBundle
    Target directory [/var/www/jobeet/src]: /var/www/jobeet/src
    Configuration format (yml, xml, php, or annotation) [yml]: yml
    Do you want to generate the whole directory structure [no]? yes
    Do you confirm generation [yes]? yes
    Confirm automatic update of your Kernel [yes]? yes
    Confirm automatic update of the Routing [yes]? yes

新しいバンドルを生成した後にキャッシュをクリアします。

.. code-block:: bash

    $ php app/console cache:clear --env=prod
    $ php app/console cache:clear --env=dev

| 現在、新しい Jobeet のバンドル は、プロジェクトの src ディレクトリにあります（ ``src/Ibw/JobeetBundle`` ）。
| バンドルジェネレータは DefaultController を index アクションとあわせて作りました。
| これにはお使いのブラウザで次のようにアクセスすることができます。
| http://jobeet.local/hello/jobeet または http://jobeet.local/app_dev.php/hello/jobeet

AcmeDemoBundle の削除の仕方
~~~~~~~~~~~~~~~~~~~~~~~~~~~

| Symfony 2.3.2 Standard Editionでは完全なデモが AcmeDemoBundle バンドルの中に付属しています。
| これは、プロジェクトの開始時に参照するためのすばらしい文例集ですが、おそらく、最終的にそれを削除したいと思うでしょう。

1. ``Acme`` ディレクトリを削除するコマンドを入力します。

   .. code-block:: bash

       $ rm -rf /var/www/jobeet/src/Acme

2. /var/www/jobeet/app/AppKernel.php を開いて行の削除をします。

   app/AppKernel.php

   .. code-block:: php

      // ...

      $bundles[] = new Acme\DemoBundle\AcmeDemoBundle();

      // ...

   そして app/config/routing_dev.yml から削除します。

   app/config/routing_dev.yml

   .. code-block:: yaml

      # ...

      # AcmeDemoBundle routes (to be removed)
      _acme_demo:
          resource: "@AcmeDemoBundle/Resources/config/routing.yml"

3. 最後に、キャッシュをクリアします。


環境
----

| Symfony 2.3.2 は、いくつかの環境を持っています。プロジェクトのwebディレクトリを見ると、app.phpとapp_dev.phpという2つの PHP ファイルを見るでしょう。
| これらのファイルは、フロントコントローラーと呼ばれます。アプリケーションへのすべてのリクエストは、それらを介して行われます。
| app.php ファイルは本番環境用であり、 app_dev.php はウェブ開発者によって使用される開発環境です。
| 開発環境は、すべてのエラーと警告とWebデバッグツールバーが表示されますので非常に便利だと分かるでしょう。開発者の親友です。
| これで今日はすべてです。明日は Jobeet の Web サイトが正確にどんなものかお話します。このチュートリアルでお会いしましょう！

.. seealso::
    *Symfony2日本語ドキュメント*

    豊富な日本語ドキュメントがありますので合わせて読み進めてみましょう。

    * `ガイドブック »Symfony のインストールと設定 <http://docs.symfony.gr.jp/symfony2/book/installation.html>`_


.. include:: common/license.rst.inc

9日目: 機能テスト(ファンクショナルテスト)
=========================================

.. include:: common/original.rst.inc

| 機能テスト(ファンクショナルテスト)は、端から端まで（ブラウザのリクエストからサーバーのレスポンスまで）アプリケーションをテストするための素晴らしいツールです。
| それらはアプリケーションのすべての層（ルーティング、モデル、アクションとテンプレート）をテストします。
| おそらく既に手動でやっていることと非常に似ています。
| アクションを追加または変更するたびに、ブラウザでページを開き、リンクをクリックし表示されたページの要素をチェックして、すべてが期待どおりに動作することを確認する必要があります。
| 言い換えると、実装したばかりのユースケースに対応するシナリオを実行します。
| これは手作業なので、退屈で間違いをしやすいです。
| コードのどこかを変更するたびに、何かを壊していないことを保証するため、すべてのシナリオを行わなければなりません。
| それは非常識です。 Symfonyでは機能テストはシナリオを簡単に書く方法を提供します。
| 各シナリオは、ユーザーがブラウザで体験することをシミュレートすることを再度自動的に何度も再生することができます。
| 単体テストのように、コードに自信を与えてくれます。
| 機能テストは、非常に明確なワークフローがあります。

..
   Functional tests are a great tool to test your application from end to end: from the request made by a browser to the response sent by the server.
   They test all the layers of an application: the routing, the model, the actions and the templates.
   They are very similar to what you probably already do manually: each time you add or modify an action, you need to go to the browser and check that everything works as expected by clicking on links and checking elements on the rendered page.
   In other words, you run a scenario corresponding to the use case you have just implemented.
   As the process is manual, it is tedious and error prone.
   Each time you change something in your code, you must step through all the scenarios to ensure that you did not break something.
   That’s insane. Functional tests in symfony provide a way to easily describe scenarios.
   Each scenario can then be played automatically over and over again by simulating the experience a user has in a browser.
   Like unit tests, they give you the confidence to code in peace.
   Functional tests have a very specific workflow:

* リクエストを作成します。
* リンクをクリックするか、フォームを送信します。
* レスポンスをテストします。
* すすぎ、繰り返す。

..
   * Make a request;
   * Test the response;
   * Click on a link or submit a form;
   * Test the response;
   * Rinse and repeat;

最初の機能テスト
----------------

| 一般的に機能テストはバンドルのディレクトリ Tests/Controller にあるシンプルな PHP ファイルです。
| CategoryController クラスによって処理するページをテストする場合は、特別な WebTestCase クラスを拡張して新しい CategoryControllerTest クラスを作成することで始めます。

..
   Functional tests are simple PHP files that typically live in the Tests/Controller directory of your bundle.
   If you want to test the pages handled by your CategoryController class, start by creating a new CategoryControllerTest class that extends a special WebTestCase class:

src/Ibw/JobeetBundle/Tests/Controller/CategoryControllerTest.php

.. code-block:: php

   <?php
   namespace Ibw\JobeetBundle\Tests\Controller;

   use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
   use Symfony\Bundle\FrameworkBundle\Console\Application;
   use Symfony\Component\Console\Output\NullOutput;
   use Symfony\Component\Console\Input\ArrayInput;
   use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;

   class CategoryControllerTest extends WebTestCase
   {
       private $em;
       private $application;

       public function setUp()
       {
           static::$kernel = static::createKernel();
           static::$kernel->boot();

           $this->application = new Application(static::$kernel);

           // drop the database
           $command = new DropDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:drop',
               '--force' => true
           ));
           $command->run($input, new NullOutput());

           // we have to close the connection after dropping the database so we don't get "No database selected" error
           $connection = $this->application->getKernel()->getContainer()->get('doctrine')->getConnection();
           if ($connection->isConnected()) {
               $connection->close();
           }

           // create the database
           $command = new CreateDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:create',
           ));
           $command->run($input, new NullOutput());

           // create schema
           $command = new CreateSchemaDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:schema:create',
           ));
           $command->run($input, new NullOutput());

           // get the Entity Manager
           $this->em = static::$kernel->getContainer()
               ->get('doctrine')
               ->getManager();

           // load fixtures
           $client = static::createClient();
           $loader = new \Symfony\Bridge\Doctrine\DataFixtures\ContainerAwareLoader($client->getContainer());
           $loader->loadFromDirectory(static::$kernel->locateResource('@IbwJobeetBundle/DataFixtures/ORM'));
           $purger = new \Doctrine\Common\DataFixtures\Purger\ORMPurger($this->em);
           $executor = new \Doctrine\Common\DataFixtures\Executor\ORMExecutor($this->em, $purger);
           $executor->execute($loader->getFixtures());
       }

       public function testShow()
       {
           $client = static::createClient();

           $crawler = $client->request('GET', '/category/index');
           $this->assertEquals('Ibw\JobeetBundle\Controller\CategoryController::showAction', $client->getRequest()->attributes->get('_controller'));
           $this->assertTrue(200 === $client->getResponse()->getStatusCode());
       }
   }

クローラーの詳細については、 Symfony ドキュメントをお読みください。

.. To learn more about crawler, read the Symfony documentation here.

機能テストの実行
----------------

単体テスト（ユニットテスト）と同様に、 PHPUnit のコマンドを実行することで機能テストを実行することができます。

.. As for unit tests, launching functional tests can be done by executing the phpunit command:

.. code-block:: bash

   $ phpunit -c app/ src/Ibw/JobeetBundle/Tests/Controller/CategoryControllerTest

.. This test will fail because the tested url, /category/index, is not a valid url in Jobeet

このテストは、テスト URL ( /category/index ) が Jobeet の内の有効な URL ではないため失敗します。::

   PHPUnit 3.7.22 by Sebastian Bergmann.

   Configuration read from /var/www/jobeet/app/phpunit.xml.dist

   F

   Time: 2 seconds, Memory: 25.25Mb

   There was 1 failure:

   1) Ibw\JobeetBundle\Tests\Controller\CategoryControllerTest::testShow
   Failed asserting that false is true.

機能テストを書く
----------------

| 機能テストを書くことはブラウザでシナリオを演じることに似ています。
| すでに 2 日目のストーリーとしてテストする必要のあるすべてのシナリオは書かかれています。
| まずは、 JobControllerTest クラスを編集して Jobeet のホームページをテストしてみましょう。 次のコードで置き換えます。

..
   Writing functional tests is like playing a scenario in a browser. We already have written all the scenarios we need to test as part of the day 2 stories.
   First, let’s test the Jobeet homepage by editing the JobControllerTest class. Replace the code with the following one:

期限切れのジョブが一覧表示されないこと
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   <?php
   namespace Ibw\JobeetBundle\Tests\Controller;

   use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
   use Symfony\Bundle\FrameworkBundle\Console\Application;
   use Symfony\Component\Console\Output\NullOutput;
   use Symfony\Component\Console\Input\ArrayInput;
   use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;

   class JobControllerTest extends WebTestCase
   {
       private $em;
       private $application;

       public function setUp()
       {
           static::$kernel = static::createKernel();
           static::$kernel->boot();

           $this->application = new Application(static::$kernel);

           // drop the database
           $command = new DropDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:drop',
               '--force' => true
           ));
           $command->run($input, new NullOutput());

           // we have to close the connection after dropping the database so we don't get "No database selected" error
           $connection = $this->application->getKernel()->getContainer()->get('doctrine')->getConnection();
           if ($connection->isConnected()) {
               $connection->close();
           }

           // create the database
           $command = new CreateDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:create',
           ));
           $command->run($input, new NullOutput());

           // create schema
           $command = new CreateSchemaDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:schema:create',
           ));
           $command->run($input, new NullOutput());

           // get the Entity Manager
           $this->em = static::$kernel->getContainer()
               ->get('doctrine')
               ->getManager();

           // load fixtures
           $client = static::createClient();
           $loader = new \Symfony\Bridge\Doctrine\DataFixtures\ContainerAwareLoader($client->getContainer());
           $loader->loadFromDirectory(static::$kernel->locateResource('@IbwJobeetBundle/DataFixtures/ORM'));
           $purger = new \Doctrine\Common\DataFixtures\Purger\ORMPurger($this->em);
           $executor = new \Doctrine\Common\DataFixtures\Executor\ORMExecutor($this->em, $purger);
           $executor->execute($loader->getFixtures());
       }

       public function testIndex()
       {
           $client = static::createClient();
           $crawler = $client->request('GET', '/');

           $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::indexAction', $client->getRequest()->attributes->get('_controller'));
           $this->assertTrue($crawler->filter('.jobs td.position:contains("Expired")')->count() == 0);
       }
   }

| ホームページから期限切れのジョブの除外を検証するために、 CSS セレクタ ``.jobs td.position:contains("Expired")`` が、
| レスポンスの中の HTML コンテンツ内のどれとも一致しないことを確認します。
| （フィクスチャーの中で唯一、有効期限が切れたジョブは、役職に「 Expired 」が含まれていたことを思い出してください）。

..
   To verify the exclusion of expired jobs from the homepage,
   we check that the CSS selector .jobs td.position:contains("Expired") does not match anywhere in the response HTML content
   (remember that in the fixtures, the only expired job we have contains “Expired” in the position).

N個のジョブだけが、カテゴリの一覧表示されること
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| testIndex() 関数の最後に次のコードを追加します。
| 機能テストで app/config/config.yml で定義されたカスタムパラメータを取得するには、カーネルを使用します。

..
   Add the following code at the end of  your testIndex() function.
   To get the custom parameter defined in app/config/config.yml in our functional test, we will use the kernel:

src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   public function testIndex()
   {
       //...
       $kernel = static::createKernel();
       $kernel->boot();
       $max_jobs_on_homepage = $kernel->getContainer()->getParameter('max_jobs_on_homepage');
       $this->assertTrue($crawler->filter('.category_programming tr')->count() <= $max_jobs_on_homepage );
   }

| このテストを動作させるために Job/index.html.twig の中で、カテゴリ毎に対応する CSS クラスを追加する必要があります。
| （それによって、各カテゴリを選択し、リストされたジョブを数えることができます）。

..
   For this test to work we will need to add the corresponding CSS class to each category in the Job/index.html.twig file
    (so we can select each category and count the jobs listed) :

src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig

.. code-block:: html+jinja

   <!-- ... -->

       {% for category in categories %}
           <div class="category_{{ category.slug }}">
              <div class="category">
   <!-- ... -->

カテゴリにジョブが多すぎる場合カテゴリページへのリンクのみを持つこと
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   public function testIndex()
   {
       //...
       $this->assertTrue($crawler->filter('.category_design .more_jobs')->count() == 0);
       $this->assertTrue($crawler->filter('.category_programming .more_jobs')->count() == 1);
   }

| これらのテストでは、デザインカテゴリに「 more jobs 」リンクがないこと(``.category_design .more_jobs`` がないこと)を確認し、
| プログラミングカテゴリには「 more jobs 」リンクがあること(``.category_programming .more_jobs`` があること)をチェックします。

..
   In these tests, we check that there is no “more jobs” link for the design category (.category_design .more_jobs does not exist),
    and that there is a “more jobs” link for the programming category (.category_programming .more_jobs does exist).

ジョブが日付でソートされること
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| ジョブが実際に日付でソートされているかどうかをテストするために、
| ホームページに記載された最初のジョブが、期待したものであることを確認する必要があります。
| URL に期待する主キーが含まれていることを確認することで行います。
| 主キーは実行の間に変わる可能性があるため、最初にデータベースから Doctrine のオブジェクトを取得する必要があります。

..
   To test if jobs are actually sorted by date, we need to check that the first job listed on the homepage is the one we expect.
   This can be done by checking that the URL contains the expected primary key.
   As the primary key can change between runs, we need to get the Doctrine object from the database first.

src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   public function testIndex()
   {
       // ...
       $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');

       $query = $em->createQuery('SELECT j from IbwJobeetBundle:Job j LEFT JOIN j.category c WHERE c.slug = :slug AND j.expires_at > :date ORDER BY j.created_at DESC');
       $query->setParameter('slug', 'programming');
       $query->setParameter('date', date('Y-m-d H:i:s', time()));
       $query->setMaxResults(1);
       $job = $query->getSingleResult();

       $this->assertTrue($crawler->filter('.category_programming tr')->first()->filter(sprintf('a[href*="/%d/"]', $job->getId()))->count() == 1);
   }

| テストがこの瞬間には動作したとしても、最初のプログラミングカテゴリのジョブを取得する処理をどのテストでも再利用できるよう、
| コードを少しリファクタリングする必要があります。
| コードはテスト固有のもののため、モデル·レイヤーにコードを移動しません。
| 代わりに、テストクラス内の getMostRecentProgrammingJob() メソッドにコードを移動します。

..
   Even if the test works in this very moment, we need to refactor the code a bit, as getting the first job of the programming category can be reused elsewhere in our tests.
   We won’t move the code to the Model layer as the code is test specific.
   Instead, we will move the code to the getMostRecentProgrammingJob function in our test class:

src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   // ...

       public function getMostRecentProgrammingJob()
       {
           $kernel = static::createKernel();
           $kernel->boot();
           $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');

           $query = $em->createQuery('SELECT j from IbwJobeetBundle:Job j LEFT JOIN j.category c WHERE c.slug = :slug AND j.expires_at > :date ORDER BY j.created_at DESC');
           $query->setParameter('slug', 'programming');
           $query->setParameter('date', date('Y-m-d H:i:s', time()));
           $query->setMaxResults(1);

           return $query->getSingleResult();
       }

   // ...

前のテストコードを次のもので置き換えることができます。

.. You can now replace the previous test code by the following one:

src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   // ...

   $this->assertTrue($crawler->filter('.category_programming tr')->first()->filter(sprintf('a[href*="/%d/"]', $this->getMostRecentProgrammingJob()->getId()))->count() == 1);

   //...

ホームページの各ジョブはクリックできること
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| ホームページの求人リンクをテストするには、「Web Developer」テキストをクリックすることをシミュレートします。
| それらの多くがページに存在するので、明示的に最初のものをクリックするようにブラウザに依頼します。
| 各リクエストパラメータは、ルーティングが正しく動いていることを確認するためにテストされます。

..
   To test the job link on the homepage, we simulate a click on the “Web Developer” text.
   As there are many of them on the page, we have explicitly to ask the browser to click on the first one.
   Each request parameter is then tested to ensure that the routing has done its job correctly.

src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   public function testIndex()
   {
       // ...

       $job = $this->getMostRecentProgrammingJob();
       $link = $crawler->selectLink('Web Developer')->first()->link();
       $crawler = $client->click($link);
       $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::showAction', $client->getRequest()->attributes->get('_controller'));
       $this->assertEquals($job->getCompanySlug(), $client->getRequest()->attributes->get('company'));
       $this->assertEquals($job->getLocationSlug(), $client->getRequest()->attributes->get('location'));
       $this->assertEquals($job->getPositionSlug(), $client->getRequest()->attributes->get('position'));
       $this->assertEquals($job->getId(), $client->getRequest()->attributes->get('id'));
   }

   // ...

お手本から学ぶ
~~~~~~~~~~~~~~

| このセクションには、ジョブとカテゴリのページをテストするために必要なすべてのコードがあります。
| いくつかの新しいトリックをきちんと学ぶように、慎重にコードを読んでください。

..
   In this section, you have all the code needed to test the job and category pages.
   Read the code carefully as you may learn some new neat tricks:

src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   <?php
   namespace Ibw\JobeetBundle\Tests\Controller;

   use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
   use Symfony\Bundle\FrameworkBundle\Console\Application;
   use Symfony\Component\Console\Output\NullOutput;
   use Symfony\Component\Console\Input\ArrayInput;
   use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;

   class JobControllerTest extends WebTestCase
   {
       private $em;
       private $application;

       public function setUp()
       {
           static::$kernel = static::createKernel();
           static::$kernel->boot();

           $this->application = new Application(static::$kernel);

           // drop the database
           $command = new DropDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:drop',
               '--force' => true
           ));
           $command->run($input, new NullOutput());

           // we have to close the connection after dropping the database so we don't get "No database selected" error
           $connection = $this->application->getKernel()->getContainer()->get('doctrine')->getConnection();
           if ($connection->isConnected()) {
               $connection->close();
           }

           // create the database
           $command = new CreateDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:create',
           ));
           $command->run($input, new NullOutput());

           // create schema
           $command = new CreateSchemaDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:schema:create',
           ));
           $command->run($input, new NullOutput());

           // get the Entity Manager
           $this->em = static::$kernel->getContainer()
               ->get('doctrine')
               ->getManager();

           // load fixtures
           $client = static::createClient();
           $loader = new \Symfony\Bridge\Doctrine\DataFixtures\ContainerAwareLoader($client->getContainer());
           $loader->loadFromDirectory(static::$kernel->locateResource('@IbwJobeetBundle/DataFixtures/ORM'));
           $purger = new \Doctrine\Common\DataFixtures\Purger\ORMPurger($this->em);
           $executor = new \Doctrine\Common\DataFixtures\Executor\ORMExecutor($this->em, $purger);
           $executor->execute($loader->getFixtures());
       }

       public function getMostRecentProgrammingJob()
       {
           $kernel = static::createKernel();
           $kernel->boot();
           $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');

           $query = $em->createQuery('SELECT j from IbwJobeetBundle:Job j LEFT JOIN j.category c WHERE c.slug = :slug AND j.expires_at > :date ORDER BY j.created_at DESC');
           $query->setParameter('slug', 'programming');
           $query->setParameter('date', date('Y-m-d H:i:s', time()));
           $query->setMaxResults(1);

           return $query->getSingleResult();
       }

       public function getExpiredJob()
       {
           $kernel = static::createKernel();
           $kernel->boot();
           $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');

           $query = $em->createQuery('SELECT j from IbwJobeetBundle:Job j WHERE j.expires_at < :date');
           $query->setParameter('date', date('Y-m-d H:i:s', time()));
           $query->setMaxResults(1);

           return $query->getSingleResult();
       }

       public function testIndex()
       {
           // get the custom parameters from app config.yml
           $kernel = static::createKernel();
           $kernel->boot();
           $max_jobs_on_homepage = $kernel->getContainer()->getParameter('max_jobs_on_homepage');

           $client = static::createClient();

           $crawler = $client->request('GET', '/');
           $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::indexAction', $client->getRequest()->attributes->get('_controller'));

           // expired jobs are not listed
           $this->assertTrue($crawler->filter('.jobs td.position:contains("Expired")')->count() == 0);

           // only $max_jobs_on_homepage jobs are listed for a category
           $this->assertTrue($crawler->filter('.category_programming tr')->count()<= $max_jobs_on_homepage);
           $this->assertTrue($crawler->filter('.category_design .more_jobs')->count() == 0);
           $this->assertTrue($crawler->filter('.category_programming .more_jobs')->count() == 1);

           // jobs are sorted by date
           $this->assertTrue($crawler->filter('.category_programming tr')->first()->filter(sprintf('a[href*="/%d/"]', $this->getMostRecentProgrammingJob()->getId()))->count() == 1);

           // each job on the homepage is clickable and give detailed information
           $job = $this->getMostRecentProgrammingJob();
           $link = $crawler->selectLink('Web Developer')->first()->link();
           $crawler = $client->click($link);
           $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::showAction', $client->getRequest()->attributes->get('_controller'));
           $this->assertEquals($job->getCompanySlug(), $client->getRequest()->attributes->get('company'));
           $this->assertEquals($job->getLocationSlug(), $client->getRequest()->attributes->get('location'));
           $this->assertEquals($job->getPositionSlug(), $client->getRequest()->attributes->get('position'));
           $this->assertEquals($job->getId(), $client->getRequest()->attributes->get('id'));

           // a non-existent job forwards the user to a 404
           $crawler = $client->request('GET', '/job/foo-inc/milano-italy/0/painter');
           $this->assertTrue(404 === $client->getResponse()->getStatusCode());

           // an expired job page forwards the user to a 404
           $crawler = $client->request('GET', sprintf('/job/sensio-labs/paris-france/%d/web-developer', $this->getExpiredJob()->getId()));
           $this->assertTrue(404 === $client->getResponse()->getStatusCode());
       }
   }

src/Ibw/JobeetBundle/Tests/Controller/CategoryControllerTest.php

.. code-block:: php

   <?php
   namespace Ibw\JobeetBundle\Tests\Controller;

   use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
   use Symfony\Bundle\FrameworkBundle\Console\Application;
   use Symfony\Component\Console\Output\NullOutput;
   use Symfony\Component\Console\Input\ArrayInput;
   use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;

   class CategoryControllerTest extends WebTestCase
   {
       private $em;
       private $application;
       public function setUp()
       {
           static::$kernel = static::createKernel();
           static::$kernel->boot();

           $this->application = new Application(static::$kernel);

           // drop the database
           $command = new DropDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:drop',
               '--force' => true
           ));
           $command->run($input, new NullOutput());

           // we have to close the connection after dropping the database so we don't get "No database selected" error
           $connection = $this->application->getKernel()->getContainer()->get('doctrine')->getConnection();
           if ($connection->isConnected()) {
               $connection->close();
           }

           // create the database
           $command = new CreateDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:create',
           ));
           $command->run($input, new NullOutput());

           // create schema
           $command = new CreateSchemaDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:schema:create',
           ));
           $command->run($input, new NullOutput());

           // get the Entity Manager
           $this->em = static::$kernel->getContainer()
               ->get('doctrine')
               ->getManager();

           // load fixtures
           $client = static::createClient();
           $loader = new \Symfony\Bridge\Doctrine\DataFixtures\ContainerAwareLoader($client->getContainer());
           $loader->loadFromDirectory(static::$kernel->locateResource('@IbwJobeetBundle/DataFixtures/ORM'));
           $purger = new \Doctrine\Common\DataFixtures\Purger\ORMPurger($this->em);
           $executor = new \Doctrine\Common\DataFixtures\Executor\ORMExecutor($this->em, $purger);
           $executor->execute($loader->getFixtures());
       }

       public function testShow()
       {
           $kernel = static::createKernel();
           $kernel->boot();

           // get the custom parameters from app/config.yml
           $max_jobs_on_category = $kernel->getContainer()->getParameter('max_jobs_on_category');
           $max_jobs_on_homepage = $kernel->getContainer()->getParameter('max_jobs_on_homepage');

           $client = static::createClient();

           $categories = $this->em->getRepository('IbwJobeetBundle:Category')->getWithJobs();

           // categories on homepage are clickable
           foreach($categories as $category) {
               $crawler = $client->request('GET', '/');

               $link = $crawler->selectLink($category->getName())->link();
               $crawler = $client->click($link);

               $this->assertEquals('Ibw\JobeetBundle\Controller\CategoryController::showAction', $client->getRequest()->attributes->get('_controller'));
               $this->assertEquals($category->getSlug(), $client->getRequest()->attributes->get('slug'));

               $jobs_no = $this->em->getRepository('IbwJobeetBundle:Job')->countActiveJobs($category->getId());

               // categories with more than $max_jobs_on_homepage jobs also have a "more" link
               if($jobs_no > $max_jobs_on_homepage) {
                   $crawler = $client->request('GET', '/');
                   $link = $crawler->filter(".category_" . $category->getSlug() . " .more_jobs a")->link();
                   $crawler = $client->click($link);

                   $this->assertEquals('Ibw\JobeetBundle\Controller\CategoryController::showAction', $client->getRequest()->attributes->get('_controller'));
                   $this->assertEquals($category->getSlug(), $client->getRequest()->attributes->get('slug'));
               }

               $pages = ceil($jobs_no/$max_jobs_on_category);

               // only $max_jobs_on_category jobs are listed
               $this->assertTrue($crawler->filter('.jobs tr')->count() <= $max_jobs_on_category);
               $this->assertRegExp("/" . $jobs_no . " jobs/", $crawler->filter('.pagination_desc')->text());

               if($pages > 1) {
                   $this->assertRegExp("/page 1\/" . $pages . "/", $crawler->filter('.pagination_desc')->text());

                   for ($i = 2; $i <= $pages; $i++) {
                       $link = $crawler->selectLink($i)->link();
                       $crawler = $client->click($link);

                       $this->assertEquals('Ibw\JobeetBundle\Controller\CategoryController::showAction', $client->getRequest()->attributes->get('_controller'));
                       $this->assertEquals($i, $client->getRequest()->attributes->get('page'));
                       $this->assertTrue($crawler->filter('.jobs tr')->count() <= $max_jobs_on_category);
                       if($jobs_no >1) {
                           $this->assertRegExp("/" . $jobs_no . " jobs/", $crawler->filter('.pagination_desc')->text());
                       }
                       $this->assertRegExp("/page " . $i . "\/" . $pages . "/", $crawler->filter('.pagination_desc')->text());
                   }
               }
           }
       }
   }

これで、今日はすべてです！明日は、フォームについて、知っておくべきことを学びます。

.. That’s all for today! Tomorrow, we will learn all there is to know about forms.

.. seealso::
    *Symfony2日本語ドキュメント*

    豊富な日本語ドキュメントがありますので合わせて読み進めてみましょう。

    * `クックブック »テスト »Doctrine のリポジトリクラスをテストする方法  <http://docs.symfony.gr.jp/symfony2/cookbook/testing/doctrine.html>`_
    * `クックブック »テスト »データベース接続を伴うコードのテストについて  <http://docs.symfony.gr.jp/symfony2/cookbook/testing/database.html>`_




.. include:: common/license.rst.inc

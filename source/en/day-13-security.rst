Day 13: Security
================

.. include:: common/original.rst.inc

Securing the Application
------------------------

Security is a two-step process whose goal is to prevent a user from accessing a resource that he/she should not have access to. In the first step of the process, the authentication, the security system identifies who the user is by requiring the user to submit some sort of identification. Once the system knows who you are, the next step, called the authorization, is to determine if you should have access to a given resource (it checks to see if you have privileges to perform a certain action).
The security component can be configured via your application configuration using the security.yml file from the app/config folder. To secure our application change  your security.yml file:

app/config/security.yml

.. code-block:: yaml

   security:
       role_hierarchy:
           ROLE_ADMIN:       ROLE_USER
           ROLE_SUPER_ADMIN: [ROLE_USER, ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH]

       firewalls:
           dev:
               pattern:  ^/(_(profiler|wdt)|css|images|js)/
               security: false

           secured_area:
               pattern:    ^/
               anonymous: ~
               form_login:
                   login_path:  /login
                   check_path:  /login_check
                   default_target_path: ibw_jobeet_homepage

       access_control:
           - { path: ^/admin, roles: ROLE_ADMIN }

       providers:
           in_memory:
               memory:
                   users:
                       admin: { password: adminpass, roles: 'ROLE_ADMIN' }

       encoders:
           Symfony\Component\Security\Core\User\User: plaintext

This configuration will secure the /admin section of the website (all urls that start with /admin) and will allow only users with ROLE_ADMIN to access it (see the access_control section). In this example the admin user is defined in the configuration file (the providers section) and the password is not encoded (encoders).
For authenticating users, a traditional login form will be used, but we need to implement it. First, create two routes: one that will display the login form (i.e. /login) and one that will handle the login form submission (i.e. /login_check):

src/Ibw/JobeetBundle/Resources/config/routing.yml

.. code-block:: yaml

   login:
       pattern:   /login
       defaults:  { _controller: IbwJobeetBundle:Default:login }
   login_check:
       pattern:   /login_check

   # ...

We will not need to implement a controller for the /login_check URL as the firewall will automatically catch and process any form submitted to this URL. But you need to create a route so that it can be used  to generate the form submission URL in the login template below.
Next, let’s create the action that will display the login form:

src/Ibw/JobeetBundle/Controller/DefaultController.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Controller;

   use Symfony\Bundle\FrameworkBundle\Controller\Controller;
   use Symfony\Component\Security\Core\SecurityContext;

   class DefaultController extends Controller
   {
       // ...

       public function loginAction()
       {
           $request = $this->getRequest();
           $session = $request->getSession();

           // get the login error if there is one
           if ($request->attributes->has(SecurityContext::AUTHENTICATION_ERROR)) {
               $error = $request->attributes->get(SecurityContext::AUTHENTICATION_ERROR);
           } else {
               $error = $session->get(SecurityContext::AUTHENTICATION_ERROR);
               $session->remove(SecurityContext::AUTHENTICATION_ERROR);
           }

           return $this->render('IbwJobeetBundle:Default:login.html.twig', array(
               // last username entered by the user
               'last_username' => $session->get(SecurityContext::LAST_USERNAME),
               'error'         => $error,
           ));
       }
   }

When the user submits the form, the security system automatically handles the form submission for you. If the user had submitted an invalid username or password, this action reads the form submission error from the security system so that it can be displayed back to the user. Your only job is to display the login form and any login errors that may have occurred, but the security system itself takes care of checking the submitted username and password and authenticating the user.
Finally, let’s create the corresponding template:

src/Ibw/JobeetBundle/Resources/views/Default/login.html.twig

.. code-block:: html+jinja

   {% if error %}
       <div>{{ error.message }}</div>
   {% endif %}

   <form action="{{ path('login_check') }}" method="post">
       <label for="username">Username:</label>
       <input type="text" id="username" name="_username" value="{{ last_username }}" />

       <label for="password">Password:</label>
       <input type="password" id="password" name="_password" />

       <button type="submit">login</button>
   </form>

Now, if you try to access http://jobeet.local/app_dev.php/admin/dashboard url, the login form will show and you will have to enter the username and password defined in security.yml (admin/adminpass) to get to the admin section of Jobeet.

User Providers
--------------

During authentication, the user submits a set of credentials (usually a username and password). The job of the authentication system is to match those credentials against some pool of users. So where does this list of users come from?
In Symfony2, users can come from anywhere – a configuration file, a database table, a web service, or anything else you can dream up. Anything that provides one or more users to the authentication system is known as a “user provider”. Symfony2 comes standard with the two most common user providers: one that loads users from a configuration file and one that loads users from a database table.
Above, we used the first case: specifying users in a configuration file.

app/config/security.yml

.. code-block:: yaml

   # ...

   providers:
       in_memory:
           memory:
               users:
                   admin: { password: adminpass, roles: 'ROLE_ADMIN' }

   # ...

But you will usually want the users to be stored in a database table. To do this we will add a new user table to our jobeet database. First let’s create the orm for this new table:

src/Ibw/JobeetBundle/Resources/config/doctrine/User.orm.yml

.. code-block:: yaml

   Ibw\JobeetBundle\Entity\User:
       type: entity
       table: user
       id:
           id:
               type: integer
               generator: { strategy: AUTO }
       fields:
           username:
               type: string
               length: 255
           password:
               type: string
               length: 255

Now run the doctrine:generate:entities command to create the new User entity class:

.. code-block:: bash

   $ php app/console doctrine:generate:entities IbwJobeetBundle

And update the database:

.. code-block:: bash

   $ php app/console doctrine:schema:update --force

The only requirement for your new user class is that it implements the UserInterface interface. This means that your concept of a “user” can be anything, as long as it implements this interface. Open the User.php file and edit it as follows:

src/Ibw/JobeetBundle/Entity/User.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Entity;

   use Symfony\Component\Security\Core\User\UserInterface;
   use Doctrine\ORM\Mapping as ORM;

   /**
    * User
    */
   class User implements UserInterface
   {
       /**
        * @var integer
        */
       private $id;

       /**
        * @var string
        */
       private $username;

       /**
        * @var string
        */
       private $password;

       /**
        * Get id
        *
        * @return integer
        */
       public function getId()
       {
           return $this->id;
       }

       /**
        * Set username
        *
        * @param string $username
        * @return User
        */
       public function setUsername($username)
       {
           $this->username = $username;

       }

       /**
        * Get username
        *
        * @return string
        */
       public function getUsername()
       {
           return $this->username;
       }

       /**
        * Set password
        *
        * @param string $password
        * @return User
        */
       public function setPassword($password)
       {
           $this->password = $password;

       }

       /**
        * Get password
        *
        * @return string
        */
       public function getPassword()
       {
           return $this->password;
       }

       public function getRoles()
       {
           return array('ROLE_ADMIN');
       }

       public function getSalt()
       {
           return null;
       }

       public function eraseCredentials()
       {

       }

       public function equals(User $user)
       {
           return $user->getUsername() == $this->getUsername();
       }
   }

To the generated entity we added the methods required by the UserInterface class: getRoles, getSalt, eraseCredentials and equals.
Next, configure an entity user provider, and point it to your User class:

app/config/security.yml

.. code-block:: yaml

   providers:
     main:
         entity: { class: Ibw\JobeetBundle\Entity\User, property: username }

   encoders:
     Ibw\JobeetBundle\Entity\User: sha512

We also changed the encoder for our new User class to use the sha512 algorithm to encrypt passwords.
Now everything is set up but we need to create our first user. To do this we will create a new symfony command:

src/Ibw/JobeetBundle/Command/JobeetUsersCommand.php

.. code-block:: php

   <?php
   namespace Ibw\JobeetBundle\Command;

   use Symfony\Bundle\FrameworkBundle\Command\ContainerAwareCommand;
   use Symfony\Component\Console\Input\InputArgument;
   use Symfony\Component\Console\Input\InputInterface;
   use Symfony\Component\Console\Input\InputOption;
   use Symfony\Component\Console\Output\OutputInterface;
   use Ibw\JobeetBundle\Entity\User;

   class JobeetUsersCommand extends ContainerAwareCommand
   {
       protected function configure()
       {
           $this
               ->setName('ibw:jobeet:users')
               ->setDescription('Add Jobeet users')
               ->addArgument('username', InputArgument::REQUIRED, 'The username')
               ->addArgument('password', InputArgument::REQUIRED, 'The password')
           ;
       }

       protected function execute(InputInterface $input, OutputInterface $output)
       {
           $username = $input->getArgument('username');
           $password = $input->getArgument('password');

           $em = $this->getContainer()->get('doctrine')->getManager();

           $user = new User();
           $user->setUsername($username);
           // encode the password
           $factory = $this->getContainer()->get('security.encoder_factory');
           $encoder = $factory->getEncoder($user);
           $encodedPassword = $encoder->encodePassword($password, $user->getSalt());
           $user->setPassword($encodedPassword);
           $em->persist($user);
           $em->flush();

           $output->writeln(sprintf('Added %s user with password %s', $username, $password));
       }
   }

To add your first user run:

.. code-block:: bash

   $ php app/console ibw:jobeet:users admin admin

This will create the admin user with the password admin. You can use it to login to the admin section.

Logout
------

Logging out is handled automatically by the firewall. All you have to do is to activate the logout config parameter:

app/config/security.yml

.. code-block:: yaml

   security:
       firewalls:
           # ...
           secured_area:
               # ...
               logout:
                   path:   /logout
                   target: /
       # ...

You will not need to implement a controller for the /logout URL as the firewall takes care of everything. Let’s create a route so that you can use it to generate the URL:

src/Ibw/JobeetBundle/Resources/config/routing.yml

.. code-block:: yaml

   # ...

   logout:
       pattern:   /logout

   # ...

Once this is configured, sending a user to /logout (or whatever you configure the path to be), will un-authenticate the current user. The user will then be sent to the homepage (the value defined by the target parameter).
All left to do is to add the logout link to our admin section. To do this we will override the user_block.html.twig from SonataAdminBundle. Create the user_block.html.twig file in app/Resources/SonataAdminBundle/views/Core folder:

app/Resources/SonataAdminBundle/views/Core/user_block.html.twig

.. code-block:: html+jinja

   {% block user_block %}<a href="{{ path('logout') }}">Logout</a>{% endblock%}

Now, if you try to enter the admin section (clear the cache first), you will be asked for an username and password and then, the logout link will be shown in the top-right corner.

The User Session
----------------

Symfony2 provides a nice session object that you can use to store information about the user between requests. By default, Symfony2 stores the attributes in a cookie by using the native PHP sessions.
You can store and retrieve information from the session easily from the controller:

.. code-block:: php

   $session = $this->getRequest()->getSession();

   // store an attribute for reuse during a later user request
   $session->set('foo', 'bar');

   // in another controller for another request
   $foo = $session->get('foo');

Unfortunately, the Jobeet user stories have no requirement that includes storing something in the user session. So let’s add a new requirement: to ease job browsing, the last three jobs viewed by the user should be displayed in the menu with links to come back to the job page later on.
When a user access a job page, the displayed job object needs to be added in the user history and stored in the session:

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   // ...

   public function showAction($id)
   {
       $em = $this->getDoctrine()->getManager();

       $entity = $em->getRepository('IbwJobeetBundle:Job')->getActiveJob($id);

       if (!$entity) {
           throw $this->createNotFoundException('Unable to find Job entity.');
       }

       $session = $this->getRequest()->getSession();

       // fetch jobs already stored in the job history
       $jobs = $session->get('job_history', array());

       // store the job as an array so we can put it in the session and avoid entity serialize errors
       $job = array('id' => $entity->getId(), 'position' =>$entity->getPosition(), 'company' => $entity->getCompany(), 'companyslug' => $entity->getCompanySlug(), 'locationslug' => $entity->getLocationSlug(), 'positionslug' => $entity->getPositionSlug());

       if (!in_array($job, $jobs)) {
           // add the current job at the beginning of the array
           array_unshift($jobs, $job);

           // store the new job history back into the session
           $session->set('job_history', array_slice($jobs, 0, 3));
       }

       $deleteForm = $this->createDeleteForm($id);

       return $this->render('IbwJobeetBundle:Job:show.html.twig', array(
           'entity'      => $entity,
           'delete_form' => $deleteForm->createView(),
       ));
   }

In the layout, add the following code before the #content div:

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html+jinja

   <!-- ... -->

   <div id="job_history">
       Recent viewed jobs:
       <ul>
           {% for job in app.session.get('job_history') %}
               <li>
                   <a href="{{ path('ibw_job_show', { 'id': job.id, 'company': job.companyslug, 'location': job.locationslug, 'position': job.positionslug }) }}">{{ job.position }} - {{ job.company }}</a>
               </li>
           {% endfor %}
       </ul>
   </div>

   <div id="content">

   <!-- ... -->

Flash Messages
--------------

Flash messages are small messages you can store on the user’s session for exactly one additional request. This is useful when processing a form: you want to redirect and have a special message shown on the next request. We already used flash messages in our project when we publish a job:

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   // ...

   public function publishAction($token)
   {
       // ...

       $this->get('session')->getFlashBag()->add('notice', 'Your job is now online for 30 days.');

       // ...
   }

The first argument of the getFlashBag()->add() function is the identifier of the flash and the second one is the message to display. You can define whatever flashes you want, but notice and error are two of the more common ones.
To show the flash messages to the user you have to include them in the template. We did this in the layout.html.twig template:

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html+jinja

   <!-- ... -->

   {% for flashMessage in app.session.flashbag.get('notice') %}
       <div>
           {{ flassMessage }}
       </div>
   {% endfor %}

   <!-- ... -->

.. include:: common/license.rst.inc

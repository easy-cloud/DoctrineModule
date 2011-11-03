# SpiffyDoctrine Module for Zend Framework 2
* THIS BRANCH REQUIRES RALPH SCHINDLER'S HOTFIX FOR ZF2/DI, https://github.com/ralphschindler/zf2.git (a8de0890e216e9826be3414a818c333e2f307028) *

The SpiffyDoctrine module intends to integrate Doctrine 2 ORM with Zend Framework 2 quickly and easily. The following features are intended to work out of the box: 
  
  - Multiple ORM entity managers
  - Multiple DBAL connections
  - Caches for metadata, queries and resultsets
  - Using a SQL logger
  - Custom dql functions, additional hydration modes
  - Named DQL and native queries
  - Multiple metadata drivers
  - Annotations registries initialization (such as Gedmo DoctrineExtensions).
  - Validators for EntityExists and NoEntityExists.
  - Authentication adapter for Zend\Authenticator.
  
## Requirements
  - Zend Framework 2

## Installation
The simplest way to install is to clone the repository into your /modules directory add the 
SpiffyDoctrine key to your modules array before your Application module key.

  1. cd my/project/folder
  2. git clone https://SpiffyJr@github.com/SpiffyJr/SpiffyDoctrine.git modules/SpiffyDoctrine --recursive
  3. open my/project/folder/configs/application.config.php and add 'SpiffyDoctrine' to your 'modules' parameter.
  4. Alter the configuration (most likely your connection and entities path(s)) by adding the required changes to 
     my/project/folder/modules/Application/module.config.php.

## Example standard configuration
    // modules/Application/module.config.php
    'di' => array(
        'instance' => array(
            'doctrine-connection' => array(
                'parameters' => array(
                    'params' => array(
                        'driver'   => 'pdo_mysql',
                        'host'     => 'localhost',
                        'port'     => '3306', 
                        'user'     => 'YOUR_USER',
                        'password' => 'YOUR_PASSWORD',
                        'dbname'   => 'YOUR_DB_NAME',
                    ),
                ),
            ),
            'doctrine-configuration' => array(
                'parameters' => array(
                    'dir' => '/path/where/to/generate/proxies',
                ),
            ),
            'doctrine-annotationdriver' => array(
                'parameters' => array(
                    'paths' => array(
                        '/path/to/Entities',
                        '/path/to/other/Entities',
                    ),
                ),
            ),
        ),
    );


## Usage

### Accessing the default, pre-configured, entity-manager instance
A default EntityManager instance has been configured for you and is called "doctrine-em". You can access
it from an ActionController using the locator as follows:

    $em = $this->getLocator()->get('doctrine-em');
    
If for some reason you want access to additional objects such as the EventManager, Cache, or Connection instances
you can get them from the locator the same way.

## Available locator items
Following locator items are preconfigured with this module:

  - doctrine-connection', a Doctrine\DBAL\Connection instance
  - doctrine-configuration, a SpiffyDoctrine\ORM\Configuration instance
  - doctrine-metadatacache, a Doctrine\Common\Cache\ArrayCache instance
  - doctrine-querycache, a Doctrine\Common\Cache\ArrayCache instance
  - doctrine-resultcache, a Doctrine\Common\Cache\ArrayCache instance
  - doctrine-eventmanager, a Doctrine\Common\EventManager instance
  - doctrine-driverchain, a Doctrine\ORM\Mapping\Driver\DriverChain instance
  - doctrine-annotationdriver, a Doctrine\ORM\Mapping\Driver\AnnotationDriver instance
  - doctrine-phpdriver, a Doctrine\ORM\Mapping\Driver\PHPDriver instance
  - doctrine-staticphpdriver, a Doctrine\ORM\Mapping\Driver\StaticPHPDriver instance 
  - doctrine-xmldriver, a Doctrine\ORM\Mapping\Driver\XmlDriver instance
  - doctrine-yamldriver, a Doctrine\ORM\Mapping\Driver\YamlDriver instance
  - doctrine-cachedreader, a Doctrine\Common\Annotations\CachedReader instance
  - doctrine-annotationcache, a Doctrine\Common\Cache\ArrayCache instance
  - doctrine-indexedreader, a Doctrine\Common\Annotations\IndexedReader instance
  - doctrine-annotationreader, a Doctrine\Common\Annotations\AnnotationReader instance

## Doctrine CLI
The Doctrine CLI has been pre-configured and is available in SpiffyDoctrine\bin. It should work as
is without any special configuration required.

## EntityExists and NoEntityExists Validators
The EntityExists and NoEntityExists are validators similar to Zend\Validator\Db validators. You can 
pass a variety of options to determine validity. The most basic use case requires an entity manager (em),
an entity, and a field. You also have the option of specifying a query_builder Closure to use if you
want to fine tune the results. 

    $validator = new \SpiffyDoctrine\Validator\NoEntityExists(array(
       'em' => $this->getLocator()->get('doctrine-em'),
       'entity' => 'SpiffyUser\Entity\User',
       'field' => 'username',
       'query_builder' => function($er) {
           return $er->createQueryBuilder('q');
       }
    ));
    var_dump($validator->isValid('test'));

## Tuning for production
Tuning the system for production should be as simple as setting the following in your
configuration (example presumes you have APC installed).

    'di' => array(
        'instance' => array(
            'alias' => array(
                'doctrine-metadatacache'      => 'Doctrine\Common\Cache\ApcCache',
                'doctrine-querycache'         => 'Doctrine\Common\Cache\ApcCache',
                'doctrine-resultcache'        => 'Doctrine\Common\Cache\ApcCache',
                'doctrine-annotationcache'    => 'Doctrine\Common\Cache\ApcCache',
            ),
            'doctrine-configuration' => array(
                'parameters' => array(
                    'autoGenerateProxies' => false,
                ),
            ),
        ),
## Authentication adapter for Zend\Authentication
    $adapter = new \SpiffyDoctrine\Authentication\Adapter\DoctrineEntity(
        $this->getLocator()->get('em-default'), // entity manager
        'Application\Test\Entity',
        'username', // optional, default shown
        'password'  // optional, default shown
    );
    $adapter->setIdentity('username');
    $adapter->setCredential('password');
    $result = $adapter->authenticate();
    
    var_dump($result);
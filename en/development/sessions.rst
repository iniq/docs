Sessions
########

CakePHP provides a wrapper and suite of utility features on top of PHP's native
``session`` extension.  Sessions allow you to identify unique users across the
requests and store persistent data for specific users. Unlike Cookies, session
data is not available on the client side.  Usage of the ``$_SESSION`` is generally
avoided in CakePHP, and instead usage of the Session classes is preferred.


Session Configuration
=====================

Session configuration is stored in ``Configure``, and the session classes will
retrive it from there as needed. Session configuration is stored under the top
level ``Session`` key, and a number of options are available:

* ``Session.cookie`` - Change the name of the session cookie.

* ``Session.timeout`` - The number of *minutes* you want sessions to last.

* ``Session.cookieTimeout`` - The number of *minutes* you want sessions to last.
  If this is undefined, the value from ``Session.timeout`` will be used.

* ``Session.checkAgent`` - Should the user agent be checked, on each request.  If
  the useragent does not match the session will be destroyed.

* ``Session.autoRegenerate`` - Auto regeneration used to only be available when
  ``Security.level`` was set to high.  Enabling this setting, turns on automatic
  renewal of sessions, and sessionids that change frequently.

* ``Session.defaults`` - Allows you to use one the built-in default session
  configurations as a base for your session configuration.

* ``Session.handler`` - Allows you to define a custom session handler. The core
  database and cache session handlers use this.  This option replaces
  ``Session.save`` in previous versions. See below for additional information on
  Session handlers.

* ``Session.ini`` - Allows you to set additional session ini settings for your
  config.  This combined with ``Session.handler`` replace the custom session
  handling features of previous versions.

Using built in defaults
=======================

CakePHP comes with several built in session configurations.  You can either use
these as the basis for your session configuration, or you can create a fully
custom solution.  To use defaults, simply set the 'defaults' key to the name of
the default you want to use.  You can then override any sub setting by declaring
it in your Session config::


    <?php
	Configure::write('Session', array(
		'defaults' => 'php'
	));

The above will use the built-in 'php' session configuration.  You could augment
part or all of it by doing the following::


    <?php
    Configure::write('Session', array(
        'defaults' => 'php',
        'cookie' => 'my_app',
        'timeout' => 4320 //3 days
    ));

The above overrides the timeout and cookie name for the 'php' session
configuration.  The built-in configurations are:

* ``php`` - Saves sessions with the standard settings in your php.ini file.
* ``cake`` - Saves sessions as files inside ``app/tmp/sessions``.  This is a
  good option when on hosts that don't allow you to write outside your own home
  dir.
* ``database`` - Use the built in database sessions. See below for more information.
* ``cache`` - Use the built in cache sessions. See below for more information.

Session Handlers
----------------

Session handlers can also be defined in the session config array.  When defined
they allow you to map the various ``session_save_handler`` values to a class or
object you want to use for session saving. There are two ways to use the
'handler'.  The first is to provide an array with 5 callables.  These callables
are then applied to ``session_set_save_handler``::

    <?php
    Configure::write('Session', array(
        'userAgent' => false,
        'cookie' => 'my_cookie',
        'timeout' => 600,
        'handler' => array(
            array('Foo', 'open'),
            array('Foo', 'close'),
            array('Foo', 'read'),
            array('Foo', 'write'),
            array('Foo', 'destroy'),
            array('Foo', 'gc'),
        ),
        'ini' => array(
            'cookie_secure' => 1,
            'use_trans_sid' => 0
        )
    ));

The second mode is to define an 'engine' key.  This key should be a classname
that implements ``CakeSessionHandlerInterface``.  Implementing this interface
will allow CakeSession to automatically map the methods for the handler.  Both
the core Cache and Database session handlers use this method for saving
sessions.  Additional settings for the handler should be placed inside the
handler array.  You can then read those values out from inside your handler.

You can also use session handlers from inside plugins.  By setting the engine to
something like ``MyPlugin.PluginSessionHandler``.  This will load and use the
``PluginSessionHandler`` class from inside the MyPlugin of your application.


CakeSessionHandlerInterface
---------------------------

This interface is used for all custom session handlers inside CakePHP, and can
be used to create custom user land session handlers.  Simply implement the
interface in your class and set ``Session.handler.engine``  to the classname
you've created.  CakePHP will attempt to load the handler from inside
``app/Model/Datasource/Session/$classname.php``.  So if your classname is
``AppSessionHandler`` the file should be
``app/Model/Datasource/Session/AppSessionHandler.php``.

Database sessions
-----------------

The changes in session configuration change how you define database sessions.
Most of the time you will only need to set ``Session.handler.model`` in your
configuration as well as choose the database defaults::


    <?php
    Configure::write('Session', array(
        'defaults' => 'database',
        'handler' => array(
            'model' => 'CustomSession'
        )
    ));

The above will tell CakeSession to use the built in 'database' defaults, and
specify that a model called ``CustomSession`` will be the delegate for saving
session information to the database.

Cache Sessions
--------------

The Cache class can be used to store sessions as well.  This allows you to store
sessions in a cache like APC, memcache, or Xcache.  There are some caveats to
using cache sessions, in that if you exhaust the cache space, sessions will
start to expire as records are evicted.

To use Cache based sessions you can configure you Session config like::

    <?php
    Configure::write('Session', array(
        'defaults' => 'cache',
        'handler' => array(
            'config' => 'session'
        )
    ));

This will configure CakeSession to use the ``CacheSession`` class as the
delegate for saving the sessions.  You can use the 'config' key which cache
configuration to use. The default cache configuration is ``'default'``.

More granular controls
======================


In addition to the new approach on session handlers, there are a few new
controls in 2.0. First up is the array of ini settings.  This array allows you
to modify and handle ini configurations that are not afforded by the built in
shortcuts.  For example you could use it to control settings like
``session.gc_divisor``::

    <?php
    Configure::write('Session', array(
        'defaults' => 'php',
        'ini' => array(
            'session.gc_divisor' => 1000,
            'session.cookie_httponly' => true
        )
    ));


Next up is ``Session.autoRegenerate``.  This configuration value was
automatically enabled in previous versions by setting ``Security.level`` to
high. In 2.0 its an independent setting, which is disabled by default.  Enabling
it will use the session's ``Config.countdown`` value to keep track of requests.
Once the countdown reaches 0, the session id will be regenerated.  This is a
good option to use for applications that need the additional security frequently
changing session ids provide.  To use this feature set
``Session.autoRegenerate`` to true.  You can control the number of requests
needed to regenerate the session by modifying
``CakeSession::$requestCountdown``.

Accessing Session data
======================

..todo::

    Complete this.


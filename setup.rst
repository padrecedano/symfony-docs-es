Instalación y configuración de Symfony Framework
=============================================

.. admonition:: Screencast
	:class: screencast

¿Prefieres los videotutoriales? Echa un vistazo a `Desarrollo armonioso con Symfony`_
serie de screencasts.

.. _symfony-tech-requirements:

Requerimientos técnicos
-------------------------------------

Antes de crear tu primera aplicación Symfony debes:

* Instalar PHP 8.1 o superior y estas extensiones de PHP (que están instaladas y
   habilitadas por defecto en la mayoría de las instalaciones de PHP 8): `Ctype`_, `iconv`_,
   `PCRE`_, `Session`_, `SimpleXML`_ y `Tokenizer`_;
* `Install Composer`_, que se utiliza para instalar paquetes PHP.

Opcionalmente, también puedes instalar `Symfony CLI`_. Esto crea un binario llamado
``Symfony`` que proporciona todas las herramientas que necesitas para desarrollar y ejecutar tu
Aplicación Symfony localmente.

El binario ``Symfony`` también proporciona una herramienta para verificar si tu computadora cumple con todos
los requisitos. Abre la terminal de tu consola y ejecuta este comando:

.. code-block:: terminal

	$ symfony check:requirements

.. note::

La CLI de Symfony está escrita en Go y puedes contribuir a ella en el
`repositorio symfony-cli/symfony-cli GitHub`_.

.. _creating-symfony-applications:

Creación de aplicaciones Symfony
--------------------------------------------

Abre la terminal de tu consola y ejecuta cualquiera de estos comandos para crear una nueva solicitud de Symfony:

.. code-block:: terminal

    # ejecuta esto si estás creando una aplicación web tradicional
    $ symfony new my_project_directory --version="6.2.*" --webapp

    # ejecuta esto si estás creando un microservicio, una aplicación de consola o una API
    $ symfony new my_project_directory --version="6.2.*"

La única diferencia entre estos dos comandos es el número de paquetes instalados por defecto. La opción ``--webapp`` instala 
todos los paquetes que por lo general son necesarios crear aplicaciones web, por lo que el tamaño de la instalación 
será mayor.

Si no estás usando el binario de Symfony, ejecuta estos comandos para crear la nueva
Aplicación Symfony usando Composer:

.. code-block:: terminal

    # ejecuta esto si estás creando una aplicación web tradicional
    $ composer create-project symfony/skeleton:"6.2.*" my_project_directory
    $ cd my_project_directory
    $ composer require webapp

    # ejecuta esto si estás creando un microservicio, una aplicación de consola o una API
    $ composer create-project symfony/skeleton:"6.2.*" my_project_directory

No importa qué comando ejecutes para crear la aplicación Symfony. Todos ellos
crearán un nuevo directorio ``my_project_directory/``, descargarán algunas dependencias
en él e incluso generarán los directorios y archivos básicos que necesitarás para 
comenzar. En otras palabras, ¡tu nueva aplicación está lista!


.. note::

El directorio de caché y registros del proyecto (por defecto, ``<proyecto>/var/cache/``
y ``<proyecto>/var/log/``) debe ser escribible por el servidor web. Si tienes
cualquier problema, lee `cómo configurar permisos para aplicaciones Symfony`_.

.. _install-existing-app:


Configuración de un proyecto Symfony existente
---------------------------------------------

Además de crear nuevos proyectos Symfony, también trabajarás en proyectos
ya creados por otros desarrolladores. En ese caso, solo necesitas obtener el
código del proyecto e instalar las dependencias con Composer. Asumiendo que tu equipo usa
Git, configura tu proyecto con los siguientes comandos:

.. code-block:: terminal

	# clona el proyecto para descargar su contenido
	$ cd projects/
	$ git clone ...

	# hace que Composer instale las dependencias del proyecto en vendor/
	$ cd my-project/
	$ composer install
	
Probablemente también necesites personalizar tu archivo `archivo .env`_
y hacer algunas otras tareas específicas del proyecto (por ejemplo, crear una base de datos). Cuando
vayas a trabajar por primera vez  en una aplicación Symfony existente, puede ser útil
ejecutar este comando que muestra información sobre el proyecto:

.. code-block:: terminal

	$ php bin/console about
	
	
Ejecutar aplicaciones de Symfony
----------------------------

En producción, debes instalar un servidor web como Nginx o Apache y `configurarlo para ejecutar Symfony`_. Este método también se puede usar si no estás usando el servidor web local de Symfony para el desarrollo.

Sin embargo, para el desarrollo local, la forma más conveniente de ejecutar Symfony es usar el `servidor web local`_ provisto por el binario de ``symfony``. Este servidor local proporciona, entre otras cosas, soporte para HTTP/2, solicitudes concurrentes, TLS/SSL y generación automática de certificados de seguridad.

Abre tu terminal de consola, muévete a tu nuevo directorio de proyectos e inicia el servidor web local de la siguiente manera:

.. code-block:: terminal

    $ cd my-project/
    $ symfony server:start
    
Abre tu navegador y navega hasta ``http://localhost:8000/``. Si todo funciona, verás una página de bienvenida. Más tarde, cuando hayas terminado de trabajar, detén el servidor presionando ``Ctrl+C`` desde tu terminal.

.. tip::

El servidor web funciona con cualquier aplicación PHP, no solo con proyectos Symfony, por lo que es una herramienta de desarrollo genérica muy útil.

Integración con Symfony Docker
~~~~~~~~~~~~~~~~~~~~~~~~~~

Si deseas utilizar Docker con Symfony, consulta `Uso de Docker con Symfony`_.

.. _symfony-flex:

Instalación de paquetes
-------------------

Una práctica común al desarrollar aplicaciones Symfony es instalar paquetes (Symfony los llama `bundles`) que brindan funciones listas para usar. Los paquetes generalmente requieren alguna configuración antes de usarlos (editar algún archivo para habilitar el paquete, crear algún archivo para agregar alguna configuración inicial, etc.)

La mayoría de las veces esta configuración se puede automatizar y es por eso que Symfony incluye `Symfony Flex`_ , una herramienta para simplificar la instalación/eliminación de paquetes en las aplicaciones de Symfony. Técnicamente hablando, Symfony Flex es un complemento de Composer que se instala por defecto al crear una nueva aplicación Symfony y que automatiza las tareas más comunes de las aplicaciones Symfony.

.. tip::

También puedes `agregar Symfony Flex a un proyecto existente`_.

Symfony Flex modifica el comportamiento de los comandos `require`, `update` y `remove` de Composer para proporcionar funciones avanzadas. Considera el siguiente ejemplo:

.. code-block:: terminal

    $ cd my-project/
    $ composer require logger
    
Si ejecutas ese comando en una aplicación Symfony que no usa Flex, verás un error de Composer que explica que ese ``logger`` no es un nombre de paquete válido. Sin embargo, si la aplicación tiene instalado Symfony Flex, ese comando instala y habilita todos los paquetes necesarios para usar el ``logger`` oficial de Symfony.

Esto es posible porque muchos paquetes/bundles de Symfony definen "recetas" , que son un conjunto de instrucciones automatizadas para instalar y habilitar paquetes en las aplicaciones de Symfony. Flex realiza un seguimiento de las recetas que instaló en un archivo ``symfony.lock``, que debe confirmarse en tu repositorio de código.

Las recetas de Symfony Flex son aportadas por la comunidad y se almacenan en dos repositorios públicos: 

* `Repositorio principal de recetas`_, es una lista seleccionada de recetas para paquetes mantenidos y de alta calidad. Symfony Flex solo busca en este repositorio de forma predeterminada.

* `Repositorio de recetas Contrib`_, contiene todas las recetas creadas por la comunidad. Todas ellas están garantizados para funcionar, pero sus paquetes asociados podrían no recibir mantenimiento. Symfony Flex te pedirá permiso antes de instalar cualquiera de estas recetas.

Lee la `documentación sobre Recetas de Symfony`_ para aprender todo sobre cómo crear recetas para tus propios paquetes.

.. _symfony-packs:

Paquetes Symfony
~~~~~~~~~~~~~

A veces, una sola característica requiere la instalación de varios paquetes y bundles. En lugar de instalarlos individualmente, Symfony proporciona **paquetes**, que son metapaquetes de Composer que incluyen varias dependencias.

Por ejemplo, para agregar funciones de depuración en tu aplicación, puedes ejecutar el comando ``composer require --dev debug``. Esto instala el ``symfony/debug-pack``, que a su vez instala varios paquetes como ``symfony/debug-bundle``, ``symfony/monolog-bundle``, ``symfony/var-dumper``, etc.

No verás la dependencia ``symfony/debug-pack`` en tu ``composer.json``, ya que Flex descomprime automáticamente el paquete. Esto significa que solo agrega los paquetes reales como dependencias (por ejemplo, verás un nuevo ``symfony/var-dumper`` en ``require-dev``). Si bien no se recomienda, puedes usar la opción ``composer require --no-unpack ... `` para deshabilitar el desempaquetado.


Comprobación de vulnerabilidades de seguridad
---------------------------------

El binario ``symfony`` creado cuando instalas `Symfony CLI`_ proporciona un comando para verificar si las dependencias de tu proyecto contienen alguna vulnerabilidad de seguridad conocida:

.. code-block:: terminal

    $ symfony check:security
    
Una buena práctica de seguridad es ejecutar este comando regularmente para poder actualizar o reemplazar las dependencias comprometidas lo antes posible. La verificación de seguridad se realiza localmente al obtener `la base de datos pública de avisos de seguridad de PHP`_, por lo que tu archivo ``composer.lock`` no se envía a la red.

El comando ``check:security`` termina con un código de salida distinto de cero si alguna de tus dependencias se ve afectada por una vulnerabilidad de seguridad conocida. De esta manera, puedes agregarlo al proceso de creación de tu proyecto y a sus flujos de trabajo de integración continua para que fallen cuando haya vulnerabilidades.

.. tip::

En los servicios de integración continua, puedes verificar las vulnerabilidades de seguridad utilizando un proyecto independiente diferente llamado `Local PHP Security Checker`_. Este es el mismo proyecto utilizado internamente por ``check:security`` pero de un tamaño mucho más pequeño que toda la CLI de Symfony.

Versiones de Symfony LTS
--------------------

De acuerdo con el `proceso de lanzamiento de Symfony`_, las versiones de "soporte a largo plazo" (o LTS para abreviar) se publican cada dos años. Consulta los `Lanzamientos de Symfony`_ para saber cuál es la última versión de LTS.

De forma predeterminada, el comando que crea nuevas aplicaciones Symfony utiliza la última versión estable. Si deseas utilizar una versión LTS, agregua la opción ``--version``:

.. code-block:: terminal

    # usar la version LTS más reciente
    $ symfony new my_project_directory --version=lts

    # usar la 'próxima' versión de Symfony que se lanzará (todavía en desarrollo)
    $ symfony new my_project_directory --version=next

    # también puedes seleccionar una versión específica exacta de Symfony
    $ symfony new my_project_directory --version="5.4.*"

Los accesos directos ``lts`` y ``next`` solo están disponibles cuando se usa Symfony para crear nuevos proyectos. Si usas Composer, debes indicar la versión exacta:

.. code-block:: terminal

    $ composer create-project symfony/skeleton:"5.4.*" my_project_directory


La aplicación de demostración de Symfony
----------------------------

La `Aplicación Demo de Symfony`_ es una aplicación totalmente funcional que muestra la forma recomendada de desarrollar aplicaciones Symfony. Es una gran herramienta de aprendizaje para los recién llegados a Symfony y su código contiene toneladas de comentarios y notas útiles.

Ejecuta este comando para crear un nuevo proyecto basado en la aplicación de demostración de Symfony:

.. code-block:: terminal

    $ symfony new my_project_directory --demo


.. _`Desarrollo armonioso con Symfony`: https://symfonycasts.com/screencast/symfony
.. _`Instalar Composer`: https://getcomposer.org/download/
.. _`Symfony CLI`: https://symfony.com/download
.. _`symfony-cli/symfony-cli GitHub repository`: https://github.com/symfony-cli/symfony-cli
.. _`cómo configurar permisos para aplicaciones Symfony`: https://symfony.com/doc/current/configuration.html#config-dot-env
.. _`configurarlo para ejecutar Symfony`: https://symfony.com/doc/current/setup/web_server_configuration.html
.. _`servidor web local`: https://symfony.com/doc/current/setup/symfony_server.html
.. _`Uso de Docker con Symfony`: https://symfony.com/doc/current/setup/docker.html
.. _`bundles`: https://symfony.com/doc/current/bundles.html
.. _`Symfony Flex`: https://github.com/symfony/flex
.. _`agregar Symfony Flex a un proyecto existente`: https://symfony.com/doc/current/setup/flex.html
.. _`la base de datos pública de avisos de seguridad de PHP`: https://github.com/FriendsOfPHP/security-advisories
.. _`Local PHP Security Checker`: https://github.com/fabpot/local-php-security-checker
.. _`proceso de lanzamiento de Symfony`: https://symfony.com/doc/current/contributing/community/releases.html
.. _`archivo .env`: https://symfony.com/doc/current/configuration.html#config-dot-env
.. _`Aplicación Demo de Symfony`: https://github.com/symfony/demo
.. _`Symfony Flex`: https://github.com/symfony/flex
.. _`Base de datos de avisos de seguridad de PHP`: https://github.com/FriendsOfPHP/security-advisories
.. _`Comprobador de seguridad de PHP local`: https://github.com/fabpot/local-php-security-checker
.. _`Lanzamientos de Symfony`: https://symfony.com/releases
.. _`Repositorio principal de recetas`: https://github.com/symfony/recipes
.. _`Repositorio de recetas Contrib`: https://github.com/symfony/recipes-contrib
.. _`documentación sobre Recetas de Symfony`: https://github.com/symfony/recipes/blob/master/README.rst
.. _`iconv`: https://www.php.net/book.iconv
.. _`Session`: https://www.php.net/book.session
.. _`Ctype`: https://www.php.net/book.ctype
.. _`Tokenizer`: https://www.php.net/book.tokenizer
.. _`SimpleXML`: https://www.php.net/book.simplexml
.. _`PCRE`: https://www.php.net/book.pcre




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

Opcionalmente, también puedes `instalar Symfony CLI`_. Esto crea un binario llamado
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
cualquier problema, lee :doc:`cómo configurar permisos para aplicaciones Symfony </setup/file_permissions>`.

.. _install-existing-app:


Configuración de un proyecto Symfony existente
---------------------------------------------

Además de crear nuevos proyectos Symfony, también trabajarás en proyectos
ya creados por otros desarrolladores. En ese caso, solo necesitas obtener el
código del proyecto e instalar las dependencias con Composer. Asumiendo que tu equipo usa
Git, configura tu proyecto con los siguientes comandos:

.. code-block:: terminal

	# clone the project to download its contents
	$ cd projects/
	$ git clone ...

	# make Composer install the project's dependencies into vendor/
	$ cd my-project/
	$ composer install
	
Probablemente también necesites personalizar tu archivo :ref:`.env <config-dot-env>`
y hacer algunas otras tareas específicas del proyecto (por ejemplo, crear una base de datos). Cuando
vayas a trabajar por primera vez  en una aplicación Symfony existente, puede ser útil
ejecutar este comando que muestra información sobre el proyecto:

.. code-block:: terminal

	$ php bin/console about

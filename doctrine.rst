Bases de datos y Doctrine ORM
==============================

Symfony proporciona todas las herramientas que necesitas para usar bases de datos en tus aplicaciones gracias a `Doctrine`_, el mejor conjunto de bibliotecas PHP para trabajar con bases de datos. Estas herramientas admiten bases de datos relacionales como MySQL y PostgreSQL y también bases de datos NoSQL como MongoDB.

Las bases de datos son un tema amplio, por lo que la documentación se divide en tres artículos:

* Este artículo explica la forma recomendada de trabajar con bases de datos relacionales en aplicaciones Symfony;
* Lee el artículo `Cómo usar Doctrine DBAL`_ si necesitas acceso de **bajo nivel** para realizar consultas SQL sin formato a **bases de datos relacionales** (similar al PDO de PHP );
* Lee los documentos de `DoctrineMongoDBBundle`_ si estás trabajando con **bases de datos MongoDB**.

Instalando Doctrine
-------------------

Primero, instala el soporte de Doctrine a través del ``orm`` `Symfony pack`_, así como del MakerBundle, que te ayudará a generar algo de código:

	.. code-block:: terminal

		$ composer require symfony/orm-pack
		$ composer require --dev symfony/maker-bundle


Configuración de la base de datos
~~~~~~~~~~~~~~~~~~~~~~~~

La información de conexión de la base de datos se almacena como una variable de entorno llamada ``DATABASE_URL``. Para el desarrollo, puedes encontrar y personalizar esto dentro de ``.env``:

.. code-block:: text

	# .env (o sobre-escribe DATABASE_URL en .env.local para evitar confirmar cambios)

	# personaliza esta línea!
	DATABASE_URL="mysql://db_user:db_password@127.0.0.1:3306/db_name?serverVersion=5.7"

	# para usar mariadb:
	DATABASE_URL="mysql://db_user:db_password@127.0.0.1:3306/db_name?serverVersion=mariadb-10.5.8"

	# para usar sqlite:
	# DATABASE_URL="sqlite:///%kernel.project_dir%/var/app.db"

	# para usar postgresql:
	# DATABASE_URL="postgresql://db_user:db_password@127.0.0.1:5432/db_name?serverVersion=11&charset=utf8"

	# para usar oracle:
	# DATABASE_URL="oci8://db_user:db_password@127.0.0.1:1521/db_name"

.. caution::

	Si el nombre de usuario, la contraseña, el host o la base de datos contienen algún carácter considerado especial en una URI (tales como ``+``, ``@``, ``$``, ``#``, ``/``, ``:``, ``*``, ``!``, ``%``), debes codificarlos. Consulta `RFC 3986`_ para obtener la lista completa de caracteres reservados o usa la función `urlencode`_ para codificarlos. En este caso, debes eliminar el prefijo `resolve:` en `config/packages/doctrine.yaml` para evitar errores: ``url: '%env(DATABASE_URL)%'``

Ahora que tus parámetros de conexión están configurados, Doctrine puede crear la base de datos `db_name` por ti:

.. code-block:: terminal

	$ php bin/console doctrine:database:create

Hay más opciones `config/packages/doctrine.yaml` que puedes configurar, incluida tu `server_version` (por ejemplo, 5.7 si estás usando MySQL 5.7), lo que puede afectar el funcionamiento de Doctrine.

.. tip::

	Hay muchos otros comandos de Doctrine. Ejecuta `php bin/console list doctrine` para ver una lista completa.


Crear una clase de entidad
------------------------

Supongamos que estás creando una aplicación en la que se deben mostrar los productos. Sin siquiera pensar en Doctrine o bases de datos, ya sabes que necesitas un objeto `Product` para representar esos productos.

Puedes usar el comando ``make:entity`` para crear esta clase y cualquier campo que necesites. El comando te hará algunas preguntas; respóndelas como se hace a continuación:

.. code-block:: bash

	$ php bin/console make:entity

	Class name of the entity to create or update:
	> Product

	New property name (press <return> to stop adding fields):
	> name

	Field type (enter ? to see all types) [string]:
	> string

	Field length [255]:
	> 255

	Can this field be null in the database (nullable) (yes/no) [no]:
	> no

	New property name (press <return> to stop adding fields):
	> price

	Field type (enter ? to see all types) [string]:
	> integer

	Can this field be null in the database (nullable) (yes/no) [no]:
	> no

	New property name (press <return> to stop adding fields):
	>
	(press enter again to finish)


¡Guau! Ahora tienes un nuevo archivo ``src/Entity/Product.php``::

	// src/Entity/Product.php
	namespace App\Entity;

	use App\Repository\ProductRepository;
	use Doctrine\ORM\Mapping as ORM;

	#[ORM\Entity(repositoryClass: ProductRepository::class)]
	class Product
	{
		#[ORM\Id]
		#[ORM\GeneratedValue]
		#[ORM\Column]
		private ?int $id = null;

		#[ORM\Column(length: 255)]
		private ?string $name = null;

		#[ORM\Column]
		private ?int $price = null;

		public function getId(): ?int
		{
			return $this->id;
		}

		// ... getter and setter methods
	}


.. note::
	
    A partir de la versión v1.44.0: MakerBundle solo admite entidades que usan atributos de PHP.

.. note::

	¿Confundido porque el precio es un número entero? No te preocupes: esto es solo un ejemplo. Pero almacenar precios como números enteros (por ejemplo, 100 = $1 USD) puede evitar problemas de redondeo.

.. note::

	Si estás utilizando una base de datos SQLite, verás el siguiente error: *PDOException: SQLSTATE[HY000]: General error: 1 Cannot add a NOT NULL column with default value NULL*. Agrega una opción ``nullable=true`` en la ``description`` para solucionar el problema.

Hay un `límite de 767 bytes para el prefijo de clave de índice`_ cuando se usan tablas InnoDB en MySQL 5.6 y versiones anteriores. Las columnas de cadenas con 255 caracteres de longitud y codificación `utf8mb4` superan ese límite. Esto significa que cualquier columna de tipo `string` y `unique=true` debe establecer su tamaño máximo en 190. De lo contrario, verás este error: *"[PDOException] SQLSTATE[42000]: Syntax error or access violation:
1071 Specified key was too long; max key length is 767 bytes"*. | "[PDOException] SQLSTATE[42000]: Error de sintaxis o infracción de acceso: 1071 La clave especificada era demasiado larga; la longitud máxima de la clave es de 767 bytes" .
Esta clase se llama una "entidad". Y pronto, podrá guardar y consultar objetos del tipo `Product` en una tabla `product` de tu base de datos. Cada propiedad en la entidad `Product` se puede asignar a una columna en esa tabla. Esto generalmente se hace con atributos: los comentarios ``#[ORM\Column(...)]`` que ves arriba de cada propiedad:

	.. image:: /_images/doctrine/mapping_single_entity.png
    	   :align: center

El comando ``make:entity`` es una herramienta para hacer la vida más fácil. Pero este es *su* código: agregar/eliminar campos, agregar/eliminar métodos o actualizar la configuración.

Doctrine admite una amplia variedad de tipos de campos, cada uno con sus propias opciones. Para ver una lista completa, consulta la documentación de `Tipos de Mapeo de Doctrine`_. Si deseas utilizar XML en lugar de anotaciones, agrega ``type: xml`` y ``dir: '%kernel.project_dir%/config/doctrine'`` a las asignaciones de entidades en tu archvo ``config/packages/doctrine.yaml``.

Ten cuidado de no utilizar palabras clave SQL reservadas como nombres de tabla o columna (por ejemplo ``GROUP`` o ``USER``). Consulta la documentación de Doctrine sobre `Palabras Reservadas en SQL`_ para obtener detalles sobre cómo evitarlas. O cambia el nombre de la tabla con la anotación ``#[ORM\Table(name: 'groups')]`` encima de la clase o configura el nombre de la columna con la opción ``name: 'group_name'``.

Migraciones: creación de tablas/esquema de la base de datos
-----------------------------------------------

La clase ``Product`` está completamente configurada y lista para guardarse en una tabla ``product``. Si acabas de definir esta clase, su base de datos aún no tiene la tabla ``product``. Para agregarla, puedes aprovechar `DoctrineMigrationsBundle`_, que ya está instalado:

.. code-block:: terminal

	$ php bin/console make:migration

Si todo funcionó, deberías ver algo como esto:

.. code-block:: text

	SUCCESS!

	Next: Review the new migration "migrations/Version20211116204726.php"
	Then: Run the migration with php bin/console doctrine:migrations:migrate

Si abres este archivo, podrás ver que contiene el SQL necesario para actualizar tu base de datos. Para ejecutar ese SQL, ejecuta tus migraciones:

.. code-block:: terminal

	$ php bin/console doctrine:migrations:migrate

Este comando ejecuta todos los archivos de migración que aún no se han ejecutado en tu base de datos. Debes ejecutar este comando en producción cuando hagas el displiegue, para mantener tu base de datos de producción actualizada.

.. _doctrine-add-more-fields:

Migraciones y agregar más campos
-------------------------------

Pero, ¿qué sucede si necesitas agregar una nueva propiedad de campo a ``Product``, como un description? Puedes editar la clase para agregar la nueva propiedad. Pero, también puedes usar ``make:entity`` de nuevo:

.. code-block:: bash

	$ php bin/console make:entity

	Class name of the entity to create or update
	> Product

	New property name (press <return> to stop adding fields):
	> description

	Field type (enter ? to see all types) [string]:
	> text

	Can this field be null in the database (nullable) (yes/no) [no]:
	> no

	New property name (press <return> to stop adding fields):
	>
	(press enter again to finish)


Esto agrega la nueva propiedad ``description`` así como los métodos ``getDescription()`` y ``setDescription()``:

.. code-block:: diff

  	// src/Entity/Product.php
  	// ...
	+  use Doctrine\DBAL\Types\Types;

  	class Product
  	{
	  	// ...

	+     #[ORM\Column(type: Types::TEXT)]
	+     private $description;

	  	// getDescription() & setDescription() were also added
  	}

La nueva propiedad está asignada, pero aún no existe en la tabla ``product``. ¡Ningún problema! Puedes generar una nueva migración:

.. code-block:: terminal

	$ php bin/console make:migration

Esta vez, el SQL en el archivo generado se verá así:

.. code-block:: sql

	ALTER TABLE product ADD description LONGTEXT NOT NULL

El sistema de migración es inteligente. ¡Compara todas tus entidades con el estado actual de la base de datos y genera el SQL necesario para sincronizarlas! Como antes, ejecuta tus migraciones:

.. code-block:: terminal

	$ php bin/console doctrine:migrations:migrate

Esto solo ejecutará el nuevo archivo de migración, porque DoctrineMigrationsBundle sabe que la primera migración ya se ejecutó antes. Detrás de escena, administra una tabla ``migration_versions`` para rastrear esto.

Cada vez que realices un cambio en su esquema, ejecuta estos dos comandos para generar la migración y luego ejecútela. Asegúrese de confirmar los archivos de migración y ejecutarlos cuando despliegues la aplicación.

.. _doctrine-generating-getters-and-setters:

.. tip::

	Si prefieres agregar nuevas propiedades manualmente, el comando ``make:entity`` puede generar los métodos getter & setter por ti:

	.. code-block:: terminal

			$ php bin/console make:entity --regenerate

Si realizas algunos cambios y deseas regenerar todos los métodos getter/setter, considera pasar también ``--overwrite`` en el comando.

Objetos persistentes en la base de datos
----------------------------------

¡Es hora de guardar un objeto ``Product`` en la base de datos! Vamos a crear un nuevo controlador para experimentar:

.. code-block:: terminal

	$ php bin/console make:controller ProductController

Dentro del controlador, puedes crear un nuevo objeto ``Product``, establecer datos en él y guardarlo::

	// src/Controller/ProductController.php
	namespace App\Controller;

	// ...
	use App\Entity\Product;
	use Doctrine\ORM\EntityManagerInterface;
	use Symfony\Component\HttpFoundation\Response;
	use Symfony\Component\Routing\Annotation\Route;

	class ProductController extends AbstractController
	{
		#[Route('/product', name: 'create_product')]
		public function createProduct(EntityManagerInterface $entityManager): Response
		{
			$product = new Product();
			$product->setName('Keyboard');
			$product->setPrice(1999);
			$product->setDescription('Ergonomic and stylish!');

			// tell Doctrine you want to (eventually) save the Product (no queries yet)
			$entityManager->persist($product);

			// actually executes the queries (i.e. the INSERT query)
			$entityManager->flush();

			return new Response('Saved new product with id '.$product->getId());
		}
	}


¡Pruébalo!

	http://localhost:8000/product

¡Felicidades! Acabas de crear tu primera fila en la tabla ``product``. Para probarlo, puedes consultar la base de datos directamente:

.. code-block:: terminal

	$ php bin/console dbal:run-sql 'SELECT * FROM product'

	# En los sistemas Windows que no usan Powershell, debes ejecutar este comando:
	# php bin/console dbal:run-sql "SELECT * FROM product"

Fíjate en el ejemplo anterior con más detalle:

.. _doctrine-entity-manager:

* **línea 13** El argumento ``EntityManagerInterface $entityManager`` le dice a Symfony que inyecte el `servicio Entity Manager`_ en el método del controlador. Este objeto es responsable de guardar objetos y recuperar objetos de la base de datos.

* **línea 17-20** En esta sección, creará una instancia y trabajará con el objeto ``$product`` como cualquier otro objeto PHP normal.

* **línea 23** La llamada ``persist($product)`` le dice a Doctrine que "administre" el objeto ``$product``. Esto **no** provoca que se realice una consulta a la base de datos.

* **línea 26** Cuando el método ``flush()`` es llamado, Doctrine revisa todos los objetos que está administrando para ver si necesitan persistir en la base de datos. En este ejemplo, los datos del objeto ``$product`` no existen en la base de datos, por lo que el administrador de la entidad ejecuta una consulta ``INSERT`` y crea una nueva fila en la tabla ``product``.

.. note::

	Si la llamada a ``flush()`` falla, una ``Doctrine\ORM\ORMException`` es lanzada. Ver `Transacciones y Concurrencia`_.

Ya sea que estés creando o actualizando objetos, el flujo de trabajo es siempre el mismo: Doctrine es lo suficientemente inteligente como para saber si debe hacer un ``INSERT`` o un ``UPDATE`` de su entidad.

.. _automatic_object_validation:

Validación de objetos
------------------

El `Validador de Symfony`_ reutiliza los metadatos de Doctrine para realizar algunas tareas básicas de validación::

	// src/Controller/ProductController.php
	namespace App\Controller;

	use App\Entity\Product;
	use Symfony\Component\HttpFoundation\Response;
	use Symfony\Component\Routing\Annotation\Route;
	use Symfony\Component\Validator\Validator\ValidatorInterface;
	// ...

	class ProductController extends AbstractController
	{
		#[Route('/product', name: 'create_product')]
		public function createProduct(ValidatorInterface $validator): Response
		{
			$product = new Product();
			// This will trigger an error: the column isn't nullable in the database
			$product->setName(null);
			// This will trigger a type mismatch error: an integer is expected
			$product->setPrice('1999');

			// ...

			$errors = $validator->validate($product);
			if (count($errors) > 0) {
				return new Response((string) $errors, 400);
			}

			// ...
		}
	}


Aunque la entidad ``Product`` no define ninguna `configuración de validación`_ explícita, Symfony analiza la configuración del mapeo de Doctrine para inferir algunas reglas de validación. Por ejemplo, dado que la propiedad ``name`` no puede ser ``null`` en la base de datos, se agrega automáticamente una restricción `NotNull`_ a la propiedad (si aún no contiene esa restricción).

La siguiente tabla resume el mapeo entre los metadatos de Doctrine y las restricciones de validación correspondientes agregadas automáticamente por Symfony:

	==================  =========================================================  =====
	Doctrine attribute  Validation constraint                                      Notes
	==================  =========================================================  =====
	``nullable=false``  `NotNull`_                                                 Requiere instalación del `componente PropertyInfo`_
	``type``            `Type`_                                                    Requiere instalación del `componente PropertyInfo`_
	``unique=true``     `UniqueEntity`_
	``length``          `Length`_
	==================  =========================================================  =====

Debido a que el `componente de Formulario`_ y la `Plataforma de API`_ utilizan internamente el componente de Validación, todos tus formularios y API web también se beneficiarán automáticamente de estas restricciones de validación automática.

Esta validación automática es una buena característica para mejorar tu productividad, pero no reemplaza la configuración de validación por completo. Todavía necesitas agregar algunas `restricciones de validación`_ para garantizar que los datos proporcionados por el usuario sean correctos.

Obtener objetos de la base de datos
----------------------------------

Recuperar un objeto de la base de datos es aún más fácil. Supongamos que deseas poder ir ``/product/1`` para ver tu nuevo producto::

	// src/Controller/ProductController.php
	namespace App\Controller;

	use App\Entity\Product;
	use Doctrine\ORM\EntityManagerInterface;
	use Symfony\Component\HttpFoundation\Response;
	use Symfony\Component\Routing\Annotation\Route;
	// ...

	class ProductController extends AbstractController
	{
		#[Route('/product/{id}', name: 'product_show')]
		public function show(EntityManagerInterface $entityManager, int $id): Response
		{
			$product = $entityManager->getRepository(Product::class)->find($id);

			if (!$product) {
				throw $this->createNotFoundException(
					'No product found for id '.$id
				);
			}

			return new Response('Check out this great product: '.$product->getName());

			// or render a template
			// in the template, print things with {{ product.name }}
			// return $this->render('product/show.html.twig', ['product' => $product]);
		}
	}

Otra posibilidad es usar el ``ProductRepository`` de Symfony e inyectarlo mediante el contenedor de inyección de dependencia::

	// src/Controller/ProductController.php
	namespace App\Controller;

	use App\Entity\Product;
	use App\Repository\ProductRepository;
	use Symfony\Component\HttpFoundation\Response;
	use Symfony\Component\Routing\Annotation\Route;
	// ...

	class ProductController extends AbstractController
	{
		#[Route('/product/{id}', name: 'product_show')]
		public function show(int $id, ProductRepository $productRepository): Response
		{
			$product = $productRepository
				->find($id);

			// ...
		}
	}

¡Pruébalo!

	http://localhost:8000/product/1

Cuando consultas un tipo particular de objeto, siempre usas lo que se conoce como su "repositorio". Puedes pensar en un repositorio como una clase de PHP cuyo único trabajo es ayudarte a obtener entidades de una determinada clase.

Una vez que tienes un objeto de repositorio, tienes muchos métodos auxiliares::

	$repository = $entityManager->getRepository(Product::class);

	// buscar un solo Product por su llave primaria, (por lo general "id")
	$product = $repository->find($id);

	// buscar un solo Product por su nombre
	$product = $repository->findOneBy(['name' => 'Keyboard']);
	// o buscar por nombre y precio
	$product = $repository->findOneBy([
		'name' => 'Keyboard',
		'price' => 1999,
	]);

	// buscar varios objetos Product por nombre, ordenados por precio
	$products = $repository->findBy(
		['name' => 'Keyboard'],
		['price' => 'ASC']
	);

	// buscar todos los objetos Product
	$products = $repository->findAll();

¡También puedes agregar métodos *personalizados* para consultas más complejas! Ampliaremos sobre eso más adelante en la sección `Bases de datos y Doctrine ORM`_.

.. tip::

	Al representar una página HTML, la barra de herramientas de depuración web en la parte inferior de la página mostrará la cantidad de consultas y el tiempo que tomó ejecutarlas:

	.. image:: /_images/doctrine/doctrine_web_debug_toolbar.png
	   	   :align: center
	   	   :class: with-browser

Si el número de consultas a la base de datos es demasiado alto, el ícono se volverá amarillo para indicar que algo puede no ser correcto. Haz clic en el icono para abrir Symfony Profiler y ver las consultas exactas que se ejecutaron. Si no ves la barra de herramientas de depuración web, instala el paquete ``profiler`` de Symfony ejecutando este comando: ``composer require --dev symfony/profiler-pack``.

.. _doctrine-entity-value-resolver:

Obtención automática de objetos (EntityValueResolver)
----------------------------------------------------

.. versionadded:: 6.2

	El Resolutor de Valor de Entidad (Entity Value Resolver) se introdujo en Symfony 6.2.

.. versionadded:: 2.7.1

	El cableado automático del ``EntityValueResolver`` se introdujo en DoctrineBundle 2.7.1.

En muchos casos, puedes usar el ``EntityValueResolver`` para hacer la consulta por ti automáticamente. Puedes simplificar el controlador a::

	// src/Controller/ProductController.php
	namespace App\Controller;

	use App\Entity\Product;
	use App\Repository\ProductRepository;
	use Symfony\Component\HttpFoundation\Response;
	use Symfony\Component\Routing\Annotation\Route;
	// ...

	class ProductController extends AbstractController
	{
		#[Route('/product/{id}')]
		public function show(Product $product): Response
		{
			// use the Product!
			// ...
		}
	}

¡Eso es todo! El paquete usa ``{id}`` desde la ruta para consultar ``Product`` por la columna ``id``. Si no se encuentra, se genera una página 404.

Este comportamiento está habilitado de forma predeterminada en todos tus controladores. Puedes deshabilitarlo dando un valor ``false`` a la opción de configuración ``doctrine.orm.controller_resolver.auto_mapping``.

Cuando está deshabilitado, puedes habilitarlo individualmente en los controladores deseados usando el atributo ``MapEntity``::

	// src/Controller/ProductController.php
	namespace App\Controller;

	use App\Entity\Product;
	use Symfony\Bridge\Doctrine\Attribute\MapEntity;
	use Symfony\Component\HttpFoundation\Response;
	use Symfony\Component\Routing\Annotation\Route;
	// ...

	class ProductController extends AbstractController
	{
		#[Route('/product/{id}')]
		public function show(
			#[MapEntity]
			Product $product
		): Response {
			// use the Product!
			// ...
		}
	}

.. tip::

	Cuando está habilitado globalmente, es posible deshabilitar el comportamiento en un controlador específico, usando el ``MapEntity`` configurado  ``disabled: true``::

		public function show(
			#[CurrentUser]
			#[MapEntity(disabled: true)]
			User $user
		): Response {
			// User is not resolved by the EntityValueResolver
			// ...
		}


Obtener automáticamente
~~~~~~~~~~~~~~~~~~~

Si los comodines de tu ruta coinciden con las propiedades de tu entidad, el resolutor los buscará automáticamente::

	/**
 	* Fetch via primary key because {id} is in the route.
 	*/
	#[Route('/product/{id}')]
	public function showByPk(Post $post): Response
	{
	}

	/**
 	* Perform a findOneBy() where the slug property matches {slug}.
 	*/
	#[Route('/product/{slug}')]
	public function showBySlug(Post $post): Response
	{
	}
La recuperación automática funciona en estas situaciones:

* Si ``{id}`` está en tu ruta, entonces se usará para buscar por clave principal a través del método ``find()``.

* El resolutor intentará realizar una búsqueda ``findOneBy()`` utilizando todos los comodines en tu ruta que en realidad son propiedades en tu entidad (se ignoran las que no son propiedades).

Puedes controlar este comportamiento *agregando* el atributo ``MapEntity`` y usando las `opciones de ``MapEntity```_.

Obtener a través de una expresión
~~~~~~~~~~~~~~~~~~~~~~~

Si la búsqueda automática no funciona, puedes escribir una expresión usando el `componente ExpressionLanguage`_::

	#[Route('/product/{product_id}')]
	public function show(
		#[MapEntity(expr: 'repository.find(product_id)')]
		Product $product
	): Response {
	}

En la expresión, la variable ``repository`` será la clase de Repositorio de su entidad y cualquier comodín de ruta, como ``{product_id}`` estarán disponibles como variables.

Esto también se puede usar para ayudar a resolver múltiples argumentos::

	#[Route('/product/{id}/comments/{comment_id}')]
	public function show(
		Product $product,
		#[MapEntity(expr: 'repository.find(comment_id)')]
		Comment $comment
	): Response {
	}

En el ejemplo anterior, el argumento ``$product`` se maneja automáticamente, pero ``$comment` se configura con el atributo ya que ambos no pueden seguir la convención predeterminada.

Opciones ``MapEntity``
~~~~~~~~~~~~~~~~~

Hay varias opciones disponibles en la anotación ``MapEntity``  para controlar el comportamiento:

``id``
	Si una opción ``id`` está configurada y coincide con un parámetro de ruta, la resolución encontrará la clave principal::

		#[Route('/product/{product_id}')]
		public function show(
			#[MapEntity(id: 'product_id')]
			Product $product
		): Response {
		}

``mapping``
	Configura las propiedades y los valores para usar con el método ``findOneBy()``: la clave es el nombre del marcador de posición de la ruta y el valor es el nombre de la propiedad de Doctrine::

		#[Route('/product/{category}/{slug}/comments/{comment_slug}')]
		public function show(
			#[MapEntity(mapping: ['category' => 'category', 'slug' => 'slug'])]
			Product $product,
			#[MapEntity(mapping: ['comment_slug' => 'slug'])]
			Comment $comment
		): Response {
		}

``exclude``
	Configura las propiedades que se deben usar en el método ``findOneBy()`` *excluyendo* una o más propiedades para que no se usen *todas*::

		#[Route('/product/{slug}/{date}')]
		public function show(
			#[MapEntity(exclude: ['date'])]
			Product $product,
			\DateTime $date
		): Response {
		}

``stripNull``
	Si es verdadero, cuando se use ``findOneBy()``, aquellos valores que sean ``null`` no serán tomados en cuenta para la consulta.

``entityManager``
	De forma predeterminada, ``EntityValueResolver`` utiliza el administrador de entidades *predeterminado*, pero puedes configurar esto::

		#[Route('/product/{id}')]
		public function show(
			#[MapEntity(entityManager: ['foo'])]
			Product $product
		): Response {
		}

``evictCache``
	Si es verdadero, obliga a Doctrine a obtener siempre la entidad de la base de datos en lugar de la caché.

``disabled``
	Si es verdadero, ``EntityValueResolver`` no intentará reemplazar el argumento.

Actualización de un objeto
------------------

Una vez que hayas obtenido un objeto de Doctrine, interactúas con él de la misma manera que con cualquier modelo de PHP::

	// src/Controller/ProductController.php
	namespace App\Controller;

	use App\Entity\Product;
	use App\Repository\ProductRepository;
	use Doctrine\ORM\EntityManagerInterface;
	use Symfony\Component\HttpFoundation\Response;
	use Symfony\Component\Routing\Annotation\Route;
	// ...

	class ProductController extends AbstractController
	{
		#[Route('/product/edit/{id}', name: 'product_edit')]
		public function update(EntityManagerInterface $entityManager, int $id): Response
		{
			$product = $entityManager->getRepository(Product::class)->find($id);

			if (!$product) {
				throw $this->createNotFoundException(
					'No product found for id '.$id
				);
			}

			$product->setName('New product name!');
			$entityManager->flush();

			return $this->redirectToRoute('product_show', [
				'id' => $product->getId()
			]);
		}
	}

El uso de Doctrine para editar un producto existente consta de tres pasos:

#. obteniendo el objeto de Doctrine;
#. modificando el objeto;
#. llamando ``flush()`` en el administrador de la entidad.

Tú *puedes* llamar a ``$entityManager->persist($product)``, pero no es necesario: Doctrine ya está "observando" tu objeto en busca de cambios.

Eliminación de un objeto
------------------

Eliminar un objeto es muy similar, pero requiere una llamada al método ``remove()`` del administrador de entidades::

	$entityManager->remove($product);
	$entityManager->flush();

Como era de esperar, el método ``remove()`` notifica a Doctrine que te gustaría eliminar el objeto dado de la base de datos. La consulta ``DELETE`` no se ejecuta realmente hasta que se llama al método ``flush()``.

.. _doctrine-queries:

Consulta de Objetos: el repositorio
------------------------------------

Ya has visto cómo el objeto del repositorio te permite ejecutar consultas básicas sin ningún trabajo::

	// from inside a controller
	$repository = $entityManager->getRepository(Product::class);
	$product = $repository->find($id);

Pero, ¿y si necesitas una consulta más compleja? Cuando generaste tu entidad con el comando ``make:entity``, **también** generaste una clase ``ProductRepository``::

	// src/Repository/ProductRepository.php
	namespace App\Repository;

	use App\Entity\Product;
	use Doctrine\Bundle\DoctrineBundle\Repository\ServiceEntityRepository;
	use Doctrine\Persistence\ManagerRegistry;

	class ProductRepository extends ServiceEntityRepository
	{
		public function __construct(ManagerRegistry $registry)
		{
			parent::__construct($registry, Product::class);
		}
	}

Cuando obtienes tu repositorio (por ejemplo, ``->getRepository(Product::class)``), ¡esto es *actualmente* una instancia de *este* objeto! Esto se debe a la configuración de la clase ``repositoryClass`` que se generó en la parte superior de tu clase de entidad ``Product``.

Supongamos que deseas consultar todos los objetos de ``Product`` mayores que un precio determinado. Agrega un nuevo método para esto a tu repositorio::

	// src/Repository/ProductRepository.php

	// ...
	class ProductRepository extends ServiceEntityRepository
	{
		public function __construct(ManagerRegistry $registry)
		{
			parent::__construct($registry, Product::class);
		}

		/**
	 	* @return Product[]
	 	*/
		public function findAllGreaterThanPrice(int $price): array
		{
			$entityManager = $this->getEntityManager();

			$query = $entityManager->createQuery(
				'SELECT p
				FROM App\Entity\Product p
				WHERE p.price > :price
				ORDER BY p.price ASC'
			)->setParameter('price', $price);

			// returns an array of Product objects
			return $query->getResult();
		}
	}

La cadena pasada a ``createQuery()`` puede parecerse a SQL, pero es `Doctrine Query Language`_. Esto te permite escribir consultas utilizando el lenguaje de consulta comúnmente conocido, pero haciendo referencia a objetos PHP en su lugar (por ejemplo en la declaración ``FROM``).

Ahora, puedes llamar a este método en el repositorio::

	// desde dentro de un controlador
	$minPrice = 1000;

	$products = $entityManager->getRepository(Product::class)->findAllGreaterThanPrice($minPrice);

	// ...

Consulta `Contenedor de Servicios`_ para saber cómo inyectar el repositorio en cualquier servicio.

Consultas con el Generador de Consultas
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine también proporciona un `Generador de Consultas`_  (Query Builder), una forma orientada a objetos de escribir consultas. Se recomienda usarlo cuando las consultas se construyen dinámicamente (es decir, según las condiciones de PHP)::

	// src/Repository/ProductRepository.php

	// ...
	class ProductRepository extends ServiceEntityRepository
	{
		public function findAllGreaterThanPrice(int $price, bool $includeUnavailableProducts = false): array
		{
			// automatically knows to select Products
			// the "p" is an alias you'll use in the rest of the query
			$qb = $this->createQueryBuilder('p')
				->where('p.price > :price')
				->setParameter('price', $price)
				->orderBy('p.price', 'ASC');

			if (!$includeUnavailableProducts) {
				$qb->andWhere('p.available = TRUE');
			}

			$query = $qb->getQuery();

			return $query->execute();

			// to get just one result:
			// $product = $query->setMaxResults(1)->getOneOrNullResult();
		}
	}

Consultando con SQL
~~~~~~~~~~~~~~~~~

Además, puedes consultar directamente con SQL si lo necesitas::

	// src/Repository/ProductRepository.php

	// ...
	class ProductRepository extends ServiceEntityRepository
	{
		public function findAllGreaterThanPrice(int $price): array
		{
			$conn = $this->getEntityManager()->getConnection();

			$sql = '
				SELECT * FROM product p
				WHERE p.price > :price
				ORDER BY p.price ASC
				';
			$stmt = $conn->prepare($sql);
			$resultSet = $stmt->executeQuery(['price' => $price]);

			// returns an array of arrays (i.e. a raw data set)
			return $resultSet->fetchAllAssociative();
		}
	}

Con SQL, obtendrás datos sin procesar, no objetos (a menos que uses la funcionalidad `NativeQuery`_).

Configuración
-------------

Consulta la `referencia de configuración de Doctrine`_.

Relaciones y Asociaciones
------------------------------

Doctrine proporciona toda la funcionalidad que necesitas para administrar las relaciones de la base de datos (también conocidas como asociaciones), incluidas las relaciones ManyToOne, OneToMany, OneToOne y ManyToMany.

Para obtener información, consulta `Cómo trabajar con asociaciones/relaciones de Doctrine`_.

Pruebas de base de datos
----------------

Lee el artículo `cómo probar el código que interactúa con la base de datos`_.

Extensiones de Doctrine (Timestampable, Translatable, etc.)
-------------------------------------------------------

La comunidad de Doctrine ha creado algunas extensiones para implementar necesidades comunes como *"establecer el valor de la propiedad createdAt automáticamente al crear una entidad"*. Lee más sobre las `extensiones de Doctrine`_ disponibles y usa `StofDoctrineExtensionsBundle`_ para integrarlas en tu aplicación.

Aprende más
----------

* `Cómo trabajar con asociaciones/relaciones de Doctrine <https://symfony.com/doc/current/doctrine/associations.html>`_
* `Eventos de Doctrine <https://symfony.com/doc/current/doctrine/events.html>`_
* `Cómo implementar un formulario de registro <https://symfony.com/doc/current/doctrine/registration_form.html>`_
* `Cómo registrar funciones DQL personalizadas <https://symfony.com/doc/current/doctrine/custom_dql_functions.html>`_
* `Cómo usar Doctrine DBAL <https://symfony.com/doc/current/doctrine/dbal.html>`_
* `Cómo trabajar con múltiples administradores de entidades y conexiones <https://symfony.com/doc/current/doctrine/multiple_entity_managers.html>`_
* `Cómo definir relaciones con clases e interfaces abstractas <https://symfony.com/doc/current/doctrine/resolve_target_entity.html>`_
* `Cómo generar entidades a partir de una base de datos existente <https://symfony.com/doc/current/doctrine/reverse_engineering.html>`_
* `Cómo probar un repositorio de Doctrine <https://symfony.com/doc/current/testing/database.html>`_


	.. _`Doctrine`: https://www.doctrine-project.org/
	.. _`Symfony pack`: https://symfony.com/doc/current/setup.html#symfony-packs
	.. _`Cómo usar Doctrine DBAL`: https://symfony.com/doc/current/doctrine/dbal.html
	.. _`urlencode`: https://secure.php.net/manual/en/function.urlencode.php
	.. _`RFC 3986`: https://www.ietf.org/rfc/rfc3986.txt
	.. _`Tipos de Mapeo de Doctrine`: https://www.doctrine-project.org/projects/doctrine-orm/en/current/reference/basic-mapping.html
	.. _`Generador de Consultas`: https://www.doctrine-project.org/projects/doctrine-orm/en/current/reference/query-builder.html
	.. _`Doctrine Query Language`: https://www.doctrine-project.org/projects/doctrine-orm/en/current/reference/dql-doctrine-query-language.html
	.. _`Palabras Reservadas en SQL`: https://www.doctrine-project.org/projects/doctrine-orm/en/current/reference/basic-mapping.html#quoting-reserved-words
	.. _`DoctrineMongoDBBundle`: https://symfony.com/doc/current/bundles/DoctrineMongoDBBundle/index.html
	.. _`servicio Entity Manager`: https://symfony.com/doc/current/service_container.html#services-constructor-injection
	.. _`Contenedor de Servicios`: https://symfony.com/doc/current/service_container.html#services-constructor-injection
	.. _`Transacciones y Concurrencia`: https://www.doctrine-project.org/projects/doctrine-orm/en/current/reference/transactions-and-concurrency.html
	.. _`Validador de Symfony`: https://symfony.com/doc/current/validation.html
	.. _`configuración de validación`: https://symfony.com/doc/current/validation.html
	.. _`NotNull`: https://symfony.com/doc/current/reference/constraints/NotNull.html
	.. _`Type`: https://symfony.com/doc/current/reference/constraints/Type.html
	.. _`UniqueEntity`: https://symfony.com/doc/current/reference/constraints/UniqueEntity.html
	.. _`Length`: https://symfony.com/doc/current/reference/constraints/Length.html
	.. _`componente PropertyInfo`: https://symfony.com/doc/current/components/property_info.html
	.. _`componente de Formulario`: https://symfony.com/doc/current/forms.html
	.. _`Plataforma de API`: https://api-platform.com/docs/core/validation/
	.. _`restricciones de validación`: https://symfony.com/doc/current/reference/constraints.html
	.. _`DoctrineMigrationsBundle`: https://github.com/doctrine/DoctrineMigrationsBundle
	.. _`NativeQuery`: https://www.doctrine-project.org/projects/doctrine-orm/en/current/reference/native-sql.html
	.. _`límite de 767 bytes para el prefijo de clave de índice`: https://dev.mysql.com/doc/refman/5.6/en/innodb-limits.html
	.. _`Doctrine screencast series`: https://symfonycasts.com/screencast/symfony-doctrine
	.. _`PDO`: https://www.php.net/pdo
	.. _`extensiones de Doctrine`: https://github.com/doctrine-extensions/DoctrineExtensions
	.. _`StofDoctrineExtensionsBundle`: https://github.com/stof/StofDoctrineExtensionsBundle
	.. _`componente ExpressionLanguage`: https://symfony.com/doc/current/components/expression_language.html
	.. _`referencia de configuración de Doctrine`: https://symfony.com/doc/current/reference/configuration/doctrine.html
	.. _`Cómo trabajar con asociaciones/relaciones de Doctrine`: https://symfony.com/doc/current/doctrine/associations.html
	.. _`cómo probar el código que interactúa con la base de datos`: https://symfony.com/doc/current/testing/database.html

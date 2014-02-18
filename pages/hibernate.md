# Hibernate y Spring

------

Aunque JDBC podría ser muy útil y elegante, podríamos considerar el uso de frameworks de peristencia, mejor conocidos como Mapeadores Objeto Relacional(ORM por sus siglas en inglés). Estos últimos necesitan ser capaces de mapear las propiedades de los objetos a columnas de base de datos, y crear nuestras sentencias de operación y búsqueda por nosotros, quitándonos de escribir mucho del SQL. Adicionalmente necesitamos caracterísitcas más sofisticadas:

* _Lazy loading_ - Así como nuestros grafos de objetos llegan a ser más complicados, a veces no queremos obtener relaciones enteras de forma inmediata, si no parcialmente ir descubriendo y poblando las relaciones entre los objetos.
* _Eager fetching_ - Esto es lo opuesto a _lazy loading_. Esta característica nos permite obtener un grafo de objetos entero en un sólo query, muy útil para cargar de forma premeditada una relación _1-m_.
* _Cascading_ - Algunas veces los cambios a una tabla de base de datos debería resultar en cambios a otras tablas también.

<blockquote>
  <p>El uso de un ORM puede ahorrarte muchas líenas de código y horas de desarrollo.</p>
</blockquote>

Spring soporta la integración con [Hibernate](http://www.hibernate.org/), [Java Persistence API(JPA)](http://www.oracle.com/technetwork/java/javaee/tech/persistence-jsp-140049.html) y [Java Data Objects(JDO)](http://www.oracle.com/technetwork/java/index-jsp-135919.html) para la administración de recursos, la implementación de los DAO's, y las estrategias de transacciones. Puedes configurar todos los elementos soportados por el ORM a través de la Inyección de Dependencias. El estilo de integración recomendado es desarrollar lps DAO's planos con Hibernate, JPA y/ JDO, el estilo de usar los templates de Spring ya no es tan recomendado.

Adicionalmente, Spring agrega mejoras significativas en la capa de persistencia con el ORM de nuestra elección al momento de crear aplicaciones con acceso a datos. Los beneficios de usar Spring al crear los DAO's con un ORM son:

* Un testing más fácil. 
* Excepciones comúnes de acceso a datos a través de la jerarquía de excepciones.
* Administración general de recursos.
* Administración integrada de transacciones.

## Consideraciones para el uso de ORM's

El mayor objetivo de la integración de Spring es tener una capa limpia de persistencia, con cualquier tecnología de acceso a datos y transacciones, y para el acoplamiento flexible de los objetos de la aplicación. Con Spring debes saber que:

* Los servicios de negocio no tratan directamente con el manejo del acceso a datos o la estrategia de transacción
* No hay código para la búsqueda de un recurso JNDI, se hace con configuración
* Puedes cambiar las implementaciones de acceso a datos mucho más fácil
* Los objetos de la aplicación son muy simples de alambrar, incluso los de acceso a datos, haciéndolo más fácil de reusar.
* No hay limitantes son Spring al respecto del uso del ORM
* En una aplicación tipica Spring, muchos objetos importantes son sólo JavaBeans: templates, DAO's, manejadores de transacciones, servicios de negocio que usan los DAO's, etc.

Spring promueve soluciones simple para el manejo apropiado del manejo de recursos, usando el IoC a través del templating en el caso de JDBC y aplicando interceptores AOP para las tecnologías de ORM.

La infraestructura de Spring provee del manejo apropiado de recursos y conversión de excepciones de API's específicas hacia una jerarquía consistente, aplicable a cualquier estrategia de acceso a datos.

### Hibernate en breve.



## El SessionFactory

No vamos a considerar y profundizar el uso y/o configuración de Hibernate como parte de este entrenamiento, debido a que nuestro enfoque va más dirigido a mostrarte la integración que Spring tiene con el ORM. Sin embargo, te recomendamos leas [la guía de usuario de Hibernate](http://docs.jboss.org/hibernate/orm/4.3/manual/en-US/html/), pues encontrarás todo lo que necesitas saber para comprender el mapeo de los objetos y la forma en que se puede personalizar los objetos de Hibernate. Sin embargo, los casos que cubrimos aquí son los más signifcativos y los que podrás encontrar/crear en tus aplicaciones.**Añun así, estamos abiertos a las preguntas que tengas al respecto del uso de Hibernate**.

<div class="bs-callout bs-callout-warning">
<h4><i class="icon-coffee"></i> Información de utilidad</h4>
  <p>
    Spring 4 soporta/requiere Hibernate 3.6+ y ya Hibernate 4.x, teniendo en consideración que Hibernate 4.3 es un proveedor de JPA 2.1, lo mismo aplica para Hibernate Validator 5.0 y Bean Validation 1.1. Y ninguno de los anteriores es soportado por Spring 3.2.x, así que tomalo en cuenta.
  </a>
  </p>
</div>

Para evitar atar nuestros objetos de la aplicación a recursos _fijos(hard-coded)_, podemos definir recursos como el `DataSource` o un `SessionFactory`


## Implementación de DAO's con Hibernate



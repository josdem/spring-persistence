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

<div class="row">
  <div class="col-md-6">
    <h4><i class="icon-code"></i> Arquitectura de alto nivel</h4>
    <img src="img/overview.png" alt="overview"/>
  </div>
  <div class="col-md-6">
    <h4><i class="icon-code"></i> Arquitectura mínima</h4>
    <img src="img/lite.png" alt="lite"/>
  </div>
</div>

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> Arquitectura comprensiva</h4>
    <img src="img/full_cream.png" alt="overview"/>
  </div>
</div>

#### API's esenciales

* SessionFactory (`org.hibernate.SessionFactory`) - Un objeto _thread-safe_, con cache de mapeos compilados para una sola base de datos.Opcionalmente mantiene un cache de segundo nivel de datos que es reusable entre transacciones en un proceso o en un cluster.
* Session (`org.hibernate.Session`) - Un objeto único, de vida corta representando una conversación entre la aplicación y el almacén persistente. Rodea una conexión JDBC dle tipo `java.sql.Connection`. Fábrica de objetos `Transaction`. Mantiene un caché de primer nivel de objetos y colecciones persistentes; este caché es usado cuando se navega el grafo de objetos.
* Objetos y colecciones persistentes - Objetos que contienen el estado persistente y las funciones de negocio. Pueden ser JavaBeans/POJO ordinarios. Están asociados con una y sólo una `Session`, una vez que este objeto se cierra los objetos son desasociados _detached_ y libres de usarse en cualquier capa de la aplicación.
* Objetos y colecciones transitorios - Instancias de clases persitentes que no están actualmente asociadas con un objeto `Session`. Quizá han sido instanciadas por la aplicación pero no han sido persistidas, o quizá han sido instanciadas por un objeto `Session` que ya ha sido cerrado.
* Transaction (`org.hibernate.Transaction`) - Un objeto de vida aún más corta que `Session` usado por la aplicación para unidades atómicas de trabajo.
* ConnectionProvider (`org.hibernate.connection.ConnectionProvider`) -  Una fábrica para un pool de conexiones de conexiones JDBC. No es expuesto por la aplicación pero se puede extender.
* TransactionFactory (`org.hibernate.TransactionFactory`) - Una fábrica que instancias `Transaction`. No es expuesto por la aplicación pero se puede extender.

## El SessionFactory

No vamos a considerar y profundizar el uso y/o configuración de Hibernate como parte de este entrenamiento, debido a que nuestro enfoque va más dirigido a mostrarte la integración que Spring tiene con el ORM. Sin embargo, te recomendamos leas [la guía de usuario de Hibernate](http://docs.jboss.org/hibernate/orm/4.3/manual/en-US/html/), pues encontrarás todo lo que necesitas saber para comprender el mapeo de los objetos y la forma en que se puede personalizar los objetos de Hibernate. Sin embargo, los casos que cubrimos aquí son los más signifcativos y los que podrás encontrar/crear en tus aplicaciones.**Aún así, estamos abiertos a las preguntas que tengas al respecto del uso de Hibernate**.

### Configuración programática con Hibernate

<div class="row">
  <div class="col-md-4">
    <h4><i class="icon-code"></i> Configuración XML</h4>
    <script type="syntaxhighlighter" class="brush: java;"><![CDATA[
      Configuration cfg = new Configuration()
      .addResource("User.hbm.xml")
      .addResource("Project.hbm.xml");
    ]]></script>
  </div>
  <div class="col-md-4">
    <h4><i class="icon-code"></i> Configuración Java con Anotaciones</h4>
    <script type="syntaxhighlighter" class="brush: java;"><![CDATA[
      Configuration cfg = new Configuration()
      .addClass(com.makingdevs.model.User.class)
      .addClass(com.makingdevs.model.Project.class);
    ]]></script>
  </div>
  <div class="col-md-4">
    <h4><i class="icon-code"></i> Configuración con propiedades</h4>
    <script type="syntaxhighlighter" class="brush: java;"><![CDATA[
      Configuration cfg = new Configuration()
      .addClass(com.makingdevs.model.User.class)
      .addClass(com.makingdevs.model.Project.class)
      .setProperty("hibernate.dialect", "org.hibernate.dialect.MySQLInnoDBDialect")
      .setProperty("hibernate.connection.datasource", "java:comp/env/jdbc/test")
      .setProperty("hibernate.order_updates", "true");
    ]]></script>
  </div>
</div>

<div class="row">
  <div class="col-md-6">
    <h4><i class="icon-code"></i> Uso del SessionFactory</h4>
    <script type="syntaxhighlighter" class="brush: java;"><![CDATA[
      SessionFactory sessions = cfg.buildSessionFactory();
      Session session = sessions.openSession(); // open a new Session
    ]]></script>
  </div>
  <div class="col-md-6">
    <h4><i class="icon-file"></i> hibernate.properties</h4>
    <script type="syntaxhighlighter" class="brush: plain;"><![CDATA[
hibernate.connection.driver_class = org.postgresql.Driver
hibernate.connection.url = jdbc:postgresql://localhost/makingdevs
hibernate.connection.username = myuser
hibernate.connection.password = secret
hibernate.c3p0.min_size=5
hibernate.c3p0.max_size=20
hibernate.c3p0.timeout=1800
hibernate.c3p0.max_statements=50
hibernate.dialect = org.hibernate.dialect.PostgreSQL82Dialect
    ]]></script>
  </div>
</div>

<div class="bs-callout bs-callout-warning">
<h4><i class="icon-coffee"></i> Información de utilidad</h4>
  <p>
    Spring 4 soporta/requiere Hibernate 3.6+ y ya Hibernate 4.x, teniendo en consideración que Hibernate 4.3 es un proveedor de JPA 2.1, lo mismo aplica para Hibernate Validator 5.0 y Bean Validation 1.1. Y ninguno de los anteriores es soportado por Spring 3.2.x, así que tomalo en cuenta.
  </a>
  </p>
</div>

Para evitar atar nuestros objetos de la aplicación a recursos _fijos(hard-coded)_, podemos definir recursos como el `DataSource` o un `SessionFactory` como beans del contenedor de Spring. Los objetos de la aplicación que necesitan acceder a los recursos reciben las referencias a dichas instancias predefinidas a través de la inyección de dependencias.

<div class="row">
  <div class="col-md-6">
    <h4><i class="icon-code"></i> HibernateAppCtx.xml</h4>
    <script type="syntaxhighlighter" class="brush: xml;"><![CDATA[
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="sessionFactory"
    class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <property name="mappingResources">
      <list>
        <value>com/makingdevs/model/User.hbm.xml</value>
        <value>com/makingdevs/model/Project.hbm.xml</value>
        <value>com/makingdevs/model/UserStory.hbm.xml</value>
        <value>com/makingdevs/model/Task.hbm.xml</value>
      </list>
    </property>
    <property name="hibernateProperties">
      <value>
        hibernate.dialect=org.hibernate.dialect.H2Dialect
      </value>
    </property>
  </bean>

</beans>
    ]]></script>
  </div>
  <div class="col-md-6">
    <h4><i class="icon-file"></i> HibernateAppCtxTests.java</h4>
    <script type="syntaxhighlighter" class="brush: java;"><![CDATA[
package com.makingdevs.practica6;

import static org.springframework.util.Assert.notNull;

import org.hibernate.SessionFactory;
import org.junit.FixMethodOrder;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.MethodSorters;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "HibernateAppCtx.xml", "../practica1/DataSourceWithNamespace.xml" })
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class HibernateAppCtxTests {

  @Autowired
  SessionFactory sessionFactory;

  @Test
  public void test0SessionFactory() {
    notNull(sessionFactory);
  }

  @Test
  public void test1Session() {
    org.springframework.util.Assert.notNull(sessionFactory.openSession());
  }

}
    ]]></script>
  </div>
</div>


## Implementación de DAO's con Hibernate



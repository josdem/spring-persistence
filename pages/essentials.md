# Conceptos esenciales de acceso a datos con Spring

----

En las recientes décadas, una filosofía se ha desarrollado como una tendencia de diseño en la comunidad de desarrollo orientado a objetos. El diseño orientado a al dominio con dos premisas:

* Para la mayoría de los proyectos de software, el enfoque principal debería ser el dominio y la lógica que yace en él.
* Diseños de dominio complejos deberían estar basados sobre un modelo.

DDD no es una tecnología o una metodología. Es una forma de pensar y establecer un conjunto de prioridades, apuntando a acelerar proyectos de software que tiene que trata con dominios complicados.

Cuando la complejidad se sale de las manos, el software ya no puede ser entendido lo suficientemente bien para ser fácilmente cambiado o extendido. En contraste, un buen diseño puede dar oportunidades de aplicar caracterísiticas complejas.

Algunos de los factores de diseño son tecnológicos, y un gran esfuerzo se pone en el diseño de reders, bases de datos o alguna otra dimensión tecnológica de software. 

Todavía, la complejidad más significativa de muchas aplicaciones no es técnica. Es el dominio en sí mismo, la actividad o el negocio del usuario. Cuando esta complejidad de dominio no se trata en el diseño, no importa que la tecnología de infraestructura está bien concebida. Un diseño exitoso debe abordar sistemáticamente este aspecto central del software.

<div id="1"></div>

## El uso de JDBC

<blockquote>
  <p>La API de JDBC fue diseñada para mantener las cosas simples.</p>
</blockquote>

Esto significa que JDBC facilita las tareas de bases de datos. Siendo que la API de JDBC puede acceder cualquier tipo de datos tabulares, especialmente los datos almacenados en Bases de datos relacionales; ayuda a escribir aplicaciones Java que administran tres actividades de programación:

* Conectarse a una fuente de datos, como una base de datos.
* Enviar busquedas y sentencias de actualización a la base de datos.
* Entregar y procesar los resultados obtenidos desde la base de datos en una respuesta a la búsqueda

El siguiente fragmento de cófigo representa  una idea lo que el uso de JDBC implicaría:

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> UsingJDBCForSimpleQuery.java</h4>
    <script type="syntaxhighlighter" class="brush: java; highlight:[19,22]"><![CDATA[
import java.sql.*;

class UsingJDBCForSimpleQuery{
  public static void main( String args[] ){
    try{
      Class.forName( "sun.jdbc.odbc.JdbcOdbcDriver" );

      Connection conn = DriverManager.getConnection( "jdbc:odbc:Database" );

      for( SQLWarning warn = conn.getWarnings(); warn != null; warn = warn.getNextWarning() ){
        System.out.println( "SQL Warning:" );
        System.out.println( "State  : " + warn.getSQLState()  );
        System.out.println( "Message: " + warn.getMessage()   );
        System.out.println( "Error  : " + warn.getErrorCode() );
      }

      Statement stmt = conn.createStatement();

      ResultSet rs = stmt.executeQuery( "SELECT * FROM my_custom_table" );

      while( rs.next() )
        System.out.println( rs.getString(1) );

      rs.close();
      stmt.close();
      conn.close();
    }
    catch( SQLException se ){
      System.out.println( "SQL Exception:" );

      while( se != null ){
        System.out.println( "State  : " + se.getSQLState()  );
        System.out.println( "Message: " + se.getMessage()   );
        System.out.println( "Error  : " + se.getErrorCode() );

        se = se.getNextException();
      }
    }
    catch( Exception e ){
      System.out.println( e );
    }
  }
}
    ]]></script>
  </div>
</div>

Algunos aspectos importantes a considerar dentro del uso de JDBC y basados en el código anterior son:

* **La API de JDBC** - Provee de acceso programático a datos relacionales desde el lenguaje Java. Con la APi se pueden ejecutar sentencias SQL, entregar resultados, y propagar cambios al DataSource suscrito. 
* **El JDBC Driver Manager** - La clase `DriverManager` define objetos los cuales pueden conectar aplicaciones Java a un driver JDBC. El `DriverManager` tiene tradicionalmente la columna vertebral de la arquitectura de JDBC.Es pequeño y simple. La extensión de paquetes estándar `javax.naming` y `javax.sql` permite usar un `DataSource` registrado en un servicio de nombrados JNDI para establecer una conexión con el DataSource.

<blockquote>
  <p>En las aplicaciones empresariales la implementación de la lógica de negocio y el reflejo de los procesos en un software es mucho más importante que un problema de acceso a datos.</p>
</blockquote>

<div id="2"></div>

## Conociendo Spring JDBC

EL valor agregado proveído por la abstracción de Spring JDBC es quizá descrita de una mejor manera en la siguiente lista, en donde se muestra que acciones son responsabilidad de Spring y cuales son del desarrollador:

<ul>
  <li><span class="label label-primary">DevOps</span> Definir parámetros de conexión</li>
  <li><span class="label label-success">Spring</span> Abrir la conexión</li>
  <li><span class="label label-primary">DevOps</span> Indicar la sentencia SQL</li>
  <li><span class="label label-primary">DevOps</span> Declarar los parámetros y proveer de los valores</li>
  <li><span class="label label-success">Spring</span> Preparar y ejecutar la sentencia</li>
  <li><span class="label label-success">Spring</span> Establecer el loop de iteración de resultados</li>
  <li><span class="label label-primary">DevOps</span> Manejar cada iteración</li>
  <li><span class="label label-success">Spring</span> Procesar cualquier excepción</li>
  <li><span class="label label-success">Spring</span> Manejar transacciones</li>
  <li><span class="label label-success">Spring</span> Cerrar la conexión, la sentencia y el resultset</li>
</ul>

Uno de los objetivos de Spring es permitir el desarrollo de aplicaciones orientada a objetos a través del desarrollo de interfaces, y Spring JDBC también aprovecha este hecho.

Muchos desarrolladores, se refieren a los objetos de persistencia de una aplicación como _repositories_. 

<blockquote>
  <p>Los objetos de servicio NO manejan su propio acceso a datos. En lugar de ello, delegan el acceso a los DAO's. La interfaz DAO mantiene el bajo acoplamiento al objeto de servicio.</p>
</blockquote>

### Patrón de diseño DAO

Usa un Objeto de Acceso a Datos(DAO) para abstraer y encapsular todo el acceso al DataSource. El DAO administra la conexión con el DataSource para obtener y acceder datos.

El DAO implementa el mecanismo de acceso requerido para trabajar con el DataSource, el cual podría ser una base de datos relacional. El DAO esconde completamente la implementación del DataSource de los componentes que lo llaman, debido a que la interfaz no lo puede modificar por sí misma.DAO

![Alt dao](/img/dao.jpg)

Si planteamos nuestros objetos de acceso a datos de esta forma, entonces los objetos de servicio podrán acceder a las interfaces de las declaraciones de los DAO's para manipular la estructura de la base de datos y los podremos desacoplar en dado caso de que necesitemos algún mock o cambio de implementación. Además, lo hace más sencillo de probar pues podemos definir _pruebas de unidad_ reales en base a las llamadas que se deberían ejecutar en los colaboradores.

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-file"></i> GenericDao.java</h4>
    <script type="syntaxhighlighter" class="brush: java"><![CDATA[
package com.makingdevs.dao;

import java.io.Serializable;
import java.util.List;

/** The Generic DAO for all persistence interfaces */
public interface GenericDao<T, PK extends Serializable> {

  /** Persist the newInstance object into database */
  PK create(T newInstance);

  /**
   * Retrieve an object that was previously persisted to the database using the
   * indicated id as primary key
   */
  T read(PK id);

  /** Save changes made to a persistent object. */
  void update(T transientObject);

  /** Remove an object from persistent storage in the database */
  void delete(T persistentObject);

  /** Retrieves a list of instances */
  List<T> findAll();

  /** Count the current instances persisted */
  int countAll();
}
    ]]></script>
  </div>
</div>

El patrón de diseño DAO debe ser bien conocido por cualquier desarrollador Java Empresarial, sin embargo, debemos clarificar algunas cosas para la implmenetación:

* Todos los accesos de la base de datos en el sistema son hechos a través de los DAO's para mantener la encapsulación.
* Cada instancia del DAO es responsable por un objeto de dominio primario o entidad.
* Si un objeto de dominio tiene un ciclo de vida independiente, debería tener su propio DAO.
* El DAO es responsable de la creación, lectura(por llave primaria), actualizaciones y el borrado de un objeto de dominio.(CRUD)
* El DAO quizá permita búsquedas basados en criterios distintos a la llave primaria. Podemos referirnos a esos métodos como _finder methods_ o _finders_. El valor que se regresa de un _finder_ es normalmente una colección de objetos de dominio de los cuales el DAO es responsable.
* El DAO **no** es responsable por el manejo de transacciones, sesiones o conexiones, esto últimos son manejados fuera del DAO para mantener la flexibilidad.
* Evitamos en la medida de lo posible el uso de _casts_ explícito.
* Aún aquí, es válido usar principios de POO como herencia y polimorfismo.

<div class="row">
  <div class="col-md-3">
    <h4><i class="icon-file"></i> UserDao.java</h4>
    <script type="syntaxhighlighter" class="brush: java"><![CDATA[
package com.makingdevs.dao;

import com.makingdevs.model.User;

public interface UserDao extends GenericDao<User, Long> {
  User findByUsername(String username);
  // So many methods as you want...
}
    ]]></script>
  </div>
  <div class="col-md-3">
    <h4><i class="icon-file"></i> ProjectDao.java</h4>
    <script type="syntaxhighlighter" class="brush: java"><![CDATA[
package com.makingdevs.dao;

import com.makingdevs.model.Project;

public interface ProjectDao extends GenericDao<Project, Long> {
  Project findByCodename(String codename);
}
    ]]></script>
  </div>
  <div class="col-md-3">
    <h4><i class="icon-file"></i> GenericDao.java</h4>
    <script type="syntaxhighlighter" class="brush: java"><![CDATA[
package com.makingdevs.dao;

import java.util.List;

import com.makingdevs.model.Project;
import com.makingdevs.model.UserStory;

public interface UserStoryDao extends GenericDao<UserStory, Long> {
  List<UserStory> findAllByEffortBetween(Integer lowValue, Integer maxValue);
  List<UserStory> findAllByPriorityBetween(Integer lowValue, Integer maxValue);
  List<UserStory> findAllByProject(Project project);
}
    ]]></script>
  </div>
  <div class="col-md-3">
    <h4><i class="icon-file"></i> GenericDao.java</h4>
    <script type="syntaxhighlighter" class="brush: java"><![CDATA[
package com.makingdevs.dao;

import java.util.List;

import com.makingdevs.model.Task;
import com.makingdevs.model.TaskStatus;
import com.makingdevs.model.User;
import com.makingdevs.model.UserStory;

public interface TaskDao extends GenericDao<Task, Long> {
  List<Task> findAllByDescriptionLike(String description);
  List<Task> findAllByUserStoryAndStatusEquals(UserStory userStory, TaskStatus taskStatus);
  List<Task> findAllByUser(User user);
}
    ]]></script>
  </div>
</div>

<div id="3"></div>

## Manejo de excepciones

Si has escrito código con la API de JDBC sin Spring, entonces debés de conocer que no puedes hacer nada sin cachar siempre `SQLException`. El significado de la excepción es que algo malo paso cuando se intentó acceder a la base de datos, pero el detalle de la excepción en la mayoría de los casos no dice mucho que pueda ayudar.

Algunos problemas comúnes que causan que se arroje `SQLException` son:

* La aplicación no es capaz de conectarse a la base de datos.
* El query que se esta ejecutando tiene errores en su sintacis.
* Las tablas y las columnas referidas en la búsqueda no existen.
* Un intento fue hecho al insertar o actualizar valores que violan las restricciones de la base de datos.

<blockquote>
  <p>¿Cómo debe ser tratada <code>SQLException</code> cuando se atrape? Realmente, si falla la base de datos no podemos hacer nada...</p>
</blockquote>

Y si no podemos hacer nada entonces **¿por qué debemos tratarla?** Incluso si desearamos tratarla tenemos que profundizar en ella para obtener la verdadera causa del error. Algunos frameworks de persistencia ofrecen una jerarquía de excepciones, cada una de ellas apuntando a un problema diferente, esto hace crear bloques `try/catch` para excepciones que se pueden esperar de antemano. El problema con ello es que cada jerarquía de excepciones es referente exclusiva al framework.

Spring provee **una jerarquía de excepciones** también conocida como **una plataforma agnóstica de excepciones para la persistencia con Spring** que resuelve los problemas de falta de claridad en los errores y las jerarquías de otros frameworks, incluso la de JDBC.

![Alt dao](/img/data_access_exception.jpg)

<div class="bs-callout bs-callout-info">
<h4><i class="icon-coffee"></i> Información de utilidad</h4>
  <p>
    Te recomendamos que explores <a href="http://docs.spring.io/spring/docs/4.0.x/javadoc-api/org/springframework/dao/DataAccessException.html">la documentación de DataAccessException</a>, pues hay actualizaciones al respecto de la jerarquía de excepciones y es muy bueno tenerlo como referencia.
  </a>
  </p>
</div>

<div id="4"></div>

## Uso de templates con Spring

<blockquote>
  <p>Un método de un template define el esqueleto de un proceso.</p>
</blockquote>

El proceso por si mísmo es fijo, nunca cambia. Algunos pasos del proceso son fijos también, algunos pasos pasan cada vez que se ejecuta un proceso. En algunos puntos, el proceso delega el trabajo a una subclase para llenarlo con algunas implementaciones con detalles muy específicos, es decir, es la parte variable del proceso. 

En términos de software, un método de un template delega porciones específicas de una implementación del proceso a una interface. Diferentes implementaciones de esta interface definen implementaciones específicas de esta porción del proceso. Este es el mismo patrón que Spring aplica al acceso a datos. Sin importar la tecnología ciertos pasos del acceso a datos son requeridos. Por ejemplo, siempre necesitamos obtener una conexión al DataSource y liberar recursos cuando ha terminado, pero cada método de acceso a datos que escribimos es ligeramente doferente. Buscamos por diferentes objetos y actualizamos en diferentes maneras.

Spring separa las partes fijas y las variables del proceso de acceso a datos en dos distintas clases: _templates_ y _callbacks_. Los _templates_ administran la parte fija del proceso mientras que el código de acceso de datos personalizados se maneja en los _callbacks_.

En un template del DAO:

* Se preparan los recursos
* Se comienza la transacción
* ....
* Se hace commit/rollback de la transacción
* Se cierran recursos y se manejan errores

En un callback del DAO:

* ....
* Se ejecuta una transacción
* Se regresan los datos y se tratan
* ...

### Templates de acceso a datos

Springframework viene con varios templates de acceos a datos, cada uno adecuado para un mecanismo de persistencia diferente. Los más comúnes precedidos por el nombre de paquete `org.springframework`:

* JDBC
    * `jdbc.core.JdbcTemplate`
    * `jdbc.core.namedparam.NameParameterJdbcTemplate`
    * `jdbc.core.simple.SimpleJdbcTemplate`
* Hibernate
    * `orm.hibernate.HibernateTemplate`
    * `orm.hibernate3.HibernateTemplate`
* JPA
    * `orm.jpa.JpaTemplate`
* iBatis
    * `orm.ibatis.SqlMapClientTemplate`

<blockquote>
  <p>Lo único que necesita un <code>JdbcTemplate</code> es un <code>DataSource</code>, y tiene varios métodos de conveniencia que ayudan a realizar operaciones de base de datos de una forma muy simple.</p>
</blockquote>

## La base de datos y el namespace

El paquete `org.springframework.jdbc.datasource.embedded` provee del soporte embebido de bases de datos con motores Java. Soporta [HSQL](http://www.hsqldb.org/), [H2](http://www.h2database.com/) y [Derby](http://db.apache.org/derby) de forma nativa. Aunque se puede extender el API para conectar nuevos tipos de bases de datos e implementaciones de `DataSource`.

<div class="bs-callout bs-callout-info">
<h4><i class="icon-coffee"></i> Información de utilidad</h4>
  <p>
    Una bases de datos embebida es útil durante la fase de desarrollo de un proyecto por que es de naturaleza ligera. Los beneficios incluyen una fácil configuración, tiempo de inicio rápido, capaz de probarse, y la habilidad de evolucionar el SQL(estructira) durante el desarrollo.
  </a>
  </p>
</div>

Para embeber la base de datos necesitamos crear algunos scripts que nos permitan definir la estructura(DDL) y después asignarlos a nuestro bean de Spring.

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> DataSourceWithNamespace.xml</h4>
    <script type="syntaxhighlighter" class="brush: xml;"><![CDATA[
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:jdbc="http://www.springframework.org/schema/jdbc"
  xmlns:jee="http://www.springframework.org/schema/jee"
  xmlns:util="http://www.springframework.org/schema/util"
  xsi:schemaLocation="http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-4.0.xsd
    http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-4.0.xsd
    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd">


  <jdbc:embedded-database type="H2" id="dataSource">
    <jdbc:script location="classpath:/com/makingdevs/scripts/user.sql"/>
    <jdbc:script location="classpath:/com/makingdevs/scripts/project.sql"/>
    <jdbc:script location="classpath:/com/makingdevs/scripts/user_story.sql"/>
    <jdbc:script location="classpath:/com/makingdevs/scripts/task.sql"/>
    <jdbc:script location="classpath:/com/makingdevs/scripts/constraints.sql"/>
  </jdbc:embedded-database>

</beans>
    ]]></script>
  </div>
</div>

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> DeclaringDataSourceTests.java</h4>
    <script type="syntaxhighlighter" class="brush: java;"><![CDATA[
package com.makingdevs.practica1;

import java.sql.SQLException;

import javax.sql.DataSource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.util.Assert;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "DataSourceWithNamespace.xml" })
public class DeclaringDataSourceTests {
  
  @Autowired
  DataSource dataSource;

  @Test
  public void test() throws SQLException {
    Assert.notNull(dataSource);
    Assert.notNull(dataSource.getConnection());
  }

}
    ]]></script>
  </div>
</div>

Aunque potencialmente, se podría utilizar cualquier manejador de base de datos que provea de un Driver de Conexión el cual permitá manipularla.

<div id="5"></div>

## Control de las conexiones(El *DataSource)

Independientemente de cual forma de soporte en Spring uses, necesitarás configurar una referencia a un `DataSource`. Spring ofrece varias opciones para configurar beans DataSource en una aplicación:

* DataSources que son definidos por el driver
* DataSources que son buscados por un recurso JNDI
* DataSources que son pool de conexiones

Adicionalmente, Spring ofrece dos tipos de clases para DataSource del paquete `org.springframework.jdbc.datasource`:

* [`DriverManagerDataSource`](http://docs.spring.io/spring/docs/4.0.0.RELEASE/javadoc-api/org/springframework/jdbc/datasource/DriverManagerDataSource.html) Regresa una nueva conexión cada vez que una conexión es solicitada.
* [`SingleConnectionDataSource`](http://docs.spring.io/spring/docs/4.0.0.RELEASE/javadoc-api/org/springframework/jdbc/datasource/SingleConnectionDataSource.html) Regresa la misma conexión cada veza que la conexión es solicitada.

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> DriverManagerDataSource</h4>
    <script type="syntaxhighlighter" class="brush: xml;"><![CDATA[
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
  <property name="driverClassName" value="org.hsqldb.jdbcDriver" />
  <property name="url" value="jdbc:hsqldb:hsql://localhost/spitter/spitter" />
  <property name="username" value="sa" />
  <property name="password" value="" />
</bean>
    ]]></script>
  </div>
</div>

### Uso de Commons DBCP y/o C3P0

<div class="row">
  <div class="col-md-6">
    <h4><i class="icon-code"></i> DriverManagerDataSource</h4>
    <script type="syntaxhighlighter" class="brush: xml;"><![CDATA[
<util:properties id="dbProperties" location="classpath:/com/makingdevs/practica1/db.properties" />

<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
  <property name="username" value="#{dbProperties['mainDataSource.username']}"/>
  <property name="password" value="#{dbProperties['mainDataSource.password']}"/>
  <property name="url" value="#{dbProperties['mainDataSource.url']}"/>
  <property name="driverClassName" value="#{dbProperties['mainDataSource.driverClassName']}"/>
</bean>
    ]]></script>
  </div>
  <div class="col-md-6">
    <h4><i class="icon-code"></i> DriverManagerDataSource</h4>
    <script type="syntaxhighlighter" class="brush: xml;"><![CDATA[
<util:properties id="dbProperties" location="classpath:/com/makingdevs/practica1/db.properties" />

<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
  <property name="user" value="#{dbProperties['mainDataSource.username']}"/>
  <property name="password" value="#{dbProperties['mainDataSource.password']}"/>
  <property name="jdbcUrl" value="#{dbProperties['mainDataSource.url']}"/>
  <property name="driverClass" value="#{dbProperties['mainDataSource.driverClassName']}"/>
</bean>
    ]]></script>
  </div>
</div>

Con Spring podemos configurar una referencia a un DataSource que esta dentro de un JNDI y alambrarlo a cualquier otra clase que lo necesite. Con el namespace `jee` tenemos disponible el tag `<jee:jndi-lookup>` que ayuda a buscarlo e inicializarlo.

```
<jee:jndi-lookup id="dataSource" jndi-name="/jdbc/MakingDevsDS" resource-ref="true" />
```

**Nota:** El uso de `resource-ref="true"` antepone al nombre JNDI `java:comp/env/`.

## Modelado de las operaciones como objetos Java(Caso de estudio)


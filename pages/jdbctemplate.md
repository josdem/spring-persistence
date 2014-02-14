# JdbcTemplate y DAO Support

------

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

Springframework viene con varios templates de acceso a datos, cada uno adecuado para un mecanismo de persistencia diferente. Los más comúnes precedidos por el nombre de paquete `org.springframework`:

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

## Usando el *JdbcTemplate

La clase `JdbcTemplate` es central en el paquete de JDBC. Maneja la creación y liberación de recursos, los cuales ayudan a evitar errores comúnes como el cerrado de las conexiones. 
Actúa tareas básicas del flujo de JDBC como la creación de sentencias y su ejecución, dejando al código de la aplicación proveer el SQL y extraer los resultados.

La clase `JdbcTemplate` ejecuta consultas, sentencias de actualización y llamadas a procedimientos almacenados, realiza las iteraciones sobre el `ResultSet` y la extracción de los valores de los parámetros. Además atrapa las excepciones JDBC y las traduce a una jerarquía de excepciones más informativa definidas por Spring.

<blockquote>
  <p>Lo único que necesita un <code>JdbcTemplate</code> es un <code>DataSource</code>, y tiene varios métodos de conveniencia que ayudan a realizar operaciones de base de datos de una forma muy simple. El DataSource siempre debe ser configurado como un bean en el contenedor de Spring.</p>
</blockquote>

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-file"></i> JdbcTemplateConfig.java</h4>
    <script type="syntaxhighlighter" class="brush: java"><![CDATA[
package com.makingdevs.practica2;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ImportResource;
import org.springframework.jdbc.core.JdbcTemplate;

@Configuration
@ImportResource(value={"classpath:/com/makingdevs/practica1/DataSourceWithNamespace.xml"})
public class JdbcTemplateConfig {

  @Autowired
  DataSource dataSource;
  
  @Bean
  public JdbcTemplate jdbcTemplate(){
    return new JdbcTemplate(dataSource);
  }
}
    ]]></script>
  </div>
</div>

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-file"></i> UsingJdbcTemplateTests.java</h4>
    <script type="syntaxhighlighter" class="brush: java"><![CDATA[
package com.makingdevs.practica2;

import static org.junit.Assert.assertEquals;

import java.util.Date;

import org.junit.FixMethodOrder;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.MethodSorters;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.dao.DataAccessException;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { JdbcTemplateConfig.class })
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class UsingJdbcTemplateTests {

  @Autowired
  JdbcTemplate jdbcTemplate;

  @Test
  public void test1CountWithJdbcTemplate() {
    int rowCount = jdbcTemplate.queryForObject("SELECT count(*) FROM user", Integer.class);
    assertEquals(7, rowCount);
  }

  @Test
  public void test2CountBindingVariableWithJdbcTemplate() {
    int rowCount = jdbcTemplate.queryForObject("SELECT count(*) FROM user WHERE username = ?", Integer.class,
        "makingdevs");
    assertEquals(1, rowCount);
  }

  @Test
  public void test3QueryStringWithJdbcTemplate() {
    String projectName = jdbcTemplate.queryForObject("SELECT code_name FROM project WHERE id = ?", new Object[] { 4L },
        String.class);
    assertEquals("AGILE-TASKBOARD", projectName);
  }

  @Test
  public void test4InsertWithJdbcTemplate() {
    int rowCount = jdbcTemplate.update(
        "INSERT INTO project(CODE_NAME,DESCRIPTION,NAME,DATE_CREATED,LAST_UPDATED) values(?,?,?,?,?)", "PROJECT",
        "This is a new project", "New project", new Date(), new Date());
    assertEquals(1, rowCount); // Why this is 1?
    String projectDescription = jdbcTemplate.queryForObject("SELECT description FROM project WHERE CODE_NAME = ?", new Object[] { "PROJECT" },
        String.class);
    assertEquals(projectDescription, "This is a new project");
  }
  
  @Test
  public void test5UpdateWithJdbcTemplate() {
    int rowCount = jdbcTemplate.update(
        "UPDATE project SET DESCRIPTION = ?,NAME = ?,LAST_UPDATED = ? WHERE CODE_NAME = ?", "The project is updated",
        "Project Updated", new Date(), "PROJECT");
    assertEquals(1, rowCount);
  }
  
  @Test(expected=DataAccessException.class)
  public void test6DeleteWithJdbcTemplate() {
    int rowCount = jdbcTemplate.update(
        "DELETE FROM project WHERE CODE_NAME = ?", "PROJECT");
    assertEquals(1, rowCount);
    String projectDescription = jdbcTemplate.queryForObject("SELECT description FROM project WHERE CODE_NAME = ?", new Object[] { "PROJECT" },
        String.class);
  }
  
  @Test
  public void test7CreateTempTableWithJdbcTemplate(){
    jdbcTemplate.execute("CREATE TABLE TEMP(ID INTEGER, NAME VARCHAR(100))");
  }

}
    ]]></script>
  </div>
</div>

<div class="bs-callout bs-callout-info">
<h4><i class="icon-coffee"></i> Información de utilidad</h4>
  <p>
    Te recomendamos explorar <a href="http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html">la documentación del JdbcTemplate</a> para que puedas determinar las diferencias entre los métodos execute y update.
  </a>
  </p>
</div>

Las instancias de `JdbcTemplate` son _threadsafe_ una vez que son configuradas. Esto es importante por que significa que podemos configurar una sola instancia e inyectar la refencia compartida de forma segura referenciandola en múltiples componentes(DAO).

El `JdbcTemplate` es _stateful_, en lo que mantiene la referencia al `DataSource`, pero este estado no es _conversacional_.

### `NamedParameterJdbcTemplate`

La clase `NamedParameterJdbcTemplate` agrega el soporte para la programación de sentencias JDBC usando parámetros nombrados, en lugar de los marcadores _"?"_. Lo que hace es rodear al `JdbcTemplate` para después delegar el trabajo.

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-file"></i> NamedJdbcTemplateAppCtx.xml</h4>
    <script type="syntaxhighlighter" class="brush: xml"><![CDATA[
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
    <constructor-arg ref="dataSource"/>
  </bean>

</beans>
    ]]></script>
  </div>
</div>

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-file"></i> UsingNamedJdbcTemplateTests.java</h4>
    <script type="syntaxhighlighter" class="brush: java"><![CDATA[
package com.makingdevs.practica2;

import static org.junit.Assert.assertEquals;

import java.util.Collections;
import java.util.HashMap;
import java.util.Map;

import org.junit.FixMethodOrder;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.MethodSorters;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.jdbc.core.namedparam.SqlParameterSource;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations={"NamedJdbcTemplateAppCtx.xml","../practica1/DataSourceWithNamespace.xml"})
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class UsingNamedJdbcTemplateTests {

  @Autowired
  NamedParameterJdbcTemplate jdbcTemplate;

  @Test
  public void test1CountWithJdbcTemplate() {
    // Easy way!
    Map<String,Object> namedParameters = new HashMap<String,Object>();
    int rowCount = jdbcTemplate.queryForObject("SELECT count(*) FROM user", namedParameters, Integer.class);
    assertEquals(7, rowCount);
  }

  @Test
  public void test2CountBindingVariableWithJdbcTemplate() {
    String sql = "SELECT count(*) FROM user WHERE username = :username";
    // Using Spring parameters
    SqlParameterSource namedParameters = new MapSqlParameterSource("username", "makingdevs");
    int rowCount = jdbcTemplate.queryForObject(sql, namedParameters, Integer.class);
    assertEquals(1, rowCount);
  }

  @Test
  public void test3QueryStringWithJdbcTemplate() {
    String sql = "SELECT code_name FROM project WHERE id = :id";
    Map<String, String> namedParameters = Collections.singletonMap("id", "4");
    String projectName = jdbcTemplate.queryForObject(sql, namedParameters, String.class);
    assertEquals("AGILE-TASKBOARD", projectName);
  }

}
    ]]></script>
  </div>
</div>


<div class="alert alert-info">
  <strong><i class="icon-terminal"></i></strong> Aunque la clase <code>JdbcTemplate</code> es muy poderosa y podría usarse de forma independiente, te recomendamos ampliamente que la uses con el soporte a DAO's que ofrece Spring.
</div>

## El JdbcTemplate y los RowMappers

## Soporte a DAO's

## Creando DAO's con JdbcDaoSupport

------

### Enfoques de acceso a datos disponibles.

Podemos escoger varios enfoques para el acceso a datos con JDBC con `JdbcTemplate`. Una vez que comenzamos a usarlos, podemos mezclarlos para lograr funcionalidad más específica; estos son:

* `JdbcTemplate` - Es el enfoque más popular, el de nivel más bajo.
* `NamedParameterJdbcTemplate` - Rodea un `JdbcTemplate` para proveer parámetros nombrados en lugar de los marcadores "?". Este enfoque provee de mejro documentación del uso del template cuando tenemos varios parámetros por aplicar.
* `SimpleJdbcInsert` y `SimpleJdbcCall` - Optimizan los metadatos de la base de datos para límitar la cantidad de configuración necesaria. Este enfoque simplifica el código a escribir de tal manera que sólo hay que proveer el nombre de la tabla o procedimiento, y proveer un mapa de parámteros coincidiendo los nombres de las columnas. Esto sólo funciona si la base de datos provee los metadatos adecuados, en caso contrario, tendremos que ponerlos nosotros mismo en configuración.
* Objetos de RDBMS
    * `MappingSqlQuery`
    * `SqlUpdate`
    * `StoredProcedure`
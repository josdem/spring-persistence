# Manejo de transacciones

------

Cuando escribimos a una base de datos, debemos asegurarnos que la integridad de los datos es mantenida cuando se realizan cambios dentro de una transacción.

Las transacciones juegan un rol importante en el desarrollo de software, asegurando que los datos y los recursos nunca se dejan en un estado incosistente.

## Conceptos esenciales

En desarrollo de software tenemos un acrónimo para referirnos a las características del desarrollo a bases de datos relacionales: ACID

* **Atomic** - Las transacciones se componen de una o más actividades empaquetadas en conjunto como una sola unidad de trabajo. La _atomicidad_ asegura que todas las operaciones de la transacción ocurran o que ninguno de ellas ocurran. Si todas las actividades tienen éxito, la operación en global es procesada. Si alguna de las actividades falla, toda la transacción falla y se deshace.
* **Consistent** - Una vez que una transacción termina (con o sin éxito), el sistema se deja en un estado coherente con el negocio que modela. Los datos no deben corromperse con respecto a la realidad.
* **Isolated** - Las transacciones deben permitir que varios usuarios trabajen con los mismos datos, sin el trabajo individual de cada usuario se enrede con los demás. Por lo tanto, las transacciones deben ser aisladas unas de otras, previniendo lecturas concurrentes y escribir sobre los mismos datos que se producen. Hay que tener en cuenta que el aislamiento suele implicar bloqueo a registros y/o las tablas en una base de datos.
* **Durable** - Una vez que la transacción se ha completado, los resultados de la transacción deben ser permanentes para que permanezcan a pesar de cualquier tipo de caída de la aplicación. Normalmente, esto implica almacenar los resultados en una base de datos o alguna otra forma de almacenamiento persistente.

<div class="bs-callout bs-callout-info">
<h4><i class="icon-coffee"></i> Información de utilidad.</h4>
  <p>
    Para profundizar en este tema te recomendamos <a href="http://www.amazon.com/Patterns-Enterprise-Application-Architecture-Martin/dp/0321127420">Patterns of Enterprise Application Architecture de Martin Fowler</a>
  </a>
  </p>
</div>

## Administración de transacciones con Spring

<blockquote>
  <p>El soporte comprensivo de transacciones de Spring es potencialmente la razón más importante de su uso e impacto dentro de la industria del software.</p>
</blockquote>

Beneficios:

* Modelo de programación consistente a través de diferentes API's como JTA, JDBC, Hibernate, JPA y JDO.
* Soporte para administración declarativa de transacciones
* Una API muy simple para la administración programática de transacciones, en comparación de la API de JTA que suele ser muy compleja.
* Excelente integración con las abstracciones de acceso a datos de Spring.

### Ventajas del modelo de soporte transaccional

En Java EE, solo tenemos dos elecciones para la administración de transacciones: _locales_ o _globales_, ambas tienen un conjunto de características que las diferencian y tienen limitantes.

#### Transacciones globales

Habilitan trabajar con múltiples recursos transaccionales, tipicamente bases de datos relaciones y colas de mensajes. Los servidores de aplicaciones administran las transacciones a través de JTA. Siin embargo, una transacción manejada por JTA normalmente viene de una fuente en un JNDI, lo cual significa el uso de JNDI adicional al de JTA.

#### Transacciones locales

Están relacionadas a un recurso específico, como una transacción asociada a una conexión JDBC. Las transacciones locales quizá sean más fácil de usar, pero tienen desventajas significativas: no pueden trabajar a lo largo de múltiples recursos transaccionales. Por ejemplo el código que administra las transacciones usa una conexión JDBC que no puede correr con la API de JTA, debido a que el servidor de aplicaciones no está involucrado en la administración de la transacción.

#### Modelo consistente de programación

Spring resuelve las desventajas de las transacciones globales y locales. Habilita al desarrollador usar un modelo consistente de programación en cualquier entorno. Se escribe el código una vez, y puede beneficiarse de las diferentes estrategias de administración de transacciones en los diferentes ambientes. Y aunque provee enfoques programáticos y declarativos, los desarrolladores se inclinan más a estos útlimos.

Con el modelo programático, los desarrolladores trabajan con la abstracción de transacción de Spring. Con el modelo declarativo, tipicamente se escribe código que no tiene relación alguna con la administración de las transacciones, y no depende de la API de Spring, o de alguna otra API.

<blockquote>
  <p>No necesitamos de un servidor de aplicaciones para el manejo de transacciones.</p>
</blockquote>

La administración de transacciones de Spring soporta cambios a reglas tradicionales como cuando una aplicación Java requiere un servidor de aplicaciones.

<div class="bs-callout bs-callout-warning">
<h4><i class="icon-coffee"></i> Información de utilidad.</h4>
  <p>
    Típicamente necesitas la capacidad de JTA del servidor de aplicaciones solamente si necesitas manejar transacciones a través de múltiples recursos, lo cual NO es un requerimiento en la mayoría de la aplicaciones. Muchas aplicaciones una sola base de datos única y escalable.
  </p>
</div>

<blockquote>
  <p>SpringFramework te da la elección de escalar la aplicación a un servidor de aplicaciones totalmente cargado cuando lo necesites.</p>
</blockquote>

## Comprensión de los Manejadores de transacciones

Spring emplea un mecanismo de _callback_ que abstrae la implementación de la transacción actual del código de la transacción. Y si la aplicación usa un sólo recurso persistente, entonces Spring puede usar el soporte ofrecido por un mecanismo de persistencia. Pero si la aplicación tiene el requerimiento de cubrir varios recursos para una transacción, entonces Spring puede soportar transacciones distribuidas(XA) usando implementaciones de terceros de JTA.

El elemento principal de Spring para la abstracción transaccional es la noción de una estrategia de transacion(_transaction strategy_), la cual está definida por la interfaz `org.springframework.transaction.PlatformTransactionManager`:

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> PlatformTransactionManager.java</h4>
    <script type="syntaxhighlighter" class="brush: java;"><![CDATA[
      public interface PlatformTransactionManager {
        TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
        void commit(TransactionStatus status) throws TransactionException;
        void rollback(TransactionStatus status) throws TransactionException;
      }
    ]]></script>
  </div>
</div>

Las implementaciones de esta interfaz son definidas como cualquier otro bean en el contenedor de IoC de Spring. Además manteniendo la filosofía de Spring, la excepción `TransactionException` puede ser arrojada por cualquier método de la interfaz, y la cual es _no checada_.

El método `getTransaction(...)` regresa un objeto `TransactionStatus`, dependiendo del objeto `TransactionDefinition`. Lo que hace `TransactionStatus` es representar una nueva transacción, o una transacción existente si una transacción coincidente existe en la pila de la llamada actual. La implicación de este último caso es que el `TransactionStatus` esta asociado con un hilo de ejecución.

La interface `TransactionDefinition` específica:

* **Isolation** - El grado el cual esta transacción esta aislada del trabajo de otras transacciones.
* **Propagation** - Típicamente, todo el código se ejecuta dentro de una transacción, sin embargo, tenemos la opción de definir de definir el comportamiento en el evento de que un método transaccional sea ejecutado en la misma transacción o abra una nueva transacción suspendiendo la actual.
* **Timeout** - Cuanto tiempo deberá correr esta transacción antes de se le haga rollback automático por superar dicho tiempo.
* **Read-only** status - Modifica la transacción para asegurar que no alterarán los datos en una operación.

La interface `TransactionStatus` provee de una forma simple para codificar la transacción y controlar el código de ejecución de la transacción, así mismo, buscar el estado de la transacción.

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> TransactionStatus.java</h4>
    <script type="syntaxhighlighter" class="brush: java;"><![CDATA[
public interface TransactionStatus extends SavepointManager {

    boolean isNewTransaction();

    boolean hasSavepoint();

    void setRollbackOnly();

    boolean isRollbackOnly();

    void flush();

    boolean isCompleted();

}
    ]]></script>
  </div>
</div>

### Definición de los manejadores de transacciones

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> DataSourceTransactionManager</h4>
    <script type="syntaxhighlighter" class="brush: xml;"><![CDATA[
      <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}" />
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
      </bean>

      <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
      </bean>
    ]]></script>
  </div>
</div>

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> HibernateTransactionManager</h4>
    <script type="syntaxhighlighter" class="brush: xml;"><![CDATA[
    <bean id="sessionFactory" class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="mappingResources">
          <list>
            <value>com/makingdevs/model/User.hbm.xml</value>
            <!-- ... -->
          </list>
        </property>
        <property name="hibernateProperties">
          <value>
            hibernate.dialect=${hibernate.dialect}
          </value>
        </property>
      </bean>

      <bean id="txManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
      </bean>
    ]]></script>
  </div>
</div>

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> JtaTransactionManager</h4>
    <script type="syntaxhighlighter" class="brush: xml;"><![CDATA[
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/jee
        http://www.springframework.org/schema/jee/spring-jee.xsd">

    <jee:jndi-lookup id="dataSource" jndi-name="jdbc/makingdevs"/>

    <bean id="txManager" class="org.springframework.transaction.jta.JtaTransactionManager" />

</beans>
    ]]></script>
  </div>
</div>

Otros manejadores de transacciones disponibles están en `org.springframework.transaction`:

* `jca.cci.connection.CciLocalTransactionManager`
* `jms.connection.JmsTransactionManager `
* `jms.connection.JmsTransactionManager102`
* `orm.jdo.JdoTransactionManager`
* `orm.jpa.JpaTransactionManager`
* `transaction.jta.OC4JJtaTransactionManager`
* `transaction.jta.WebLogicJtaTransactionManager`
* `transaction.jta.WebSphereUowTransactionManager`

## Programando transacciones

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> TransactionTemplateConfig.java</h4>
    <script type="syntaxhighlighter" class="brush: java;"><![CDATA[
package com.makingdevs.practica8;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ImportResource;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.support.TransactionTemplate;

@Configuration
@ImportResource(value={"classpath:/com/makingdevs/practica1/DataSourceWithNamespace.xml"})
// Look ma!, I'm reusing the DataSource and the JdbcTemplate
@ComponentScan(basePackages={"com.makingdevs.practica4","com.makingdevs.practica8"})
public class TransactionTemplateConfig {
  
  @Autowired
  DataSource dataSource;

  @Bean
  public DataSourceTransactionManager transactionManager(){
    return new DataSourceTransactionManager(dataSource);
  }
  
  // Look ma!, I'm declaring the Transaction Template
  @Bean
  public TransactionTemplate transactionTemplate(){
    return new TransactionTemplate(transactionManager());
  }
}
    ]]></script>
  </div>
</div>

<div class="row">
  <div class="col-md-6">
    <h4><i class="icon-code"></i> TransactionTemplateConfig.java</h4>
    <script type="syntaxhighlighter" class="brush: java;"><![CDATA[
package com.makingdevs.practica8;

import java.io.IOException;
import java.util.Date;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.TransactionCallback;
import org.springframework.transaction.support.TransactionCallbackWithoutResult;
import org.springframework.transaction.support.TransactionTemplate;

import com.makingdevs.dao.UserStoryDao;
import com.makingdevs.model.Project;
import com.makingdevs.model.UserStory;
import com.makingdevs.services.UserStoryService;

@Service
public class UserStoryServiceImpl implements UserStoryService {
  
  @Autowired
  TransactionTemplate transactionTemplate;
  
  @Autowired
  UserStoryDao userStoryDao;

  @Override
  public void createUserStory(final UserStory userStory) {
    transactionTemplate.execute(new TransactionCallbackWithoutResult() {
      @Override
      protected void doInTransactionWithoutResult(TransactionStatus status) {
        userStory.setDateCreated(new Date());
        userStory.setLastUpdated(new Date());
        userStoryDao.create(userStory);
      }
    });
  }

  @Override
  public List<UserStory> findUserStoriesByProject(final String codeName) {
    transactionTemplate.setReadOnly(true);
    List<UserStory> userStories = transactionTemplate.execute(new TransactionCallback<List<UserStory>>() {
      @Override
      public List<UserStory> doInTransaction(TransactionStatus status) {
        Project project = new Project();
        project.setCodeName(codeName);
        project.setId(1L);
        // TODO: Find project by codeName must be implemented...
        return userStoryDao.findAllByProject(project);
      }
    });
    transactionTemplate.setReadOnly(false);
    return userStories;
  }

  @Override
  public boolean isUserStoryDone(Long userStoryId) {
    transactionTemplate.execute(new TransactionCallbackWithoutResult() {
      @Override
      protected void doInTransactionWithoutResult(TransactionStatus status) {
        try {
          throw new IOException("Checked exception");
        } catch (IOException e) {
          e.printStackTrace();
        }
      }
    });
    return false;
  }

  @Override
  public UserStory findUserStoryByIdentifier(Long userStoryId) {
    transactionTemplate.execute(new TransactionCallbackWithoutResult() {
      @Override
      protected void doInTransactionWithoutResult(TransactionStatus status) {
        throw new UnsupportedOperationException("Runtime exception");
      }
    });
    return null;
  }

}
    ]]></script>
  </div>
  <div class="col-md-6">
    <h4><i class="icon-code"></i> ProgrammaticTransactionsTests.java</h4>
    <script type="syntaxhighlighter" class="brush: java;"><![CDATA[
package com.makingdevs.practica8;

import java.util.List;

import org.junit.Assert;
import org.junit.FixMethodOrder;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.MethodSorters;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import com.makingdevs.model.Project;
import com.makingdevs.model.UserStory;
import com.makingdevs.services.UserStoryService;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { TransactionTemplateConfig.class })
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class ProgrammaticTransactionsTests {

  @Autowired
  UserStoryService userStoryService;

  @Test
  public void test1CreateUSWithTx() {
    UserStory us = new UserStory();
    us.setDescription("As an user...I want...Beacuse...");
    us.setEffort(5);
    us.setPriority(3);
    Project p = new Project();
    p.setId(1L);
    us.setProject(p);
    userStoryService.createUserStory(us);
    Assert.assertTrue(us.getId() > 0);
    System.out.println(us.getId());
  }

  @Test
  public void test2FindUSByProjectCodeNameWithTX() {
    List<UserStory> userStories = userStoryService.findUserStoriesByProject("PROJECTNAME");
    Assert.assertTrue(userStories.size() > 0);
  }

  @Test
  public void test3FindCheckedExceptionTX() {
    userStoryService.isUserStoryDone(1L);
  }

  @Test(expected = RuntimeException.class)
  public void test4FindUncheckedExceptionTX() {
    userStoryService.findUserStoryByIdentifier(1L);
  }

}
    ]]></script>
  </div>
</div>

Para poder apreciar mucho mejor el manejo de transacciones, te recomendamos habilites en tu logger algunos paquetes que reflejan el uso del `DataSource`:

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> log4j.properties</h4>
    <script type="syntaxhighlighter" class="brush: plain;"><![CDATA[
log4j.category.org.springframework.beans.factory=INFO
log4j.category.org.springframework.transaction=DEBUG
log4j.category.org.springframework.jdbc=DEBUG
    ]]></script>
  </div>
</div>

Y obtendrás una salida similar a la siguiente:

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> Console output</h4>
    <script type="syntaxhighlighter" class="brush: plain;"><![CDATA[
DataSourceTransactionManager  - Creating new transaction with name [null]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
SimpleDriverDataSource  - Creating new JDBC Driver Connection to [jdbc:h2:mem:dataSource;DB_CLOSE_DELAY=-1]
DataSourceTransactionManager  - Acquired Connection [conn3: url=jdbc:h2:mem:dataSource user=SA] for JDBC transaction
DataSourceTransactionManager  - Switching JDBC Connection [conn3: url=jdbc:h2:mem:dataSource user=SA] to manual commit
JdbcTemplate  - Executing prepared SQL update
JdbcTemplate  - Executing prepared SQL statement [INSERT INTO user_story(DATE_CREATED,DESCRIPTION,EFFORT,LAST_UPDATED,PRIORITY,PROJECT_ID) VALUES(?,?,?,?,?,?);]
JdbcTemplate  - SQL update affected 1 rows
JdbcTemplate  - Executing prepared SQL query
JdbcTemplate  - Executing prepared SQL statement [SELECT id FROM user_story WHERE DESCRIPTION = ? AND PROJECT_ID = ?]
DataSourceTransactionManager  - Initiating transaction commit
DataSourceTransactionManager  - Committing JDBC transaction on Connection [conn3: url=jdbc:h2:mem:dataSource user=SA]
DataSourceTransactionManager  - Releasing JDBC Connection [conn3: url=jdbc:h2:mem:dataSource user=SA] after transaction
DataSourceUtils  - Returning JDBC Connection to DataSource
    ]]></script>
  </div>
</div>

## Transacciones declarativas

![Alt tx](/img/tx.png)

## Transacciones con anotaciones



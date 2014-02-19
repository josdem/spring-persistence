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

<div class="alert alert-info">
  <strong><i class="icon-terminal"></i></strong> Revisa los mapeos de Hibernate con XML, considera que puedes crear la configuración de Hibernate con XML o anotaciones, incluso las de JPA.
</div>

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
    class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
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

<div class="row">
  <div class="col-md-6">
    <h4><i class="icon-code"></i> GenericDaoHibernateImpl.java</h4>
    <script type="syntaxhighlighter" class="brush: java;"><![CDATA[
package com.makingdevs.practica7;

import java.io.Serializable;
import java.lang.reflect.ParameterizedType;
import java.util.List;

import org.hibernate.SessionFactory;
import org.hibernate.criterion.Projections;

import com.makingdevs.dao.GenericDao;

public abstract class GenericDaoHibernateImpl<T, PK extends Serializable> implements GenericDao<T, PK> {

  private SessionFactory sessionFactory;

  private Class<T> type = null;

  public SessionFactory getSessionFactory() {
    return sessionFactory;
  }

  public void setSessionFactory(SessionFactory sessionFactory) {
    this.sessionFactory = sessionFactory;
  }

  @Override
  public void create(T newInstance) {
    sessionFactory.getCurrentSession().save(newInstance);
  }

  @SuppressWarnings("unchecked")
  @Override
  public T read(PK id) {
    return (T) sessionFactory.getCurrentSession().get(getType(), id);
  }

  @Override
  public void update(T transientObject) {
    sessionFactory.getCurrentSession().update(transientObject);
  }

  @Override
  public void delete(T persistentObject) {
    sessionFactory.getCurrentSession().delete(persistentObject);
  }

  @SuppressWarnings("unchecked")
  @Override
  public List<T> findAll() {
    return sessionFactory.getCurrentSession().createCriteria(getType()).list();
  }

  @Override
  public int countAll() {
    return (Integer) sessionFactory.getCurrentSession().createCriteria(getType()).setProjection(Projections.rowCount())
        .uniqueResult();
  }

  @SuppressWarnings("unchecked")
  public Class<T> getType() {
    if (type == null) {
      Class<?> clazz = getClass();
      while (!(clazz.getGenericSuperclass() instanceof ParameterizedType)) {
        clazz = clazz.getSuperclass();
      }
      type = (Class<T>) ((ParameterizedType) clazz.getGenericSuperclass()).getActualTypeArguments()[0];
    }
    return type;
  }

}
    ]]></script>
  </div>
  <div class="col-md-6">
    <h4><i class="icon-code"></i> HibernateAppCtx.xml</h4>
    <script type="syntaxhighlighter" class="brush: xml;"><![CDATA[
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
  xsi:schemaLocation="http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-4.0.xsd
    http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

  <context:component-scan base-package="com.makingdevs.practica7" />

  <jdbc:embedded-database type="H2" id="dataSource">
    <jdbc:script location="classpath:/com/makingdevs/scripts/user.sql" />
    <jdbc:script location="classpath:/com/makingdevs/scripts/project.sql" />
    <jdbc:script location="classpath:/com/makingdevs/scripts/user_story.sql" />
    <jdbc:script location="classpath:/com/makingdevs/scripts/task.sql" />
    <jdbc:script location="classpath:/com/makingdevs/scripts/constraints.sql" />
  </jdbc:embedded-database>

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

  <bean
    class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor" />

  <!-- This is very important, but it's explained later!!! Don't Worry about... -->
  <bean id="transactionManager"
    class="org.springframework.orm.hibernate4.HibernateTransactionManager">
    <property name="sessionFactory" ref="sessionFactory" />
  </bean>

  <aop:config>
    <aop:pointcut id="allMethods"
      expression="execution(public * com.makingdevs.practica7.**.*(..))" />
    <aop:advisor advice-ref="txAdvice" pointcut-ref="allMethods" />
  </aop:config>

  <tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
      <tx:method name="*" />
    </tx:attributes>
  </tx:advice>

</beans>
    ]]></script>
  </div>
</div>

<div class="bs-callout bs-callout-info">
<h4><i class="icon-coffee"></i> Información de utilidad.</h4>
  <p>
    <code>PersistenceExceptionTranslationPostProcessor</code> es un post procesador de beans el cual agrega un advisor a cualquier bean anotado con @Repository, de tal forma que, cualquier excepción de cualqueir plataforma de acceso a datos es cachada y relanzada por una excepción No Checada de Spring.
  </a>
  </p>
</div>

<div class="row">
  <div class="col-md-6">
    <h4><i class="icon-code"></i> ProjectDaoHibernateImpl.java</h4>
    <script type="syntaxhighlighter" class="brush: java;"><![CDATA[
package com.makingdevs.practica7;

import org.hibernate.Query;
import org.hibernate.SessionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import com.makingdevs.dao.ProjectDao;
import com.makingdevs.model.Project;

@Repository
public class ProjectDaoHibernateImpl extends GenericDaoHibernateImpl<Project, Long> implements ProjectDao {

  @Autowired
  public ProjectDaoHibernateImpl(SessionFactory sessionFactory) {
    super.setSessionFactory(sessionFactory);
  }

  @Override
  public Project findByCodename(String codename) {
    Query query = getSessionFactory().getCurrentSession().createQuery("from Project where codeName = ?");
    query.setString(0, codename);
    return (Project) query.uniqueResult();
  }

}
    ]]></script>
  </div>

  <div class="col-md-6">
    <h4><i class="icon-code"></i> ProjectDaoHibernateImplTests.java</h4>
    <script type="syntaxhighlighter" class="brush: java;"><![CDATA[
package com.makingdevs.practica7;

import static org.springframework.util.Assert.notNull;

import java.util.Date;

import javax.sql.DataSource;

import org.junit.FixMethodOrder;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.MethodSorters;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.util.Assert;
import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertNotEquals;
import static org.junit.Assert.assertNull;

import com.makingdevs.dao.ProjectDao;
import com.makingdevs.model.Project;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "HibernateAppCtx.xml" })
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class ProjectDaoHibernateImplTests {

  @Autowired
  ProjectDao projectDao;
  @Autowired
  DataSource dataSource;
  
  private static Long projectId;

  @Test
  public void test0ProjectDao() {
    notNull(projectDao);
    notNull(dataSource);
  }

  @Test
  public void test1CreateProject() {
    Project project = new Project();
    project.setName("New Project");
    project.setCodeName("NEWPROJECT");
    project.setDescription("This is a new project");
    project.setDateCreated(new Date());
    project.setLastUpdated(new Date());
    projectDao.create(project);
    Assert.isTrue(project.getId() > 0);
    projectId = project.getId();
  }
  
  @Test
  public void test2ReadProject(){
    Project project = projectDao.read(projectId);
    Assert.isTrue(project.getId() > 0);
    assertEquals("New Project", project.getName());
    assertEquals("NEWPROJECT", project.getCodeName());
  }
  
  @Test
  public void test3UpdateProject(){
    Project project = projectDao.read(projectId);
    String originalCodeName = project.getCodeName();
    project.setCodeName("PROJECTUPDATED");
    project.setName("Project updated");
    projectDao.update(project);
    Project projectUpdated = projectDao.read(projectId);
    assertNotEquals(originalCodeName, projectUpdated.getCodeName());
  }
  
  @Test 
  public void test4FindProjectByCodeName(){
    Project project = projectDao.findByCodename("PROJECTUPDATED");
    assertEquals("PROJECTUPDATED", project.getCodeName());
  }
  
  @Test 
  public void test5DeleteProject(){
    Project project = projectDao.read(projectId);
    projectDao.delete(project);
    Project projectDeleted = projectDao.read(projectId);
    assertNull(projectDeleted);
  }

}
    ]]></script>
  </div>
</div>

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> HibernateConfiguration.java</h4>
    <script type="syntaxhighlighter" class="brush: java;"><![CDATA[
package com.makingdevs.practica7;

import javax.sql.DataSource;

import org.springframework.context.annotation.Bean;
import org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor;
import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseBuilder;
import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseType;
import org.springframework.orm.hibernate3.HibernateTransactionManager;
import org.springframework.orm.hibernate4.LocalSessionFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;

import com.makingdevs.dao.ProjectDao;

@Configuration
public class HibernateConfiguration {

  @Bean
  public DataSource dataSource() {
    EmbeddedDatabaseBuilder builder = new EmbeddedDatabaseBuilder();
    builder.addScript("classpath:/com/makingdevs/scripts/project.sql");
    builder.addScript("classpath:/com/makingdevs/scripts/user_story.sql");
    builder.addScript("classpath:/com/makingdevs/scripts/task.sql");
    builder.addScript("classpath:/com/makingdevs/scripts/user.sql");
    builder.addScript("classpath:/com/makingdevs/scripts/constraints.sql");
    return builder.setType(EmbeddedDatabaseType.H2).build();
  }

  @Bean
  public LocalSessionFactoryBean sessionFactory() {
    LocalSessionFactoryBean localSessionFactory = new LocalSessionFactoryBean();
    localSessionFactory.setDataSource(dataSource());
    localSessionFactory.setMappingResources("com/makingdevs/model/Project.hbm.xml",
        "com/makingdevs/model/UserStory.hbm.xml", "com/makingdevs/model/Task.hbm.xml",
        "com/makingdevs/model/User.hbm.xml");
    localSessionFactory.getHibernateProperties().put("hibernate.dialect", "org.hibernate.dialect.H2Dialect");
    return localSessionFactory;
  }

  @Bean
  public PersistenceExceptionTranslationPostProcessor exceptionTranslation() {
    return new PersistenceExceptionTranslationPostProcessor();
  }

  @Bean
  public ProjectDao projectDao() {
    return new ProjectDaoHibernateImpl(sessionFactory().getObject());
  }
  
  @Bean
  public HibernateTransactionManager transactionManager(){
    HibernateTransactionManager transactionManager = new HibernateTransactionManager(sessionFactory().getObject());
    return transactionManager;
  }
}
    ]]></script>
  </div>
</div>

<div class="bs-callout bs-callout-info">
<h4><i class="icon-coffee"></i> Información de utilidad.</h4>
  <p>
    Recuerda que cualquier configuración en XML la puedes realizar con anotaciones o con JavaConfig.
  </a>
  </p>
</div>
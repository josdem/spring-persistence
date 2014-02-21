# Refactor de base de datos

------

Un aspecto crítico de una refactor es que conserva la semántica de comportamiento del código. Por ejemplo, hay una refactorización muy simple llamado Cambio de Nombre de Método, tal vez de `getPersons()` a `getPeople()`. Aunque este cambio parece fácil en la superficie, hay que hacer algo más que solo este cambio, también debe cambiar cada invocación de esta operación a lo largo de todo su código de la aplicación para invocar el nombre nuevo. Una vez que haya realizado estos cambios, entonces se puede decir que verdaderamente ha refactorizado su código, ya que todavía funciona de nuevo como antes.

**Es importante entender que cuando se refactoriza **no** se agrega funcionalidad.**

<blockquote>
  <p>La *refactorización de base de datos* es un simple cambio al esquema de base de datos que mejora su diseño, manteniendo su comportamiento e información.</p>
</blockquote>

Una aspecto a considerar es que un refactor de base de datos es conceptualmente más difícil que un refactor de código; el de código sólo debe de mantener el comportamiento semántico y el de base de datos debe mantener el semántico y el informacional.

### ¿Por qué el refactor de base de datos?

* Para arreglar de forma segura bases de datos legadas
* Para soportar el desarrollo que evoluciona el software

### ¿Por qué es difícil el refactor de base de datos?

Los esquemas relacionales de base de datos están potencialmente atadois a una amplia variedad de cosas:

* El código fuente de la aplicación
* Otro código fuente de la aplicación
* La carga de datos del código fuente
* La estracción de datos del código fuente
* Frameworks de persistencia y capas de la aplicación
* Esquema de base de datos
* Scripts de migración de datos
* Código de pruebas
* Documentación - _Aunque esta no nos importa_

### ¿Cómo refactorizas tu base de datos?

Se puede resumir en un proceso de tres pasos:

1. Comienza en tu sandbox de desarrollo
    * Verificar que un refactor de base de datos es necesario
    * Escoger el refactor más apropiado
    * Desaprobar el esquema original
    * Escribir una prueba de unidad
    * Modificar el esquema de base de datos
    * Migrar los datos de origen
    * Actualizar el acceso a programas externos
    * Actualizar los scripts de migración de datos
    * Correr las pruebas de regresión
    * Anunciar el refactor
    * Versionar el trabajo hecho
2. Implementa en los sandboxes de integración(QA)
3. Instalalo en producción

<blockquote>
  <p>Liquibase es una librería independiente open source para el rastreo, administración y aplicación de cambios de base de datos.</p>
</blockquote>

## El archivo changelog

```
jdbc.username=makingdevs
jdbc.password=makingdevs
jdbc.url=jdbc:mysql://localhost:3306/makingdevs
jdbc.driverClass=com.mysql.jdbc.Driver
```

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> LiquibaseAppCtx.xml</h4>
    <script type="syntaxhighlighter" class="brush: xml;"><![CDATA[
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:util="http://www.springframework.org/schema/util"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

  <context:property-placeholder location="com/makingdevs/practica12/jdbc.properties"/>  

  <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
    <property name="url" value="${jdbc.url}" />
    <property name="driverClassName" value="${jdbc.driverClass}" />
  </bean>

  <bean id="liquibase" class="liquibase.integration.spring.SpringLiquibase">
    <property name="dataSource" ref="dataSource" />
    <property name="changeLog" value="classpath:/com/makingdevs/practica12/db-changelog.xml" />
  </bean>

</beans>
    ]]></script>
  </div>
</div>

<div class="row">
  <div class="col-md-6">
    <h4><i class="icon-code"></i> db-changelog.xml</h4>
    <script type="syntaxhighlighter" class="brush: xml;"><![CDATA[
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

</databaseChangeLog>
    ]]></script>
  </div>
  <div class="col-md-6">
    <h4><i class="icon-code"></i> RefactorDatabaseTests.java</h4>
    <script type="syntaxhighlighter" class="brush: java;"><![CDATA[
package com.makingdevs.practica12;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.util.Assert;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "LiquibaseAppCtx.xml" })
public class RefactorDatabaseTests {

  @Autowired
  ApplicationContext applicationContext;
  
  @Test
  public void testRefactor(){
    Assert.notNull(applicationContext);
  }

}
    ]]></script>
  </div>
</div>

<div class="bs-callout bs-callout-info">
<h4><i class="icon-coffee"></i> Información de utilidad</h4>
  <p>
    Observa que en el momento en que se crea y se levanta el ApplicationContext, se ejecutan los cambios en la base de datos.
  </a>
  </p>
</div>

## Conjunto de cambios: El changeset

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> db-changelog.xml</h4>
    <script type="syntaxhighlighter" class="brush: xml;"><![CDATA[
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

  <changeSet id="1" author="makingdevs">
    <createTable tableName="someTable">
      <column name="id" type="int">
        <constraints primaryKey="true" nullable="false" />
      </column>
      <column name="name" type="varchar(50)">
        <constraints nullable="false" />
      </column>
      <column name="active" type="boolean" defaultValueBoolean="true" />
    </createTable>
  </changeSet>

</databaseChangeLog>
    ]]></script>
  </div>
</div>

<div class="bs-callout bs-callout-info">
<h4><i class="icon-coffee"></i> Información de utilidad</h4>
  <p>
    Para los cambios en la base de datos(changesets) podemos utilizar varios formatos: XML, YAML, JSON, SQL, Groovy, Clojure
  </a>
  </p>
</div>

Nosotros vemos muy flexible el uso de Scripts SQL, y para ello existen una seríe de elementos que debes de conocer.

Todos los archivos de script deben comenzaar con el encabezado `--liquibase formatted sql`

Cada changeset en SQL debe de comenzar con un comentario de la forma: `--changeset author:id attribute1:value1 attribute2:value2 [...]`

El comentario es seguido por una o más sentencias SQL, separadas por `;`.

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> project.sql</h4>
    <script type="syntaxhighlighter" class="brush: sql;"><![CDATA[
--liquibase formatted sql

--changeset makingdevs:2
CREATE TABLE IF NOT EXISTS PROJECT(
    ID BIGINT AUTO_INCREMENT PRIMARY KEY,
    CODE_NAME VARCHAR(50) NOT NULL,
    DATE_CREATED TIMESTAMP NOT NULL,
    DESCRIPTION VARCHAR(255) NOT NULL,
    LAST_UPDATED TIMESTAMP NOT NULL,
    NAME VARCHAR(100) NOT NULL
);
--rollback drop table project;

--changeset makingdevs:3
INSERT INTO PROJECT(ID, CODE_NAME, DATE_CREATED, DESCRIPTION, LAST_UPDATED, NAME) VALUES
(1, 'FACTURANOT', TIMESTAMP '2014-02-12 13:31:52.366', 'Desarrollo de la app de Facturacion', TIMESTAMP '2014-02-12 13:31:52.366', 'Modulo de Facturacion'),
(2, 'VIMCHALLENGES', TIMESTAMP '2014-02-12 13:32:27.509', 'Aplicacion para desafiar a tus amigos con VIM', TIMESTAMP '2014-02-12 13:32:27.509', 'The Vim Challenges'),
(3, 'SPRING-WEB', TIMESTAMP '2014-02-12 13:33:17.968', 'Todos los temas de desarrollo web con Spring', TIMESTAMP '2014-02-12 13:33:17.968', 'Desarrollo Web con Spring'),
(4, 'AGILE-TASKBOARD', TIMESTAMP '2014-02-12 13:37:09.803', 'Un tablero de control de proyectos con historias de usuario y tareas', TIMESTAMP '2014-02-12 13:37:09.803', 'My uber taskboard');

--changeset makingdevs:4 dbms:mysql
CREATE TABLE IF NOT EXISTS PROJECT_USER(
    PROJECT_PARTICIPANTS_ID BIGINT,
    USER_ID BIGINT
);
    ]]></script>
  </div>
</div>

Incluyelo en tu archivo **db-changelog.xml** con ayuda del tag `<include file="com/makingdevs/practica12/project.sql"/>`.

<div class="alert alert-info">
  <strong><i class="icon-terminal"></i></strong> Agrega los archivos que corresponden a todo el modelo relacional.
</div>

<div class="row">
  <div class="col-md-12">
    <h4><i class="icon-code"></i> rename-column.sql</h4>
    <script type="syntaxhighlighter" class="brush: sql;"><![CDATA[
--liquibase formatted sql

--changeset makingdevs:18
alter table project add column full_description varchar(255);

--changeset makingdevs:19
update project set full_description=description where id=id and 1=1;

--changeset makingdevs:20
alter table project drop column description;
    ]]></script>
  </div>
</div>


<div class="bs-callout bs-callout-info">
<h4><i class="icon-coffee"></i> Información de utilidad</h4>
  <p>
    Te recomendamos profundizar en elementos como las precondiciones y los rollbacks, pues te serán de utilidad para asegurar la integridad de los cambios.
  </a>
  </p>
</div>
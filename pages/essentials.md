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

<div id="3"></div>

## Manejo de excepciones 


<div id="4"></div>

## Soporte a DAO


<div id="5"></div>

## ¿ActiveRecord?


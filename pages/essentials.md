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
  <div class="col-md-6">
    <h4><i class="icon-code"></i> UsingJDBCForSimpleQuery.java</h4>
    <script type="syntaxhighlighter" class="brush: java"><![CDATA[
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


<div id="2"></div>

## Jerarquía de excepciones


<div id="3"></div>

## Acceso a datos por Templates


<div id="4"></div>

## Soporte a DAO


<div id="5"></div>

## ¿ActiveRecord?


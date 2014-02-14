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
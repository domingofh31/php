# php
PHP: Clases y funciones utiles para desarrollar sitios web.
## Comencemos
La clase permite conexiones a bases de datos en MySQL y SQL Server, para efecto de esta breve documentación supondremos un ejemplo en MySQL. Para inicializarla basta con las siguientes líneas:
```php
$con = new Conexion(array(
    "tipo"       => "mysql",
    "servidor"   => "localhost",
    "bd"         => "Punto_Venta",
    "usuario"    => "root",
    "contrasena" => ""
));
```
## Patrones
En general en cada clase encontraremos este parametro el cual tendra valores por defecto según cada clase, al ser un valor predeterminado no tiene ninguna restricción y es personalizable al adicionarlo como parametro en el método que corresponda.

| **Clase**  | **Método**                       | **Pattern**       |
| :--------- | :------------------------------: | ----------------: |
| *Insert*   | *value*                          | {val}             |
| *Select*   | *where*, *where_and*, *where_or* | {cam} {rel} {val} |
| *Update*   | *set*                            | {val}             |
|            | *where*, *where_and*, *where_or* | {cam} {rel} {val} |
| *Delete*   | *where*, *where_and*, *where_or* | {cam} {rel} {val} |

## Instrucciones  DML (INSERT, SELECT, UPDATE, DELETE)
### INSERT
Contiene una clase en la cual pasaremos el *nombre de la tabla* y los *campos* separados por comas como parámetros del constructor. Esta clase tiene el método *value* en el cual pasaremos el *valor* ligado en secuencia, en el ejemplo *Nombre_Producto* con Galletas Sponch y *Precio* con 18.
```php
$insert = $con->insert("productos", "Nombre_Producto, Precio");
$insert->value("Galletas Sponch");
$insert->value(18);

if ($insert->execute()) {
  echo "Producto agregado con éxito";
}
```
#### Patrones en INSERT
Un ejemplo práctico de patrones al utilizar el método value, es cuando queremos agregar un valor encriptado con AES_ENCRYPT.
```php
$insert = $con->insert("usuarios", "Nombre_Usuario, Contrasena");
$insert->value("domingofh31");
$insert->value("12345", "AES_ENCRYPT({val}, 'SECRET')");

if ($insert->execute()) {
  echo "Usuario agregado con éxito";
}
```

### SELECT
Contiene una clase en la cual pasaremos el *nombre de la tabla* y los *campos* que vamos a seleccionar como parámetros del constructor. Esta clase tiene una variedad de métodos, todos clausulas de la instrucción SELECT, por ejemplo *leftjoin*, *rightjoin*, *innerjoin*, *where*, *where_and*, *where_or*, *groupby*, *orderby* y *limit*.
He aquí una variedad de ejemplos:
```php
$select = $con->select("productos");
$select->where("existencias", "=", 0);

$productos = $select->execute();
```
Veamos un ejemplo gradual con una consulta de ventas:
```php
$select = $con->select("ventas", "Id_Venta, Nombre_Empleado, DATE_FORMAT(Fecha_Hora, '%d/%m/%Y %H:%i')");
$select->innerjoin("ON empleados.Id_Empleado = ventas.Empleado");

$ventas = $select->execute();
```
- Agrupemos ventas por empleado
```php
$select = $con->select("ventas", "Id_Venta, Nombre_Empleado, DATE_FORMAT(Fecha_Hora, '%d/%m/%Y %H:%i')");
$select->innerjoin("ON empleados.Id_Empleado = ventas.Empleado");
$select->groupby("Fecha_Hora");

$ventas = $select->execute();
```
- Ordenemos ventas por fecha y hora
```php
$select = $con->select("ventas", "Id_Venta, Nombre_Empleado, DATE_FORMAT(Fecha_Hora, '%d/%m/%Y %H:%i')");
$select->innerjoin("ON empleados.Id_Empleado = ventas.Empleado");
$select->orderby("Fecha_Hora ASC");

$ventas = $select->execute();
```
- Agreguemos una unión más y consultemos una venta en especifico:
```php
$select = $con->select("ventas", "Id_Venta, Nombre_Empleado, DATE_FORMAT(Fecha_Hora, '%d/%m/%Y %H:%i'), SUM(Precio_Venta) AS Total");
$select->innerjoin("ON empleados.Id_Empleado = ventas.Empleado");
$select->innerjoin("ON detalles_ventas.Venta = ventas.Id_Venta");
$select->where("Id_Venta", "=", 190053241);

$ventas = $select->execute();
```
#### Patrones en SELECT
Para los ejemplos prácticos retomaremos de los de SELECT e INSERT, y de paso vemos los métodos *where_and* y *where_or*.
- Desencriptación de contraseña en una consulta para iniciar sesión.
```php
$select = $con->select("usuarios", "Id_Usuario, Nombre_Usuario");
$select->where("Nombre_Usuario", "=", "domingofh31");
$select->where_and("Contrasena", "=", "12345", "{cam} {rel} AES_DECRYPT({val}, 'SECRET')");

$login = $select->execute();
```
- Selección específica de ventas por empleados usando operador IN.
ATENCIÓN Se recomienda uso de funciones limpiadoras de caracteres para evitar inyecciones SQL.
```php
$select = $con->select("ventas", "Id_Venta, Nombre_Empleado, DATE_FORMAT(Fecha_Hora, '%d/%m/%Y %H:%i')");
$select->innerjoin("ON empleados.Id_Empleado = ventas.Empleado");
$select->where("Id_Empleado", "IN", "220297", "{cam} {rel} ({val}, '220299', '200300')");

$ventas = $select->execute();
```

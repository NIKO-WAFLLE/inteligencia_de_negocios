# topicos  SQLSERVER (T-sql)

```sql
if not Exists (select name from sys.databases where name=N'miniDB')
BEGIN
	create database miniDB
	collate latin1_general_100_CI_AS_SC_UTF8;
end
go
SELECT name from sys.databases

use miniDB
go

-- creacion de tablas

IF object_id ('Clientes', 'u') is not null drop table Clientes;

create table clientes(
	IdCliente int not null,
	Nombre nvarchar(100),
	Edad int,
	Ciudad nvarchar(100),
	Constraint pk_Clientes
	primary key (IdCliente)
);
GO

IF object_id ('Productos', 'u') is not null drop table Productos;

create table Productos(
	Idproducto int primary key,
	Nombre nvarchar(100),
	Categoria nvarchar(200),
	Precio decimal (12,2)
)

GO

/*
+++++++++++++++++insercion de registros en las tablas +++++++++
*/


INSERT INTO clientes
VALUES (1, 'ANA TORRES ', 25, 'CIUDAD DE MEXICO');

INSERT INTO clientes (IdCliente,Nombre,Edad,Ciudad)
VALUES(2,'Luis Perez',34,'Guadalajara');

INSERT INTO clientes (IdCliente, Edad, Nombre,Ciudad)
VALUES(3,29,'soyla vaca', Null);

INSERT INTO clientes (IdCliente,Nombre,Edad)
VALUES(4,'NATACHA', 41);

INSERT INTO clientes(IdCliente, Nombre, Edad,Ciudad)
VALUES  (5,'SOFIA LOPEZ',19,'CHAPULHUACAN'),
		(6,'LAURA HERNANDEZ',38,NULL),
		(7,'VICTOR TRUJILLO', 25, 'ZACULTIPAN');
GO

CREATE OR ALTER PROCEDURE SP_ADD_CUSTUMER
	@Nombre nvarchar(100),
	@edad int, 
	@ciudad nvarchar(100)
as
	BEGIN
		INSERT INTO clientes (Nombre,Edad, Ciudad)
		VALUES (@Nombre,@edad,@ciudad);

	END;
	GO
	EXEC SP_ADD_CUSTUMER 'CARLOS RUIS', 41, 'MONTERRY'
	EXEC SP_ADD_CUSTUMER 'JOSE ANGEL PEREZ', 74, 'SALTE SI PUEDES'

	SELECT TOP 10 Nombre FROM clientes

	SELECT COUNT(*) AS [NumeroClientes]
	FROM clientes

	--- MOSTRAR LOS CLIENTES POR EDAD DE MENOR A MAYOR

	SELECT UPPER (Nombre) AS [cliente], edad, UPPER (Ciudad) AS [CIUDAD]
	FROM clientes
	ORDER BY Edad DESC;

	-- LISTAR LOS CLIENTES QUE VIVEN EN GUADALAJARA

		SELECT UPPER (Nombre) AS [cliente], edad, UPPER (Ciudad) AS [CIUDAD]
	FROM clientes
	WHERE [Ciudad] = 'GUADALAJARA'

	-- listar los clientes con una edad mayor o igual a 30

	SELECT UPPER (Nombre) AS [cliente], edad, UPPER (Ciudad) AS [CIUDAD]
		FROM clientes
		WHERE [Edad] >= 30

-- listar los clientes cuya ciuedad sea null

	SELECT UPPER (Nombre) AS [cliente], edad, UPPER (Ciudad) AS [CIUDAD]
		FROM clientes
		WHERE [Ciudad] is null;

-- Reemplazar en la consulta las ciudades nulas por la palabra desconocidad sin modificar los datos originales

	SELECT UPPER (Nombre) AS [cliente], edad,
	ISNULL(UPPER (Ciudad), 'Desconocido') as [ciudad]
		FROM clientes;

-- Seleccionar los clientes que tenga edad entre 20 y 35 
-- y que vivan en puebla o monterry

	SELECT UPPER (Nombre) AS [cliente], edad, UPPER (Ciudad) AS [CIUDAD]
		FROM clientes
		WHERE Edad between 20 AND 35 AND
		Ciudad IN ('GUADALAJARA', 'CHAPULHUACAN')


		/*

		======================== ACTULIZAR DATOS ====================
		
		*/

	UPDATE clientes
	SET Ciudad = 'Xochitlan'
	where IdCliente = 5;

	select *  from clientes


	UPDATE clientes
	SET Ciudad = 'sin ciudad'
	where Ciudad is null;


	update clientes 
	SET Edad = 30
	WHERE IdCliente between 3 and 6;

	
	UPDATE clientes
	SET Ciudad = 'Metropoli'
	where Ciudad IN ('Guadalajara','CIUDAD DE MEXICO');

	UPDATE clientes
	SET [Nombre] = 'JUAN PEREZ',
		Edad = 35,
		Ciudad = 'CIUDAD GOTICA'
	where IdCliente = 2;

	UPDATE clientes
	SET Nombre = 'CLIENTE PREMUIN'
	WHERE Nombre LIKE 'S%';

	
	UPDATE clientes
	SET Nombre = 'SILVER CUSTUMER'
	WHERE Nombre LIKE '%RR%';

	UPDATE clientes
	SET Edad = (Edad *2)
	WHERE Edad >= 30 and Ciudad = 'Metropoli'

/*
	========== eliminar datos ========
*/

DELETE FROM clientes
WHERE Edad BETWEEN 25 AND 30;

SELECT *  FROM clientes

/*
======== update delete select
*/
GO

CREATE OR ALTER PROC SP_UPDATE_CUSTUMERS

@id int , @nombre nvarchar(100),
@edad int, @ciudad nvarchar(100)

as 
begin 
	update clientes
	set Nombre = @nombre,
		Edad = @edad,
		Ciudad = @ciudad
		where IdCliente = @id;


end;

GO

 SELECT * FROM clientes

EXEC SP_UPDATE_CUSTUMERS 7 , 'BENITO KANO', 24, 'LIMA'

EXEC SP_UPDATE_CUSTUMERS 
@ciudad='martines de la torre',
@edad = 53,
@id = 3,
@nombre = 'ivan'

-- ejercicio completo donde se inserta datos en una tabla principal por encabezado
-- y una tabla detalle utilizando un sp

-- Tabla principal 
GO
create table ventas(
	IdVenta int identity(1,1) primary key,
	FechaVenta DateTime not Null default getdate(),
	cliente NVARCHAR(100) NOT NULL,
	TOTAL DECIMAL(10,2) NULL
);
go
-- tabla detalle 
create table DetalleDeVenta (
	IdDetalle INT IDENTITY (1,1) PRIMARY KEY,
	IDVenta INT NOT NULL,
	Producto Nvarchar(100) NOT NULL,
	Cantidad int not null, 
	precio decimal (10,2) NOT NULL,
	CONSTRAINT pk_DetalleVenta_Venta
	FOREIGN KEY (IdVenta)
	REFERENCES VENTAS(IdVenta)
);


-- CREAR UN TIPO DE TABLA TABLE TYPE

-- ESTE TIPO DE TABLA SERVIRA COMO ESTRUCTURA PARA ENVIAR LOS DETALLES AL SP
GO
CREATE TYPE TipoDetalleVentas AS TABLE(
	Producto NVARCHAR(100),
	Cantidad INT,
	Precio DECIMAL (10,2)

);

-- CREAR el STORE PROCEDURE

-- EL SP INSERTARA EL CABESADO Y LUEGO TODOS LOS DETALLES 
-- UTILISANDO EL TIPO DE TABLA
GO

CREATE OR ALTER PROCEDURE InsertarVentasConDetalle
	@Cliente NVARCHAR(100),
	@Detalles TipoDetalleVentas READONLY
AS 
BEGIN 
	SET NOCOUNT ON;
	
	DECLARE @IDVenta INT;

	BEGIN TRY
		BEGIN TRANSACTION 
		-- INSERTA EN LA TABLA PRINCIPAL   
			INSERT INTO ventas (cliente)
			VALUES(@Cliente);
			-- OBTENER EL ID RECIEN GENERADO

			SET @IDVenta = SCOPE_IDENTITY();

			INSERT INTO DetalleDeVenta(IDVenta,Producto,Cantidad,precio)

			SELECT @IDVenta, Producto, Cantidad , Precio
			FROM @Detalles;

			-- TOTAL VENTA 

			UPDATE ventas 
			SET TOTAL = (SELECT SUM(Cantidad * Precio) FROM @Detalles)
			WHERE IdVenta = @IDVenta;

			COMMIT TRANSACTION; 

	END TRY
		BEGIN CATCH
			ROLLBACK TRANSACTION;
			THROW;
		END CATCH

END;

GO

-- EJECUCUION EL SP CON DATOS DE PRUEBA 

-- DECLARAR UNA VARIable  tipo tabla 
GO
DECLARE @Misdetalles as TipoDetalleVentas
-- insertar producto en el type table 

INSERT INTO @Misdetalles (Producto,Cantidad,precio)
values
('laptop',1,15000),
('Mouse',2,300),
('Teclado',1,500),
('pantalla', 5, 4500)

-- Ejecutar el sp
Exec InsertarVentasConDetalle @cliente='uriel edgar', @Detalles = @Misdetalles

GO 

select * from ventas

select * from DetalleDeVenta
```
## Funciones integradas 

(￣y▽￣)╭ Ohohoho.....

- Funciones de cadena 

| Función                               | Descripción                                      | Ejemplo                                            |
| ------------------------------------- | ------------------------------------------------ | -------------------------------------------------- |
| `LEN(cadena)`                         | Longitud del texto (sin contar espacios finales) | `LEN('SQL Server ') → 10`                          |
| `LTRIM(cadena)`                       | Elimina espacios a la izquierda                  | `'  Hola' → 'Hola'`                                |
| `RTRIM(cadena)`                       | Elimina espacios a la derecha                    | `'Hola  ' → 'Hola'`                                |
| `LOWER(cadena)`                       | Convierte a minúsculas                           | `'HOLA' → 'hola'`                                  |
| `UPPER(cadena)`                       | Convierte a mayúsculas                           | `'hola' → 'HOLA'`                                  |
| `SUBSTRING(cadena, inicio, longitud)` | Extrae una parte del texto                       | `SUBSTRING('SQLServer', 4, 6) → 'Server'`          |
| `LEFT(cadena, n)`                     | Devuelve los primeros *n* caracteres             | `LEFT('SQLServer', 3) → 'SQL'`                     |
| `RIGHT(cadena, n)`                    | Devuelve los últimos *n* caracteres              | `RIGHT('SQLServer', 6) → 'Server'`                 |
| `CHARINDEX(subcadena, cadena)`        | Devuelve la posición de una subcadena            | `CHARINDEX('S', 'SQL Server') → 1`                 |
| `REPLACE(cadena, buscar, reemplazo)`  | Reemplaza texto                                  | `REPLACE('SQL 2022', '2022', '2025') → 'SQL 2025'` |
| `REVERSE(cadena)`                     | Invierte el texto                                | `REVERSE('SQL') → 'LQS'`                           |
| `CONCAT(val1, val2, ...)`             | Une varios valores en una sola cadena            | `CONCAT('Cliente ', Nombre)`                       |
| `CONCAT_WS(sep, val1, val2, ...)`     | Une valores con un separador                     | `CONCAT_WS('-', 'MX', '001') → 'MX-001'`           |

```sql

-- funciones integradas built in fuctions 

SELECT TOP 0
IdCliente,
Nombre as  [Nombre Fuente],
UPPER(Nombre) AS Mayusculas,
LOWER(Nombre) AS Minusculas,
LEN(Nombre) AS Longitud,
SUBSTRING(Nombre,1,3) as prefijo,
LTRIM(Nombre) as [sin espacios a la izquierda],
CONCAT(Nombre,' - ',Edad) as [Nombre Edad],
UPPER(REPLACE(TRIM(Ciudad),'chapulhuacan', 'chapu')) AS [ciudad normalisada]
into Stage_Clientes
from clientes


alter table  Stage_Clientes
add constraint pk_srage_cliente
primary key (idCliente);


---insertar dtaos a partir de una consulta 
go 

Insert INTO Stage_Clientes (IdCliente,
                           [Nombre Fuente],
						   Mayusculas,
						   Minusculas,
						   Longitud,
						   prefijo,
						   [sin espacios a la izquierda],
						   [Nombre Edad],[ciudad normalisada] )
select
IdCliente,
Nombre as  [Nombre Fuente],
UPPER(Nombre) AS Mayusculas,
LOWER(Nombre) AS Minusculas,
LEN(Nombre) AS Longitud,
SUBSTRING(Nombre,1,3) as prefijo,
LTRIM(Nombre) as [sin espacios a la izquierda],
CONCAT(Nombre,' - ',Edad) as [Nombre Edad],
UPPER(REPLACE(TRIM(Ciudad),'chapulhuacan', 'chapu')) AS [ciudad normalisada]
from clientes
go

select * from clientes

```

funciones de fecha 
| Función                               | Descripción                        |
| ------------------------------------- | ---------------------------------- |
| `GETDATE()`                           | Devuelve la fecha y hora actual    |
| `DATEADD(intervalo, cantidad, fecha)` | Suma o resta unidades de tiempo    |
| `DATEDIFF(intervalo, fecha1, fecha2)` | Calcula la diferencia entre fechas |

```sql

---- funcion de fecha

use NORTHWND
go

select 
OrderDate
,GETDATE() as FechaActual -- fecha consulta
,DATEADD(DAY,10, OrderDate) as FechaMas10Dias
,DATEPART(QUARTER,OrderDate) as Trimestre
,DATEPART(MONTH,OrderDate) as MesNumero
,DATENAME(month,OrderDate) as MesNombre
,DATENAME(WEEKDAY,OrderDate) as NombreDia
,DATEDIFF(DAY,OrderDate,GETDATE()) as DiasTranscurridos
,DATEDIFF(YEAR,OrderDate,GETDATE()) as AñosTranscurridos
from Orders;

```
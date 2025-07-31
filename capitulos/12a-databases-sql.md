# Capítulo 12A: Bases de Datos SQL

## 12A.1 Fundamentos del Modelo Relacional

### 12A.1.1 Historia y orígenes

El modelo relacional fue propuesto por Edgar F. Codd en 1970 mientras trabajaba en IBM. Sus principios fundamentales se basaron en la teoría matemática de conjuntos y la lógica de predicados.

**Hitos importantes:**
- 1970: Publicación del artículo "A Relational Model of Data for Large Shared Data Banks"
- 1974: IBM desarrolla System R, primer prototipo de SGBDR
- 1979: Oracle lanza la primera implementación comercial
- 1986: SQL se convierte en estándar ANSI
- 1992: SQL-92 establece el estándar moderno básico

### 12A.1.2 Componentes del modelo relacional

#### Relaciones (Tablas)

Una relación es una tabla bidimensional compuesta por filas y columnas:

- **Tupla (fila)**: Representa una instancia o entidad individual
- **Atributo (columna)**: Representa una propiedad o característica
- **Dominio**: Conjunto de valores posibles para un atributo
- **Grado**: Número de atributos en una relación
- **Cardinalidad**: Número de tuplas en una relación

#### Claves

Las claves son fundamentales para establecer la unicidad e identificación:

- **Clave primaria**: Identifica de forma única cada fila
- **Clave candidata**: Podría servir como clave primaria
- **Clave alternativa**: Clave candidata no seleccionada como primaria
- **Clave foránea**: Establece relaciones entre tablas
- **Clave compuesta**: Formada por múltiples atributos

#### Restricciones

Las restricciones garantizan la integridad de los datos:

- **Integridad de entidad**: Cada tupla debe ser única (clave primaria no nula)
- **Integridad referencial**: Las claves foráneas deben referenciar a valores existentes
- **Integridad de dominio**: Los valores deben cumplir con el tipo de dato y reglas definidas
- **Restricciones de usuario**: Reglas específicas del negocio

### 12A.1.3 Esquema y diagrama ER

#### Esquema relacional

El esquema define la estructura de la base de datos:

```
Usuario(id, nombre, email, fecha_registro)
Producto(id, nombre, precio, categoria_id)
Categoria(id, nombre, descripcion)
```

#### Diagrama Entidad-Relación (ER)

Los diagramas ER visualizan entidades y sus relaciones:

- **Entidad**: Objeto distinguible (representado por rectángulos)
- **Atributo**: Propiedad de una entidad (óvalos)
- **Relación**: Asociación entre entidades (diamantes)
- **Cardinalidad**: 1:1, 1:N, N:M

#### Tipos de relaciones

- **Uno a uno (1:1)**: Un registro en la tabla A se relaciona con exactamente un registro en la tabla B
- **Uno a muchos (1:N)**: Un registro en la tabla A se relaciona con varios registros en la tabla B
- **Muchos a muchos (N:M)**: Múltiples registros en A se relacionan con múltiples registros en B (requiere tabla de unión)

## 12A.2 Lenguaje SQL

SQL (Structured Query Language) es el lenguaje estándar para interactuar con bases de datos relacionales.

### 12A.2.1 Categorías de comandos SQL

#### DDL (Data Definition Language)

Comandos para definir y modificar la estructura:

```sql
-- Crear una tabla
CREATE TABLE usuarios (
    id INTEGER PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    fecha_registro DATE DEFAULT CURRENT_DATE,
    activo BOOLEAN DEFAULT true
);

-- Modificar una tabla
ALTER TABLE usuarios ADD COLUMN telefono VARCHAR(15);

-- Eliminar una tabla
DROP TABLE usuarios;
```

#### DML (Data Manipulation Language)

Comandos para manipular los datos:

```sql
-- Insertar datos
INSERT INTO usuarios (nombre, email) 
VALUES ('Ana López', 'ana@ejemplo.com');

-- Actualizar datos
UPDATE usuarios 
SET activo = false 
WHERE fecha_registro < '2023-01-01';

-- Eliminar datos
DELETE FROM usuarios 
WHERE activo = false;
```

#### DQL (Data Query Language)

Comandos para consultar datos:

```sql
-- Consulta básica
SELECT id, nombre, email 
FROM usuarios 
WHERE activo = true 
ORDER BY nombre;
```

#### DCL (Data Control Language)

Comandos para controlar accesos:

```sql
-- Otorgar permisos
GRANT SELECT, INSERT ON usuarios TO usuario_app;

-- Revocar permisos
REVOKE DELETE ON usuarios FROM usuario_app;
```

#### TCL (Transaction Control Language)

Comandos para control de transacciones:

```sql
BEGIN TRANSACTION;
    -- Operaciones SQL
    UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
    UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;
COMMIT;  -- o ROLLBACK para deshacer
```

### 12A.2.2 Consultas SQL avanzadas

#### Joins (Uniones)

```sql
-- INNER JOIN (intersección)
SELECT u.nombre, p.titulo
FROM usuarios u
INNER JOIN publicaciones p ON u.id = p.usuario_id;

-- LEFT JOIN (todos de la tabla izquierda)
SELECT u.nombre, p.titulo
FROM usuarios u
LEFT JOIN publicaciones p ON u.id = p.usuario_id;

-- RIGHT JOIN (todos de la tabla derecha)
SELECT u.nombre, p.titulo
FROM usuarios u
RIGHT JOIN publicaciones p ON u.id = p.usuario_id;

-- FULL JOIN (todos los registros de ambas tablas)
SELECT u.nombre, p.titulo
FROM usuarios u
FULL JOIN publicaciones p ON u.id = p.usuario_id;
```

#### Subconsultas

```sql
-- Subconsulta en WHERE
SELECT nombre
FROM productos
WHERE precio > (SELECT AVG(precio) FROM productos);

-- Subconsulta en FROM
SELECT dept_name, avg_salary
FROM (
    SELECT d.nombre as dept_name, AVG(e.salario) as avg_salary
    FROM departamentos d
    JOIN empleados e ON d.id = e.departamento_id
    GROUP BY d.nombre
) AS dept_stats
WHERE avg_salary > 50000;

-- Subconsulta correlacionada
SELECT p.nombre
FROM productos p
WHERE p.precio > (
    SELECT AVG(precio)
    FROM productos
    WHERE categoria_id = p.categoria_id
);
```

#### Funciones de agregación

```sql
-- COUNT, SUM, AVG, MIN, MAX
SELECT 
    categoria_id,
    COUNT(*) as total_productos,
    AVG(precio) as precio_promedio,
    MIN(precio) as precio_minimo,
    MAX(precio) as precio_maximo,
    SUM(stock) as stock_total
FROM productos
GROUP BY categoria_id
HAVING COUNT(*) > 5;
```

#### Common Table Expressions (CTE)

```sql
WITH ventas_mensuales AS (
    SELECT 
        EXTRACT(MONTH FROM fecha) as mes,
        SUM(monto) as total
    FROM ventas
    WHERE EXTRACT(YEAR FROM fecha) = 2023
    GROUP BY EXTRACT(MONTH FROM fecha)
)
SELECT mes, total
FROM ventas_mensuales
WHERE total > 10000
ORDER BY mes;
```

#### Window Functions (Funciones de ventana)

```sql
SELECT 
    nombre,
    departamento,
    salario,
    AVG(salario) OVER (PARTITION BY departamento) as salario_promedio_dept,
    salario - AVG(salario) OVER (PARTITION BY departamento) as diferencia,
    RANK() OVER (PARTITION BY departamento ORDER BY salario DESC) as ranking
FROM empleados;
```

### 12A.2.3 Procedimientos almacenados y funciones

#### Procedimientos almacenados

```sql
-- Ejemplo en PostgreSQL
CREATE OR REPLACE PROCEDURE transferir_fondos(
    origen_id INT,
    destino_id INT,
    monto DECIMAL
)
LANGUAGE plpgsql
AS $$
BEGIN
    -- Verificar fondos suficientes
    IF (SELECT saldo FROM cuentas WHERE id = origen_id) >= monto THEN
        -- Restar del origen
        UPDATE cuentas SET saldo = saldo - monto WHERE id = origen_id;
        -- Sumar al destino
        UPDATE cuentas SET saldo = saldo + monto WHERE id = destino_id;
        
        -- Registrar transacción
        INSERT INTO transacciones (origen_id, destino_id, monto, fecha)
        VALUES (origen_id, destino_id, monto, CURRENT_TIMESTAMP);
        
        COMMIT;
    ELSE
        RAISE EXCEPTION 'Fondos insuficientes';
    END IF;
END;
$$;

-- Llamada al procedimiento
CALL transferir_fondos(1, 2, 100.00);
```

#### Funciones

```sql
-- Ejemplo en PostgreSQL
CREATE OR REPLACE FUNCTION calcular_edad(fecha_nacimiento DATE)
RETURNS INTEGER
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN EXTRACT(YEAR FROM AGE(CURRENT_DATE, fecha_nacimiento));
END;
$$;

-- Uso de la función
SELECT nombre, calcular_edad(fecha_nacimiento) as edad
FROM usuarios;
```

### 12A.2.4 Triggers (Disparadores)

```sql
-- Ejemplo en PostgreSQL
CREATE OR REPLACE FUNCTION registrar_cambio_precio()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO historial_precios (producto_id, precio_anterior, precio_nuevo, fecha_cambio)
    VALUES (NEW.id, OLD.precio, NEW.precio, CURRENT_TIMESTAMP);
    
    RETURN NEW;
END;
$$;

CREATE TRIGGER precio_actualizado
AFTER UPDATE OF precio ON productos
FOR EACH ROW
WHEN (OLD.precio IS DISTINCT FROM NEW.precio)
EXECUTE FUNCTION registrar_cambio_precio();
```

## 12A.3 Sistemas de Gestión de Bases de Datos Relacionales (SGBDR)

### 12A.3.1 MySQL

MySQL es uno de los SGBDR más populares, especialmente para aplicaciones web.

**Características principales:**
- Código abierto (aunque hay versión Enterprise)
- Alta velocidad para lecturas
- Amplia adopción en el stack LAMP
- Motores de almacenamiento intercambiables (InnoDB, MyISAM)

**Ejemplo de conexión desde Node.js:**

```javascript
const mysql = require('mysql2/promise');

async function conectarMySQL() {
  const connection = await mysql.createConnection({
    host: 'localhost',
    user: 'usuario',
    password: 'contraseña',
    database: 'mi_db'
  });
  
  const [rows] = await connection.execute('SELECT * FROM usuarios WHERE activo = ?', [true]);
  console.log(rows);
  
  await connection.end();
}
```

### 12A.3.2 PostgreSQL

PostgreSQL es un sistema avanzado de código abierto con características empresariales.

**Características principales:**
- Extensible y altamente personalizable
- Soporte avanzado para tipos de datos (JSON, arrays, geoespacial)
- Funcionalidad robusta para transacciones y concurrencia
- Cumplimiento estricto de estándares SQL

**Ejemplo de conexión desde Python:**

```python
import psycopg2

def conectar_postgres():
    conn = psycopg2.connect(
        host="localhost",
        database="mi_db",
        user="usuario",
        password="contraseña"
    )
    
    cur = conn.cursor()
    cur.execute("SELECT nombre, email FROM usuarios WHERE activo = %s", (True,))
    
    for registro in cur.fetchall():
        print(registro)
        
    cur.close()
    conn.close()
```

### 12A.3.3 SQL Server

Microsoft SQL Server es una solución empresarial con integración al ecosistema Microsoft.

**Características principales:**
- Alto rendimiento para cargas OLTP y OLAP
- Herramientas avanzadas de administración (SSMS)
- Integración con servicios Azure
- Características de seguridad avanzadas

**Ejemplo de conexión desde C#:**

```csharp
using System;
using System.Data.SqlClient;

class Program {
    static void Main() {
        string connectionString = "Server=localhost;Database=mi_db;User Id=usuario;Password=contraseña;";
        
        using (SqlConnection connection = new SqlConnection(connectionString)) {
            connection.Open();
            
            string sql = "SELECT nombre, email FROM usuarios WHERE activo = @activo";
            
            using (SqlCommand command = new SqlCommand(sql, connection)) {
                command.Parameters.AddWithValue("@activo", true);
                
                using (SqlDataReader reader = command.ExecuteReader()) {
                    while (reader.Read()) {
                        Console.WriteLine($"{reader["nombre"]} - {reader["email"]}");
                    }
                }
            }
        }
    }
}
```

### 12A.3.4 SQLite

SQLite es una biblioteca ligera que implementa un motor SQL autocontenido, sin servidor.

**Características principales:**
- Autocontenido, no requiere instalación
- Archivo único para toda la base de datos
- Ideal para aplicaciones móviles y embebidas
- Transacciones ACID completas

**Ejemplo de conexión desde JavaScript (Node.js):**

```javascript
const sqlite3 = require('sqlite3').verbose();

// Abrir base de datos en archivo
const db = new sqlite3.Database('./mi_db.sqlite', (err) => {
  if (err) {
    console.error(err.message);
    return;
  }
  console.log('Conectado a la base de datos SQLite.');
});

// Consulta
db.all("SELECT nombre, email FROM usuarios WHERE activo = ?", [1], (err, rows) => {
  if (err) {
    throw err;
  }
  
  rows.forEach(row => {
    console.log(`${row.nombre}: ${row.email}`);
  });
});

// Cerrar conexión
db.close();
```

## 12A.4 Diseño de Bases de Datos Relacionales

### 12A.4.1 Normalización

La normalización es un proceso para organizar los datos de manera eficiente y eliminar redundancias.

#### Primera Forma Normal (1NF)
- Eliminar grupos repetitivos
- Cada celda contiene un valor atómico
- Cada registro es único

**Antes de 1NF:**
```
| Cliente   | Teléfonos                 |
|-----------|---------------------------|
| Juan      | 555-1234, 555-5678        |
| María     | 555-8765                  |
```

**Después de 1NF:**
```
| Cliente   | Teléfono  |
|-----------|-----------|
| Juan      | 555-1234  |
| Juan      | 555-5678  |
| María     | 555-8765  |
```

#### Segunda Forma Normal (2NF)
- Debe estar en 1NF
- Todos los atributos no clave deben depender de toda la clave primaria

**Antes de 2NF:**
```
| Estudiante | Curso     | Profesor        | Aula   |
|------------|-----------|-----------------|--------|
| Ana        | Física    | Dr. López       | A-101  |
| Pedro      | Física    | Dr. López       | A-101  |
| Ana        | Química   | Dra. Martínez   | B-202  |
```

**Después de 2NF:**
```
Tabla Inscripciones:
| Estudiante | Curso     |
|------------|-----------|
| Ana        | Física    |
| Pedro      | Física    |
| Ana        | Química   |

Tabla Cursos:
| Curso     | Profesor        | Aula   |
|-----------|-----------------|--------|
| Física    | Dr. López       | A-101  |
| Química   | Dra. Martínez   | B-202  |
```

#### Tercera Forma Normal (3NF)
- Debe estar en 2NF
- Los atributos no clave no deben depender transitivamente de la clave primaria

**Antes de 3NF:**
```
| Estudiante | Departamento | Director_Departamento |
|------------|--------------|----------------------|
| Ana        | Ciencias     | Dr. García           |
| Pedro      | Artes        | Dra. Ruiz            |
| María      | Ciencias     | Dr. García           |
```

**Después de 3NF:**
```
Tabla Estudiantes:
| Estudiante | Departamento |
|------------|--------------|
| Ana        | Ciencias     |
| Pedro      | Artes        |
| María      | Ciencias     |

Tabla Departamentos:
| Departamento | Director_Departamento |
|--------------|----------------------|
| Ciencias     | Dr. García           |
| Artes        | Dra. Ruiz            |
```

### 12A.4.2 Desnormalización

En algunos casos, puede ser beneficioso desnormalizar para mejorar el rendimiento:

**Cuándo desnormalizar:**
- Consultas de solo lectura frecuentes
- Reportes y análisis
- Datos que rara vez cambian
- Necesidad de evitar joins complejos

**Ejemplo de desnormalización:**
```sql
-- Tablas normalizadas
SELECT o.id, o.fecha, c.nombre, c.email
FROM ordenes o
JOIN clientes c ON o.cliente_id = c.id;

-- Tabla desnormalizada
SELECT id, fecha, cliente_nombre, cliente_email
FROM ordenes_denormalizadas;
```

### 12A.4.3 Indexación

Los índices son estructuras que mejoran la velocidad de recuperación de datos.

**Tipos de índices:**

1. **Índice de árbol B (B-Tree)**
   - El más común
   - Bueno para búsquedas por igualdad y rangos
   ```sql
   CREATE INDEX idx_usuarios_email ON usuarios(email);
   ```

2. **Índice único**
   - Garantiza valores únicos
   ```sql
   CREATE UNIQUE INDEX idx_productos_codigo ON productos(codigo);
   ```

3. **Índice compuesto**
   - Combina múltiples columnas
   ```sql
   CREATE INDEX idx_ventas_fecha_cliente ON ventas(fecha, cliente_id);
   ```

4. **Índice de texto completo**
   - Para búsquedas textuales
   ```sql
   -- En MySQL
   CREATE FULLTEXT INDEX idx_articulos_contenido ON articulos(contenido);
   
   -- En PostgreSQL
   CREATE INDEX idx_articulos_contenido ON articulos USING GIN (to_tsvector('spanish', contenido));
   ```

**Consideraciones sobre índices:**
- Aceleran las consultas pero ralentizan las inserciones/actualizaciones
- Ocupan espacio adicional
- Deben mantenerse (reconstruirse periódicamente)
- No indexar columnas que rara vez se utilizan en consultas

## 12A.5 Optimización de Rendimiento

### 12A.5.1 Análisis y optimización de consultas

#### EXPLAIN y planes de ejecución

La mayoría de los SGBDR ofrecen herramientas para analizar cómo se ejecutan las consultas:

```sql
-- MySQL/PostgreSQL
EXPLAIN SELECT u.nombre, COUNT(p.id) as num_pedidos
FROM usuarios u
LEFT JOIN pedidos p ON u.id = p.usuario_id
WHERE u.fecha_registro > '2023-01-01'
GROUP BY u.id
HAVING COUNT(p.id) > 5;
```

#### Optimización de consultas comunes

```sql
-- Evitar SELECT *
SELECT * FROM productos;  -- Malo
SELECT id, nombre, precio FROM productos;  -- Bueno

-- Usar límites para conjuntos grandes
SELECT id, titulo FROM articulos ORDER BY fecha DESC LIMIT 10;

-- Evitar funciones en WHERE que impidan usar índices
SELECT * FROM usuarios WHERE YEAR(fecha_registro) = 2023;  -- Malo
SELECT * FROM usuarios WHERE fecha_registro BETWEEN '2023-01-01' AND '2023-12-31';  -- Bueno
```

### 12A.5.2 Estrategias de caché

Los sistemas de caché pueden mejorar drásticamente el rendimiento:

1. **Caché de consultas del SGBDR**
   - MySQL: query_cache_size
   - PostgreSQL: pg_prewarm, pg_buffercache

2. **Caché a nivel de aplicación**
   - Redis, Memcached
   - Caché de resultados de consultas frecuentes

```javascript
// Ejemplo con Redis y Node.js
const redis = require('redis');
const client = redis.createClient();

async function getProducto(id) {
  // Intentar obtener del caché
  const cachedProduct = await client.get(`producto:${id}`);
  
  if (cachedProduct) {
    return JSON.parse(cachedProduct);
  }
  
  // Si no está en caché, obtener de la base de datos
  const producto = await db.query('SELECT * FROM productos WHERE id = ?', [id]);
  
  // Guardar en caché (expira en 1 hora)
  await client.setEx(`producto:${id}`, 3600, JSON.stringify(producto));
  
  return producto;
}
```

### 12A.5.3 Particionamiento de tablas

El particionamiento divide tablas grandes en partes más manejables:

**Tipos de particionamiento:**

1. **Particionamiento horizontal (sharding)**: Divide filas entre particiones
   ```sql
   -- MySQL ejemplo
   CREATE TABLE ventas (
     id INT,
     fecha DATE,
     monto DECIMAL(10,2)
   )
   PARTITION BY RANGE (YEAR(fecha)) (
     PARTITION p2020 VALUES LESS THAN (2021),
     PARTITION p2021 VALUES LESS THAN (2022),
     PARTITION p2022 VALUES LESS THAN (2023),
     PARTITION p2023 VALUES LESS THAN (2024)
   );
   ```

2. **Particionamiento vertical**: Divide columnas entre diferentes tablas
   ```sql
   -- Tabla original
   CREATE TABLE productos (
     id INT PRIMARY KEY,
     nombre VARCHAR(100),
     descripcion TEXT,  -- Podría ser muy grande
     precio DECIMAL(10,2),
     imagenes JSON      -- Podría ser muy grande
   );
   
   -- Después del particionamiento vertical
   CREATE TABLE productos_base (
     id INT PRIMARY KEY,
     nombre VARCHAR(100),
     precio DECIMAL(10,2)
   );
   
   CREATE TABLE productos_detalle (
     id INT PRIMARY KEY,
     descripcion TEXT,
     imagenes JSON,
     FOREIGN KEY (id) REFERENCES productos_base(id)
   );
   ```

## 12A.6 Seguridad en Bases de Datos SQL

### 12A.6.1 Autenticación y autorización

```sql
-- Crear usuario
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'password';

-- Otorgar permisos limitados
GRANT SELECT, INSERT ON basedatos.tabla TO 'app_user'@'localhost';

-- Crear rol (PostgreSQL)
CREATE ROLE app_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_readonly;
GRANT app_readonly TO usuario1, usuario2;
```

### 12A.6.2 Protección contra inyección SQL

La inyección SQL es uno de los ataques más comunes:

```javascript
// Vulnerable a inyección SQL
const query = `SELECT * FROM usuarios WHERE username = '${username}' AND password = '${password}'`;
// Si username = "admin'--", la consulta se convierte en:
// SELECT * FROM usuarios WHERE username = 'admin'--' AND password = 'xyz'

// Solución: Usar consultas parametrizadas
// En Node.js
const query = 'SELECT * FROM usuarios WHERE username = ? AND password = ?';
db.query(query, [username, password]);
```

### 12A.6.3 Cifrado de datos

```sql
-- Cifrado a nivel de columna (ejemplo MySQL)
CREATE TABLE usuarios (
  id INT PRIMARY KEY,
  nombre VARCHAR(100),
  -- Almacenar información sensible cifrada
  tarjeta_credito VARBINARY(255)
);

-- Insertar con cifrado
INSERT INTO usuarios (id, nombre, tarjeta_credito) 
VALUES (1, 'Juan Pérez', AES_ENCRYPT('1234-5678-9012-3456', 'clave_secreta'));

-- Consultar con descifrado
SELECT id, nombre, AES_DECRYPT(tarjeta_credito, 'clave_secreta') as tarjeta
FROM usuarios;
```

## 12A.7 Tendencias Actuales en SQL

### 12A.7.1 JSON en bases de datos relacionales

La mayoría de los SGBDR modernos soportan JSON:

```sql
-- PostgreSQL
CREATE TABLE productos (
  id SERIAL PRIMARY KEY,
  nombre VARCHAR(100),
  atributos JSONB
);

INSERT INTO productos (nombre, atributos)
VALUES ('Smartphone', '{"color": "negro", "memoria": "128GB", "camara": {"trasera": "48MP", "frontal": "12MP"}}');

-- Consulta con JSON
SELECT nombre, atributos->>'color' as color
FROM productos
WHERE atributos->'camara'->>'trasera' = '48MP';
```

### 12A.7.2 Bases de datos relacionales en la nube

Servicios gestionados que simplifican administración:

- Amazon RDS/Aurora
- Google Cloud SQL
- Azure SQL Database
- Herramientas de migración y replicación

### 12A.7.3 NewSQL

Sistemas que combinan las ventajas de SQL y NoSQL:

- **Google Spanner**: Consistencia global, alta disponibilidad
- **CockroachDB**: Inspirado en Spanner, código abierto
- **TiDB**: Compatible con MySQL, escalabilidad horizontal
- **YugabyteDB**: API PostgreSQL con escalabilidad NoSQL

## 12A.8 Recursos y herramientas

### 12A.8.1 Herramientas de administración

- **MySQL Workbench**: Diseño visual, administración, desarrollo
- **pgAdmin**: Herramienta de administración para PostgreSQL
- **DBeaver**: Cliente universal de bases de datos
- **DataGrip**: IDE para bases de datos de JetBrains

### 12A.8.2 ORM (Object-Relational Mapping)

Los ORM facilitan la interacción con bases de datos desde código:

**Sequelize (JavaScript/Node.js)**
```javascript
const { Sequelize, DataTypes } = require('sequelize');
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'mysql'
});

const Usuario = sequelize.define('Usuario', {
  nombre: {
    type: DataTypes.STRING,
    allowNull: false
  },
  email: {
    type: DataTypes.STRING,
    unique: true
  },
  activo: {
    type: DataTypes.BOOLEAN,
    defaultValue: true
  }
});

async function crearUsuario() {
  await Usuario.sync();
  const usuario = await Usuario.create({
    nombre: 'Juan Pérez',
    email: 'juan@ejemplo.com'
  });
  console.log(usuario.toJSON());
}
```

**Hibernate (Java)**
```java
@Entity
@Table(name = "usuarios")
public class Usuario {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String nombre;
    
    @Column(unique = true)
    private String email;
    
    @Column
    private Boolean activo = true;
    
    // Getters y setters
}
```

**SQLAlchemy (Python)**
```python
from sqlalchemy import create_engine, Column, Integer, String, Boolean
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

Base = declarative_base()
engine = create_engine('sqlite:///mi_base.db')
Session = sessionmaker(bind=engine)

class Usuario(Base):
    __tablename__ = 'usuarios'
    
    id = Column(Integer, primary_key=True)
    nombre = Column(String, nullable=False)
    email = Column(String, unique=True)
    activo = Column(Boolean, default=True)

# Crear tablas
Base.metadata.create_all(engine)

# Usar el ORM
session = Session()
nuevo_usuario = Usuario(nombre='Ana García', email='ana@ejemplo.com')
session.add(nuevo_usuario)
session.commit()
```

## 12A.9 Conclusión

Las bases de datos SQL han demostrado una notable resistencia y evolución a lo largo de décadas. Su modelo relacional fundamentado matemáticamente sigue siendo la columna vertebral de la mayoría de los sistemas de información. 

La combinación de estándares SQL bien establecidos, soporte transaccional robusto y la reciente incorporación de capacidades para datos no estructurados las mantiene relevantes incluso en la era de los grandes datos y aplicaciones distribuidas.

A medida que los sistemas de datos continúan evolucionando, las bases de datos SQL se adaptan, adoptando características de otros paradigmas mientras mantienen sus fortalezas fundamentales: integridad, consistencia y un poderoso lenguaje declarativo para consultas y manipulación de datos.

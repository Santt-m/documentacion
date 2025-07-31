# Capítulo 12B-2: Bases de Datos NoSQL - Key-Value y Columnares

## 12B.4 Bases de Datos Clave-Valor

Las bases de datos clave-valor son el tipo más simple de bases de datos NoSQL. Almacenan datos como pares clave-valor donde cada valor está asociado con una clave única.

### 12B.4.1 Conceptos clave

**Características principales:**
- Estructura simple: pares clave-valor
- Acceso extremadamente rápido por clave
- Alta escalabilidad horizontal
- Ideales para cachés y datos volátiles
- Flexibilidad en el tipo de valor almacenado

**Limitaciones comunes:**
- Consultas limitadas (principalmente por clave)
- Sin esquema o estructura para validación de datos
- Limitaciones para datos relacionales complejos

### 12B.4.2 Redis

Redis (Remote Dictionary Server) es una base de datos clave-valor en memoria, conocida por su velocidad y versatilidad.

#### Tipos de datos en Redis

- **Strings**: Secuencias de bytes (textos, números, datos binarios)
- **Lists**: Listas de strings
- **Sets**: Conjuntos no ordenados de strings únicos
- **Sorted Sets**: Conjuntos ordenados por un score
- **Hashes**: Mapas de campo-valor
- **Bit arrays**: Manipulación de bits
- **HyperLogLogs**: Estructuras probabilísticas para conteo de elementos únicos
- **Streams**: Append-only para registros tipo log
- **Geospatial**: Datos con información geoespacial

#### Operaciones básicas

```bash
# Strings
SET usuario:1:nombre "Carlos Pérez"
GET usuario:1:nombre
INCR contador
EXPIRE session:token123 3600  # expira en 1 hora

# Lists
LPUSH notificaciones:user1 "Nueva mensaje de admin"
RPUSH notificaciones:user1 "Actualización disponible"
LRANGE notificaciones:user1 0 -1  # obtener todas

# Sets
SADD tags:articulo1 "programación" "nosql" "redis"
SMEMBERS tags:articulo1
SINTER tags:articulo1 tags:articulo2  # intersección

# Sorted Sets
ZADD puntuaciones 100 "usuario1"
ZADD puntuaciones 85 "usuario2"
ZADD puntuaciones 95 "usuario3"
ZRANGE puntuaciones 0 -1 WITHSCORES  # ordenados ascendente
ZREVRANGE puntuaciones 0 2 WITHSCORES  # top 3

# Hashes
HSET usuario:1 nombre "Carlos" edad 30 ciudad "Madrid"
HGET usuario:1 nombre
HGETALL usuario:1
```

#### Patrones de uso comunes

1. **Caché**:
```javascript
// Pseudocódigo para caché con Redis
function obtenerProducto(id) {
  // Intentar obtener de Redis
  let producto = redis.get(`producto:${id}`);
  
  if (!producto) {
    // No está en caché, obtener de la BD principal
    producto = baseDatos.queryProducto(id);
    
    // Guardar en Redis con expiración
    redis.set(`producto:${id}`, JSON.stringify(producto), 'EX', 3600);
  } else {
    producto = JSON.parse(producto);
  }
  
  return producto;
}
```

2. **Rate Limiting**:
```javascript
function verificarRateLimit(usuario, operacion, limiteMaximo, ventanaSegundos) {
  const clave = `rate:${operacion}:${usuario}`;
  
  // Incrementar contador y establecer expiración si es nuevo
  const contador = redis.incr(clave);
  if (contador === 1) {
    redis.expire(clave, ventanaSegundos);
  }
  
  // Verificar si excede el límite
  return contador <= limiteMaximo;
}

// Uso
if (!verificarRateLimit('usuario123', 'login', 5, 60)) {
  throw new Error('Demasiados intentos de login, inténtelo más tarde');
}
```

3. **Publish/Subscribe**:
```javascript
// Servidor (publisher)
redis.publish('notificaciones', JSON.stringify({
  tipo: 'nuevo_mensaje',
  usuario: 'user123',
  mensaje: 'Hola mundo'
}));

// Cliente (subscriber)
redis.subscribe('notificaciones');
redis.on('message', (channel, message) => {
  const datos = JSON.parse(message);
  console.log(`Nuevo mensaje de ${datos.usuario}: ${datos.mensaje}`);
});
```

#### Persistencia en Redis

Redis ofrece varias opciones de persistencia:

- **RDB** (Redis Database): Snapshots periódicos
- **AOF** (Append-Only File): Registro de operaciones de escritura
- **Híbrido**: Combinación de RDB y AOF

```conf
# Configuración en redis.conf
# RDB
save 900 1    # snapshot si hay al menos 1 cambio en 15 min
save 300 10   # snapshot si hay al menos 10 cambios en 5 min
save 60 10000 # snapshot si hay al menos 10000 cambios en 1 min

# AOF
appendonly yes
appendfsync everysec  # sync cada segundo (otras opciones: always, no)
```

#### Redis desde Node.js

```javascript
const redis = require('redis');
const { promisify } = require('util');

// Crear cliente
const client = redis.createClient({
  host: 'localhost',
  port: 6379,
  password: 'micontraseña'
});

// Promisificar métodos
const getAsync = promisify(client.get).bind(client);
const setAsync = promisify(client.set).bind(client);
const hgetallAsync = promisify(client.hgetall).bind(client);

// Manejar eventos
client.on('error', (err) => console.log('Error en Redis:', err));
client.on('connect', () => console.log('Conectado a Redis'));

async function ejemploRedis() {
  try {
    // Operaciones básicas
    await setAsync('contador', '100');
    const contador = await getAsync('contador');
    console.log('Contador:', contador);
    
    // Hash
    client.hmset('usuario:101', {
      nombre: 'Laura',
      email: 'laura@ejemplo.com',
      visitas: 5
    });
    
    const usuario = await hgetallAsync('usuario:101');
    console.log('Usuario:', usuario);
    
    // Incrementar visitas
    client.hincrby('usuario:101', 'visitas', 1);
    
    // Transacción (multi/exec)
    const multi = client.multi();
    multi.set('clave1', 'valor1');
    multi.set('clave2', 'valor2');
    multi.exec((err, replies) => {
      console.log('Transacción completada:', replies);
    });
    
  } catch (error) {
    console.error('Error:', error);
  } finally {
    client.quit();
  }
}

ejemploRedis();
```

### 12B.4.3 Amazon DynamoDB

DynamoDB es un servicio de base de datos NoSQL clave-valor y documental totalmente gestionado por AWS.

#### Características principales

- Servicio gestionado sin servidor
- Escalado automático
- Rendimiento predecible con capacidad aprovisionada
- Baja latencia constante
- Replicación multi-región
- Capacidades transaccionales
- Integración con el ecosistema AWS

#### Estructura de datos

- **Tablas**: Colecciones de ítems
- **Ítems**: Equivalente a registros o filas
- **Atributos**: Pares nombre-valor dentro de un ítem
- **Clave primaria**:
  - Simple: Solo clave de partición (hash)
  - Compuesta: Clave de partición + clave de ordenación (hash + range)

#### Ejemplo con AWS SDK para JavaScript

```javascript
const AWS = require('aws-sdk');

AWS.config.update({
  region: 'us-west-2',
  accessKeyId: 'TU_ACCESS_KEY',
  secretAccessKey: 'TU_SECRET_KEY'
});

const documentClient = new AWS.DynamoDB.DocumentClient();

// Crear ítem
async function crearUsuario(id, datos) {
  const params = {
    TableName: 'Usuarios',
    Item: {
      usuarioId: id,
      ...datos,
      createdAt: new Date().toISOString()
    }
  };
  
  try {
    await documentClient.put(params).promise();
    console.log(`Usuario ${id} creado exitosamente`);
  } catch (error) {
    console.error('Error al crear usuario:', error);
  }
}

// Leer ítem
async function obtenerUsuario(id) {
  const params = {
    TableName: 'Usuarios',
    Key: {
      usuarioId: id
    }
  };
  
  try {
    const data = await documentClient.get(params).promise();
    return data.Item;
  } catch (error) {
    console.error('Error al obtener usuario:', error);
    return null;
  }
}

// Actualizar ítem
async function actualizarUsuario(id, datos) {
  // Construir expresiones para update
  let UpdateExpression = 'set';
  let ExpressionAttributeValues = {};
  let ExpressionAttributeNames = {};
  
  Object.entries(datos).forEach(([key, value], index) => {
    const attributeName = `#attr${index}`;
    const attributeValue = `:val${index}`;
    
    UpdateExpression += `${index === 0 ? ' ' : ', '}${attributeName} = ${attributeValue}`;
    ExpressionAttributeValues[attributeValue] = value;
    ExpressionAttributeNames[attributeName] = key;
  });
  
  const params = {
    TableName: 'Usuarios',
    Key: {
      usuarioId: id
    },
    UpdateExpression,
    ExpressionAttributeValues,
    ExpressionAttributeNames,
    ReturnValues: 'UPDATED_NEW'
  };
  
  try {
    const data = await documentClient.update(params).promise();
    console.log('Usuario actualizado:', data.Attributes);
    return data.Attributes;
  } catch (error) {
    console.error('Error al actualizar usuario:', error);
    return null;
  }
}

// Eliminar ítem
async function eliminarUsuario(id) {
  const params = {
    TableName: 'Usuarios',
    Key: {
      usuarioId: id
    }
  };
  
  try {
    await documentClient.delete(params).promise();
    console.log(`Usuario ${id} eliminado exitosamente`);
    return true;
  } catch (error) {
    console.error('Error al eliminar usuario:', error);
    return false;
  }
}

// Consultar items (query)
async function buscarUsuariosPorCiudad(ciudad) {
  const params = {
    TableName: 'Usuarios',
    IndexName: 'ciudad-index',  // Índice secundario global
    KeyConditionExpression: 'ciudad = :c',
    ExpressionAttributeValues: {
      ':c': ciudad
    }
  };
  
  try {
    const data = await documentClient.query(params).promise();
    return data.Items;
  } catch (error) {
    console.error('Error en la consulta:', error);
    return [];
  }
}

// Escanear tabla
async function buscarUsuariosPorEdad(edadMinima) {
  const params = {
    TableName: 'Usuarios',
    FilterExpression: 'edad >= :e',
    ExpressionAttributeValues: {
      ':e': edadMinima
    }
  };
  
  try {
    const data = await documentClient.scan(params).promise();
    return data.Items;
  } catch (error) {
    console.error('Error en el escaneo:', error);
    return [];
  }
}
```

#### Índices secundarios

DynamoDB soporta dos tipos de índices secundarios:

1. **Índices Secundarios Locales (LSI)**:
   - Misma clave de partición que la tabla base
   - Diferente clave de ordenación
   - Se crean al momento de crear la tabla

2. **Índices Secundarios Globales (GSI)**:
   - Pueden tener una clave de partición diferente
   - Se pueden crear o eliminar en cualquier momento
   - Tienen su propia capacidad de lectura/escritura

```javascript
// Definición de tabla con índices
const params = {
  TableName: 'Usuarios',
  KeySchema: [
    { AttributeName: 'usuarioId', KeyType: 'HASH' }  // Clave de partición
  ],
  AttributeDefinitions: [
    { AttributeName: 'usuarioId', AttributeType: 'S' },
    { AttributeName: 'email', AttributeType: 'S' },
    { AttributeName: 'ciudad', AttributeType: 'S' },
    { AttributeName: 'fechaRegistro', AttributeType: 'S' }
  ],
  GlobalSecondaryIndexes: [
    {
      IndexName: 'email-index',
      KeySchema: [
        { AttributeName: 'email', KeyType: 'HASH' }
      ],
      Projection: {
        ProjectionType: 'ALL'
      },
      ProvisionedThroughput: {
        ReadCapacityUnits: 5,
        WriteCapacityUnits: 5
      }
    },
    {
      IndexName: 'ciudad-fecha-index',
      KeySchema: [
        { AttributeName: 'ciudad', KeyType: 'HASH' },
        { AttributeName: 'fechaRegistro', KeyType: 'RANGE' }
      ],
      Projection: {
        ProjectionType: 'INCLUDE',
        NonKeyAttributes: ['nombre', 'edad']
      },
      ProvisionedThroughput: {
        ReadCapacityUnits: 5,
        WriteCapacityUnits: 5
      }
    }
  ],
  ProvisionedThroughput: {
    ReadCapacityUnits: 10,
    WriteCapacityUnits: 10
  }
};
```

## 12B.5 Bases de Datos Columnares

Las bases de datos columnares almacenan datos por columnas en lugar de por filas, lo que permite un acceso eficiente a grandes volúmenes de datos para operaciones analíticas.

### 12B.5.1 Conceptos clave

**Características principales:**
- Almacenamiento por columnas (vs. por filas)
- Alta compresión de datos
- Rendimiento optimizado para operaciones analíticas
- Escalabilidad horizontal masiva
- Ideales para big data y data warehousing

**Terminología común:**
- **Column Family**: Grupo lógico de columnas relacionadas
- **Row Key**: Identificador único de una fila
- **Column**: Par nombre-valor dentro de una column family
- **Timestamp**: Valor temporal para versionado

### 12B.5.2 Apache Cassandra

Cassandra es una base de datos distribuida diseñada para manejar grandes cantidades de datos a través de múltiples servidores, proporcionando alta disponibilidad sin punto único de fallo.

#### Modelo de datos

- **Keyspace**: Equivalente a una base de datos en RDBMS
- **Table/Column Family**: Grupo de filas y columnas
- **Row**: Identificada por una clave primaria
- **Column**: Unidad básica de almacenamiento con nombre, valor y timestamp

#### CQL (Cassandra Query Language)

```sql
-- Crear keyspace
CREATE KEYSPACE mi_aplicacion
WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 3
};

-- Usar keyspace
USE mi_aplicacion;

-- Crear tabla
CREATE TABLE usuarios (
  id UUID PRIMARY KEY,
  nombre TEXT,
  email TEXT,
  fecha_registro TIMESTAMP,
  activo BOOLEAN,
  ciudad TEXT,
  intereses SET<TEXT>
);

-- Crear índice secundario
CREATE INDEX ON usuarios (email);

-- Insertar datos
INSERT INTO usuarios (id, nombre, email, fecha_registro, activo, ciudad, intereses)
VALUES (uuid(), 'Ana Martínez', 'ana@ejemplo.com', toTimestamp(now()), true, 'Barcelona', {'música', 'viajes'});

-- Consultar datos
SELECT * FROM usuarios WHERE email = 'ana@ejemplo.com';

-- Actualizar datos
UPDATE usuarios
SET ciudad = 'Madrid', intereses = intereses + {'deportes'}
WHERE id = 123e4567-e89b-12d3-a456-426614174000;

-- Eliminar datos
DELETE FROM usuarios WHERE id = 123e4567-e89b-12d3-a456-426614174000;
```

#### Modelado de datos en Cassandra

El modelado de datos en Cassandra se basa en las consultas que se realizarán, no en normalizar los datos:

**Principios:**
1. **Desnormalización**: Duplicar datos es aceptable para optimizar lecturas
2. **Consulta-primero**: Diseñar tablas para consultas específicas
3. **Sin joins**: Todas las consultas deben poder resolverse con una sola tabla

**Ejemplo: Modelado para diferentes patrones de acceso:**

```sql
-- Tabla para buscar usuarios por ID
CREATE TABLE usuarios_por_id (
  usuario_id UUID PRIMARY KEY,
  nombre TEXT,
  email TEXT,
  ciudad TEXT
);

-- Tabla para buscar usuarios por email (patrón de acceso diferente)
CREATE TABLE usuarios_por_email (
  email TEXT PRIMARY KEY,
  usuario_id UUID,
  nombre TEXT,
  ciudad TEXT
);

-- Tabla para buscar usuarios por ciudad (con agrupación)
CREATE TABLE usuarios_por_ciudad (
  ciudad TEXT,
  usuario_id UUID,
  nombre TEXT,
  email TEXT,
  PRIMARY KEY ((ciudad), usuario_id)
);
```

#### Ejemplo con Node.js y driver de Cassandra

```javascript
const cassandra = require('cassandra-driver');

const client = new cassandra.Client({
  contactPoints: ['localhost'],
  localDataCenter: 'datacenter1',
  keyspace: 'mi_aplicacion'
});

// Conectar
client.connect()
  .then(() => console.log('Conectado a Cassandra'))
  .catch(err => console.error('Error de conexión:', err));

// Insertar usuario
async function crearUsuario(datos) {
  const query = 'INSERT INTO usuarios (id, nombre, email, fecha_registro, activo, ciudad, intereses) VALUES (?, ?, ?, ?, ?, ?, ?)';
  const params = [
    cassandra.types.Uuid.random(),
    datos.nombre,
    datos.email,
    new Date(),
    true,
    datos.ciudad,
    datos.intereses
  ];
  
  try {
    const result = await client.execute(query, params, { prepare: true });
    console.log('Usuario creado');
    return result;
  } catch (err) {
    console.error('Error al crear usuario:', err);
    throw err;
  }
}

// Buscar usuario por email
async function buscarPorEmail(email) {
  const query = 'SELECT * FROM usuarios WHERE email = ? LIMIT 1';
  
  try {
    const result = await client.execute(query, [email], { prepare: true });
    return result.rows[0];
  } catch (err) {
    console.error('Error al buscar usuario:', err);
    throw err;
  }
}

// Listar usuarios por ciudad con paginación
async function listarPorCiudad(ciudad, pageSize = 10, pageState = null) {
  const query = 'SELECT * FROM usuarios_por_ciudad WHERE ciudad = ?';
  const options = {
    prepare: true,
    fetchSize: pageSize
  };
  
  if (pageState) {
    options.pageState = pageState;
  }
  
  try {
    const result = await client.execute(query, [ciudad], options);
    return {
      usuarios: result.rows,
      nextPageState: result.pageState
    };
  } catch (err) {
    console.error('Error al listar usuarios:', err);
    throw err;
  }
}

// Actualizar usuario
async function actualizarUsuario(id, datos) {
  let query = 'UPDATE usuarios SET ';
  const params = [];
  
  Object.entries(datos).forEach(([key, value], index) => {
    if (index > 0) query += ', ';
    query += `${key} = ?`;
    params.push(value);
  });
  
  query += ' WHERE id = ?';
  params.push(id);
  
  try {
    const result = await client.execute(query, params, { prepare: true });
    console.log('Usuario actualizado');
    return result;
  } catch (err) {
    console.error('Error al actualizar usuario:', err);
    throw err;
  }
}

// Eliminar usuario
async function eliminarUsuario(id) {
  const query = 'DELETE FROM usuarios WHERE id = ?';
  
  try {
    const result = await client.execute(query, [id], { prepare: true });
    console.log('Usuario eliminado');
    return result;
  } catch (err) {
    console.error('Error al eliminar usuario:', err);
    throw err;
  }
}

// Batch operations
async function batchOps() {
  const queries = [
    {
      query: 'INSERT INTO usuarios_por_id (usuario_id, nombre, email) VALUES (?, ?, ?)',
      params: [cassandra.types.Uuid.random(), 'Nuevo Usuario', 'nuevo@ejemplo.com']
    },
    {
      query: 'UPDATE contadores SET total = total + 1 WHERE tipo = ?',
      params: ['registros']
    }
  ];
  
  try {
    await client.batch(queries, { prepare: true });
    console.log('Batch completado');
  } catch (err) {
    console.error('Error en batch:', err);
    throw err;
  }
}

// Cerrar conexión al finalizar
process.on('SIGINT', () => {
  client.shutdown()
    .then(() => console.log('Conexión cerrada'))
    .catch(err => console.error('Error al cerrar:', err))
    .finally(() => process.exit(0));
});
```

### 12B.5.3 HBase

Apache HBase es una base de datos columnar distribuida basada en el paper BigTable de Google, construida sobre Hadoop.

#### Características principales

- Almacenamiento masivamente escalable para datos estructurados
- Optimizado para acceso de baja latencia
- Alta consistencia
- Parte del ecosistema Hadoop
- Diseñado para tablas extremadamente grandes (billones de filas × millones de columnas)

#### Modelo de datos

- **Table**: Colección de filas
- **Row**: Identificada por una row key
- **Column Family**: Grupo de columnas
- **Column Qualifier**: Nombre de columna dentro de una Column Family
- **Cell**: Intersección de row y column, con versiones por timestamp

#### Ejemplo de shell HBase

```bash
# Crear tabla
create 'usuarios', {NAME => 'info'}, {NAME => 'contacto'}

# Insertar datos
put 'usuarios', 'row1', 'info:nombre', 'Juan Pérez'
put 'usuarios', 'row1', 'info:edad', '30'
put 'usuarios', 'row1', 'contacto:email', 'juan@ejemplo.com'

# Obtener una fila
get 'usuarios', 'row1'

# Escanear tabla
scan 'usuarios'

# Escanear con filtros
scan 'usuarios', {FILTER => "ValueFilter(=, 'binary:Juan')"}

# Eliminar valor
delete 'usuarios', 'row1', 'contacto:email'

# Eliminar fila
deleteall 'usuarios', 'row1'

# Deshabilitar y eliminar tabla
disable 'usuarios'
drop 'usuarios'
```

#### Acceso desde Java

```java
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

public class HBaseExample {
    public static void main(String[] args) {
        // Configuración
        Configuration config = HBaseConfiguration.create();
        config.set("hbase.zookeeper.quorum", "localhost");
        config.set("hbase.zookeeper.property.clientPort", "2181");
        
        try (Connection connection = ConnectionFactory.createConnection(config)) {
            // Referencia a la tabla
            TableName tableName = TableName.valueOf("usuarios");
            Table table = connection.getTable(tableName);
            
            // Insertar datos
            Put put = new Put(Bytes.toBytes("user123"));
            put.addColumn(
                Bytes.toBytes("info"),
                Bytes.toBytes("nombre"),
                Bytes.toBytes("Ana García")
            );
            put.addColumn(
                Bytes.toBytes("contacto"),
                Bytes.toBytes("email"),
                Bytes.toBytes("ana@ejemplo.com")
            );
            table.put(put);
            
            // Obtener datos
            Get get = new Get(Bytes.toBytes("user123"));
            Result result = table.get(get);
            byte[] value = result.getValue(
                Bytes.toBytes("info"),
                Bytes.toBytes("nombre")
            );
            System.out.println("Valor: " + Bytes.toString(value));
            
            // Escanear
            Scan scan = new Scan();
            scan.setStartRow(Bytes.toBytes("user"));
            scan.setStopRow(Bytes.toBytes("user~")); // '~' es mayor que cualquier caracter ASCII
            
            try (ResultScanner scanner = table.getScanner(scan)) {
                for (Result scanResult : scanner) {
                    // Procesar resultados
                    System.out.println("Row: " + Bytes.toString(scanResult.getRow()));
                }
            }
            
            // Eliminar
            Delete delete = new Delete(Bytes.toBytes("user123"));
            table.delete(delete);
            
            // Cerrar recursos
            table.close();
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 12B.6 Conclusión

Las bases de datos clave-valor y columnares representan dos enfoques especializados dentro del ecosistema NoSQL, cada uno con ventajas para casos de uso específicos:

**Bases de datos clave-valor:**
- Ofrecen simplicidad y rendimiento excepcional para operaciones de lectura/escritura por clave
- Ideales para cachés, gestión de sesiones y datos transitorios
- Sistemas como Redis añaden estructuras de datos avanzadas que amplían su versatilidad

**Bases de datos columnares:**
- Optimizadas para operaciones analíticas sobre grandes volúmenes de datos
- Destacan en escenarios de escritura intensiva con lecturas predecibles
- Su modelo de almacenamiento por columnas permite alta compresión y consultas eficientes

Ambos tipos de bases de datos están diseñados para escalar horizontalmente, pero difieren significativamente en su modelo de datos y casos de uso óptimos. La elección entre estos tipos dependerá de los patrones de acceso a datos de la aplicación, los requisitos de consulta y la escala prevista.

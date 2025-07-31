# Capítulo 12: Bases de Datos (Introducción)

## 12.1 Fundamentos de Bases de Datos

### 12.1.1 ¿Qué es una base de datos?

Una base de datos es una colección organizada de información estructurada (datos), típicamente almacenada electrónicamente en un sistema informático. Está diseñada para permitir el acceso, gestión y actualización eficientes de los datos.

**Propósito principal:**
- Almacenar grandes volúmenes de datos
- Mantener la integridad de los datos
- Proporcionar acceso rápido y eficiente
- Permitir manipulación y recuperación de datos
- Garantizar la seguridad y privacidad

### 12.1.2 Evolución histórica

1. **Archivos planos (1950s-1960s)**: Datos almacenados en archivos de texto sin estructuras complejas.
2. **Bases de datos jerárquicas (1960s)**: Organización en estructura de árbol (padre-hijo).
3. **Bases de datos en red (1970s)**: Permitían relaciones más complejas que el modelo jerárquico.
4. **Bases de datos relacionales (1970s-presente)**: Basadas en el modelo relacional de Edgar F. Codd.
5. **Bases de datos orientadas a objetos (1980s-1990s)**: Almacenan objetos completos.
6. **Bases de datos NoSQL (2000s-presente)**: Respuesta a las necesidades de escalabilidad y flexibilidad.
7. **Bases de datos NewSQL (2010s-presente)**: Combinan lo mejor de SQL y NoSQL.

### 12.1.3 Tipos de bases de datos

#### Clasificación por modelo de datos

1. **Relacionales (SQL)**
   - Basadas en tablas con relaciones
   - Ejemplos: MySQL, PostgreSQL, Oracle, SQL Server

2. **NoSQL**
   - **Documentales**: MongoDB, CouchDB
   - **Clave-valor**: Redis, DynamoDB
   - **Columnares**: Cassandra, HBase
   - **Grafos**: Neo4j, Amazon Neptune

3. **Otros modelos**
   - **Series temporales**: InfluxDB, TimescaleDB
   - **Vectoriales**: Pinecone, Milvus
   - **En memoria**: Redis, Memcached

#### Clasificación por uso

1. **Transaccionales (OLTP)**: Optimizadas para operaciones cotidianas
2. **Analíticas (OLAP)**: Optimizadas para análisis complejo
3. **Híbridas (HTAP)**: Combinan capacidades OLTP y OLAP

### 12.1.4 Conceptos fundamentales

#### Estructura de datos básica

- **Campo**: La unidad más pequeña de datos (ej. nombre, email)
- **Registro**: Conjunto de campos relacionados (ej. información de un usuario)
- **Tabla/Colección**: Conjunto de registros con estructura similar
- **Base de datos**: Conjunto organizado de tablas/colecciones

#### Propiedades ACID

Las propiedades ACID garantizan la fiabilidad en el procesamiento de transacciones:

- **Atomicidad**: Una transacción es todo o nada
- **Consistencia**: Los datos siempre pasan de un estado válido a otro
- **Aislamiento**: Las transacciones no interfieren entre sí
- **Durabilidad**: Una vez confirmada, la transacción persiste

#### Propiedades BASE (para sistemas NoSQL)

Un enfoque alternativo para sistemas distribuidos:

- **Básicamente Disponible**: El sistema sigue funcionando incluso con fallos
- **Estado Suave**: No tiene que ser consistente en todo momento
- **Eventual Consistencia**: Llegará a ser consistente en algún momento

### 12.1.5 Diseño de bases de datos

#### Modelado de datos

El proceso de crear un modelo de datos para almacenar información:

1. **Modelado conceptual**: Entidades y relaciones (ER)
2. **Modelado lógico**: Traducir a un modelo específico (relacional, documental)
3. **Modelado físico**: Implementación en un SGBD específico

#### Normalización vs. desnormalización

- **Normalización**: Proceso de organizar datos para reducir redundancia y mejorar integridad
  - 1NF: Eliminar grupos repetitivos
  - 2NF: Eliminar dependencias parciales
  - 3NF: Eliminar dependencias transitivas
  - BCNF, 4NF, 5NF: Formas normales avanzadas

- **Desnormalización**: Proceso inverso que introduce redundancia controlada para mejorar rendimiento

#### Indexación

Estructuras de datos que mejoran la velocidad de las operaciones de recuperación:

- **Índices B-Tree**: Equilibrados, eficientes para rangos y búsquedas exactas
- **Índices Hash**: Rápidos para búsquedas exactas
- **Índices de texto completo**: Optimizados para búsqueda textual
- **Índices espaciales**: Para datos geográficos

### 12.1.6 Sistemas de Gestión de Bases de Datos (SGBD)

Un SGBD es el software que permite crear, mantener y utilizar una base de datos:

**Funciones principales:**
- Definición de datos (DDL)
- Manipulación de datos (DML)
- Control de acceso
- Integridad y consistencia
- Recuperación y respaldo
- Optimización de rendimiento

**Arquitectura típica:**
- Motor de almacenamiento
- Procesador de consultas
- Gestor de transacciones
- Administrador de memoria
- Gestor de seguridad

## 12.2 Próximos capítulos

En las siguientes secciones, profundizaremos en:

- **12A: Bases de datos SQL** - Modelo relacional, SQL, diseño, optimización
- **12B: Bases de datos NoSQL** - Tipos, casos de uso, implementaciones
- **12C: Estrategias avanzadas** - Replicación, sharding, data warehousing

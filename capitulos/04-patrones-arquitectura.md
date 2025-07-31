# Capítulo 4: Patrones y Arquitecturas de Software

La arquitectura de software constituye la columna vertebral de cualquier sistema informático robusto y escalable. Este capítulo explora las principales arquitecturas y patrones que definen el desarrollo de software moderno, ofreciendo una visión integral de cómo organizar, estructurar y diseñar sistemas complejos.

En un entorno tecnológico en constante evolución, donde los requisitos cambian rápidamente y la escala de las aplicaciones crece exponencialmente, elegir la arquitectura correcta se vuelve fundamental para garantizar la mantenibilidad, flexibilidad y rendimiento de los sistemas. A lo largo de este capítulo, analizaremos las ventajas y desventajas de cada enfoque arquitectónico, sus casos de uso ideales, y proporcionaremos ejemplos prácticos de implementación.

## 4.1 Arquitecturas Modernas

Las arquitecturas modernas han evolucionado para responder a las exigencias actuales de los sistemas distribuidos, la computación en la nube y la necesidad de escalar horizontalmente. Estos enfoques arquitectónicos permiten crear sistemas más resilientes, flexibles y adaptables a los cambios, tanto en requisitos de negocio como en volúmenes de carga. A continuación, exploraremos las arquitecturas más relevantes en el panorama actual de desarrollo.

### 4.1.1 Microservicios

La arquitectura de microservicios representa un paradigma revolucionario en el desarrollo de software, alejándose del tradicional enfoque monolítico. Esta arquitectura propone descomponer una aplicación en un conjunto de servicios pequeños, autónomos y especializados, cada uno ejecutándose en su propio proceso y comunicándose mediante mecanismos ligeros, típicamente API REST con formato JSON.

El fundamento de esta arquitectura radica en el principio de responsabilidad única y desacoplamiento, donde cada servicio se centra exclusivamente en resolver un problema específico del dominio de negocio. Este enfoque permite que equipos independientes trabajen en diferentes servicios sin afectar al resto del sistema, facilitando la entrega continua y el despliegue independiente.

#### Características de los Microservicios:

- **Componentes Desplegables Independientemente**: Cada servicio puede ser desarrollado, desplegado y escalado de forma independiente.
- **Organización en torno a Capacidades de Negocio**: Los servicios se dividen según funcionalidades de negocio, no capas técnicas.
- **Productos, no Proyectos**: Un equipo es responsable de un servicio durante todo su ciclo de vida.
- **Endpoints Inteligentes, Tuberías Simples**: Lógica de procesamiento en los servicios, comunicación simple entre ellos.
- **Gobierno Descentralizado**: Equipos autónomos con libertad para elegir tecnologías y herramientas.
- **Gestión de Datos Descentralizada**: Cada servicio tiene su propio almacenamiento de datos.

```
graph TD
    A[Cliente Web] --> B[API Gateway]
    C[Cliente Móvil] --> B
    B --> D[Servicio de Autenticación]
    B --> E[Servicio de Productos]
    B --> F[Servicio de Pedidos]
    B --> G[Servicio de Pagos]
    E --> H[(Base de Datos de Productos)]
    F --> I[(Base de Datos de Pedidos)]
    G --> J[(Base de Datos de Pagos)]
    F --> G
```

#### Ventajas:

- **Escalabilidad Selectiva**: Escalar solo los componentes que lo necesitan.
- **Resistencia a Fallos**: El fallo de un servicio no afecta a toda la aplicación.
- **Agilidad en el Desarrollo**: Equipos pequeños pueden desarrollar y desplegar de forma independiente.
- **Flexibilidad Tecnológica**: Cada servicio puede usar la tecnología más adecuada para su función.

#### Desventajas:

- **Complejidad Operativa**: Gestión de múltiples servicios y sus dependencias.
- **Latencia de Red**: Las llamadas entre servicios pueden introducir retardos.
- **Transacciones Distribuidas**: Dificultad para mantener consistencia entre servicios.
- **Depuración y Trazabilidad**: Seguir el flujo de una solicitud a través de múltiples servicios es complejo.

#### Implementación de un Microservicio en Node.js con Express:

```javascript
// Servicio de Productos
const express = require('express');
const mongoose = require('mongoose');
const app = express();
const PORT = process.env.PORT || 3001;

// Conexión a la base de datos
mongoose.connect('mongodb://localhost:27017/products', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

// Definición del esquema y modelo
const productSchema = new mongoose.Schema({
  name: String,
  description: String,
  price: Number,
  stock: Number,
  category: String
});

const Product = mongoose.model('Product', productSchema);

// Middleware
app.use(express.json());

// Rutas
app.get('/products', async (req, res) => {
  try {
    const products = await Product.find();
    res.json(products);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.get('/products/:id', async (req, res) => {
  try {
    const product = await Product.findById(req.params.id);
    if (!product) return res.status(404).json({ message: 'Producto no encontrado' });
    res.json(product);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.post('/products', async (req, res) => {
  try {
    const product = new Product(req.body);
    await product.save();
    res.status(201).json(product);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// Iniciar el servidor
app.listen(PORT, () => {
  console.log(`Servicio de productos ejecutándose en el puerto ${PORT}`);
});
```

### 4.1.2 Arquitectura Serverless

La arquitectura serverless representa un cambio de paradigma en la forma de desarrollar y desplegar aplicaciones. A pesar de su nombre, "sin servidor" no implica la ausencia total de servidores, sino que abstrae completamente la gestión de la infraestructura del proceso de desarrollo. En este modelo, los desarrolladores se centran exclusivamente en escribir la lógica de negocio, mientras que la plataforma subyacente gestiona automáticamente aspectos como el aprovisionamiento, la escalabilidad y el mantenimiento de los servidores.

Este enfoque está fundamentado en el principio de la computación basada en eventos, donde el código se ejecuta en respuesta a disparadores específicos, como solicitudes HTTP, cambios en bases de datos, carga de archivos o temporizadores programados. Los recursos computacionales se asignan dinámicamente solo durante el tiempo necesario para procesar estos eventos, lo que resulta en un modelo de costos altamente eficiente basado en el uso real y no en capacidad reservada.

#### Características de Serverless:

- **Sin Administración de Servidores**: La infraestructura es gestionada por el proveedor.
- **Facturación Basada en el Uso**: Solo se paga por el tiempo de ejecución real.
- **Escalado Automático**: La plataforma escala automáticamente en función de la demanda.
- **Alta Disponibilidad**: La plataforma maneja la redundancia y la tolerancia a fallos.

```
graph TD
    A[Evento: HTTP Request] --> B[API Gateway]
    B --> C[Función Lambda]
    C --> D[DynamoDB]
    
    E[Evento: Nuevo Archivo] --> F[S3 Bucket]
    F --> G[Función Lambda]
    G --> H[Servicio de Notificaciones]
```

#### Ventajas:

- **Costo Eficiente**: Solo se paga por el tiempo de ejecución real.
- **Escalabilidad**: Escala automáticamente según la demanda.
- **Menor Tiempo de Desarrollo**: Menos preocupación por la infraestructura.
- **Actualizaciones y Parches Automáticos**: Gestionados por el proveedor.

#### Desventajas:

- **Cold Start**: Latencia adicional en la primera ejecución.
- **Limitaciones de Tiempo de Ejecución**: Las funciones suelen tener un tiempo máximo de ejecución.
- **Dependencia del Proveedor**: Posible bloqueo con un proveedor específico.
- **Depuración Compleja**: Entorno más difícil para depurar.

#### Implementación de una Función AWS Lambda con Node.js:

```javascript
// Función Lambda para procesar imágenes subidas a S3
const AWS = require('aws-sdk');
const sharp = require('sharp');

const s3 = new AWS.S3();

exports.handler = async (event) => {
  // Obtener detalles del bucket y la clave del objeto
  const bucket = event.Records[0].s3.bucket.name;
  const key = decodeURIComponent(event.Records[0].s3.object.key.replace(/\+/g, ' '));
  
  // Solo procesar archivos de imagen
  if (!key.match(/\.(jpg|jpeg|png|gif)$/i)) {
    console.log('No es una imagen, omitiendo...');
    return;
  }
  
  try {
    // Descargar la imagen original desde S3
    const originalImage = await s3.getObject({ Bucket: bucket, Key: key }).promise();
    
    // Redimensionar la imagen
    const resizedImage = await sharp(originalImage.Body)
      .resize(300, 300, { fit: 'inside' })
      .toBuffer();
    
    // Nombre para la versión en miniatura
    const thumbnailKey = `thumbnails/${key.split('/').pop()}`;
    
    // Subir la miniatura a S3
    await s3.putObject({
      Bucket: bucket,
      Key: thumbnailKey,
      Body: resizedImage,
      ContentType: originalImage.ContentType
    }).promise();
    
    console.log(`Miniatura creada con éxito: ${thumbnailKey}`);
    
    return {
      statusCode: 200,
      body: JSON.stringify({
        message: 'Procesamiento exitoso',
        thumbnail: thumbnailKey
      })
    };
  } catch (error) {
    console.error('Error al procesar la imagen:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({
        message: 'Error al procesar la imagen',
        error: error.message
      })
    };
  }
};
```

### 4.1.3 Arquitectura de Monolito Modular

La arquitectura de monolito modular representa un equilibrio estratégico entre la simplicidad de los monolitos tradicionales y la flexibilidad de los microservicios. Esta aproximación reconoce que no todos los sistemas requieren la complejidad operativa de una arquitectura distribuida, pero tampoco deben caer en el caos de un monolito sin estructura.

En esencia, un monolito modular es una aplicación única que mantiene una base de código cohesionada, pero organizada internamente en módulos bien definidos con responsabilidades específicas e interfaces claramente delimitadas. Aunque se despliega como una unidad, su diseño interno facilita la comprensión, el mantenimiento y la evolución del sistema, permitiendo que equipos diferentes trabajen en módulos distintos con mínimo acoplamiento.

Esta arquitectura es particularmente valiosa como punto de partida para muchos proyectos o como destino consciente para sistemas donde la complejidad operativa de los microservicios no aporta suficiente beneficio para justificar sus costos.

#### Características:

- **Modularidad Interna**: División clara de responsabilidades en módulos.
- **Despliegue Único**: Todo el código se implementa como una unidad.
- **Fronteras Bien Definidas**: Interfaces claras entre módulos.
- **Cohesión Alta, Acoplamiento Bajo**: Cada módulo encapsula su funcionalidad.

```
graph TD
    A[Cliente] --> B[Monolito Modular]
    B --> C[Módulo de Autenticación]
    B --> D[Módulo de Productos]
    B --> E[Módulo de Pedidos]
    B --> F[Módulo de Pagos]
    B --> G[Base de Datos Compartida]
```

#### Ventajas:

- **Simplicidad Operativa**: Un solo sistema para desplegar y monitorear.
- **Menor Latencia**: Comunicación interna sin latencia de red.
- **Transacciones Simplificadas**: Mayor facilidad para mantener la consistencia de datos.
- **Más Simple que Microservicios**: Menor sobrecarga arquitectónica.

#### Desventajas:

- **Escalado en Bloque**: No se pueden escalar módulos individualmente.
- **Tecnología Compartida**: Todos los módulos usan la misma pila tecnológica.
- **Base de Código Grande**: Potencial complejidad a medida que crece.

#### Implementación de un Monolito Modular en Node.js:

```javascript
// app.js - Punto de entrada de la aplicación
const express = require('express');
const mongoose = require('mongoose');
const authRoutes = require('./modules/auth/routes');
const productRoutes = require('./modules/products/routes');
const orderRoutes = require('./modules/orders/routes');
const paymentRoutes = require('./modules/payments/routes');

const app = express();
const PORT = process.env.PORT || 3000;

// Conexión a la base de datos
mongoose.connect('mongodb://localhost:27017/ecommerce', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

// Middleware
app.use(express.json());

// Rutas modulares
app.use('/auth', authRoutes);
app.use('/products', productRoutes);
app.use('/orders', orderRoutes);
app.use('/payments', paymentRoutes);

// Iniciar el servidor
app.listen(PORT, () => {
  console.log(`Servidor ejecutándose en el puerto ${PORT}`);
});
```

```javascript
// modules/products/routes.js
const express = require('express');
const router = express.Router();
const productController = require('./controllers');
const { authenticate } = require('../auth/middleware');

router.get('/', productController.getAllProducts);
router.get('/:id', productController.getProductById);
router.post('/', authenticate, productController.createProduct);
router.put('/:id', authenticate, productController.updateProduct);
router.delete('/:id', authenticate, productController.deleteProduct);

module.exports = router;
```

```javascript
// modules/products/controllers.js
const Product = require('./model');

exports.getAllProducts = async (req, res) => {
  try {
    const products = await Product.find();
    res.json(products);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

exports.getProductById = async (req, res) => {
  try {
    const product = await Product.findById(req.params.id);
    if (!product) return res.status(404).json({ message: 'Producto no encontrado' });
    res.json(product);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

exports.createProduct = async (req, res) => {
  try {
    const product = new Product(req.body);
    await product.save();
    res.status(201).json(product);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
};

exports.updateProduct = async (req, res) => {
  try {
    const product = await Product.findByIdAndUpdate(req.params.id, req.body, { new: true });
    if (!product) return res.status(404).json({ message: 'Producto no encontrado' });
    res.json(product);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

exports.deleteProduct = async (req, res) => {
  try {
    const product = await Product.findByIdAndDelete(req.params.id);
    if (!product) return res.status(404).json({ message: 'Producto no encontrado' });
    res.json({ message: 'Producto eliminado con éxito' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

### 4.1.4 Arquitectura Hexagonal (Ports and Adapters)

La Arquitectura Hexagonal, también conocida como Ports and Adapters, representa un enfoque revolucionario en el diseño de software propuesto por Alistair Cockburn. Este patrón arquitectónico busca crear sistemas altamente mantenibles y adaptables, donde la lógica de negocio central permanece completamente aislada de los detalles técnicos y las influencias externas.

El concepto fundamental detrás de esta arquitectura es que una aplicación debe poder ser controlada por cualquier agente externo (usuarios, programas, pruebas automatizadas o scripts) y debe poder interactuar con cualquier servicio externo (bases de datos, APIs, sistemas de archivos), sin que estos agentes o servicios externos afecten al núcleo de la aplicación. Esto se logra mediante una estricta separación de responsabilidades y la introducción de abstracciones en forma de puertos e implementaciones concretas en forma de adaptadores.

La metáfora del hexágono (aunque el número de lados es irrelevante) ilustra cómo la aplicación se comunica con el mundo exterior a través de múltiples facetas o puertos, cada uno con adaptadores específicos que traducen entre el lenguaje interno de la aplicación y los protocolos externos.

#### Conceptos Fundamentales

- **Dominio**: El núcleo de la aplicación que contiene la lógica de negocio.
- **Puertos (Ports)**: Interfaces que definen cómo el dominio interactúa con el exterior.
- **Adaptadores (Adapters)**: Implementaciones específicas de los puertos para tecnologías concretas.

```
graph TD
    A[Usuario/UI] --> B[Adaptador Primario]
    B --> C[Puerto Primario]
    C --> D[Dominio/Aplicación]
    D --> E[Puerto Secundario]
    E --> F[Adaptador Secundario]
    F --> G[(Base de Datos/Servicios Externos)]
```

#### Implementación de Arquitectura Hexagonal en TypeScript

```typescript
// Domain - Entidad del dominio
class Product {
  constructor(
    public id: string,
    public name: string,
    public price: number,
    public description: string
  ) {}
  
  increasePrice(amount: number): void {
    this.price += amount;
  }
  
  applyDiscount(percentage: number): void {
    const discount = (this.price * percentage) / 100;
    this.price -= discount;
  }
}

// Port - Puerto secundario (salida)
interface ProductRepository {
  findById(id: string): Promise<Product | null>;
  findAll(): Promise<Product[]>;
  save(product: Product): Promise<void>;
  delete(id: string): Promise<boolean>;
}

// Port - Puerto primario (entrada)
interface ProductService {
  getProductById(id: string): Promise<Product | null>;
  getAllProducts(): Promise<Product[]>;
  createProduct(productData: Omit<Product, 'id'>): Promise<Product>;
  updateProductPrice(id: string, newPrice: number): Promise<Product | null>;
  removeProduct(id: string): Promise<boolean>;
}

// Application - Implementación del servicio (lógica de aplicación)
class ProductServiceImpl implements ProductService {
  constructor(private productRepository: ProductRepository) {}
  
  async getProductById(id: string): Promise<Product | null> {
    return await this.productRepository.findById(id);
  }
  
  async getAllProducts(): Promise<Product[]> {
    return await this.productRepository.findAll();
  }
  
  async createProduct(productData: Omit<Product, 'id'>): Promise<Product> {
    const id = this.generateId(); // Método para generar un ID único
    const newProduct = new Product(
      id,
      productData.name,
      productData.price,
      productData.description
    );
    await this.productRepository.save(newProduct);
    return newProduct;
  }
  
  async updateProductPrice(id: string, newPrice: number): Promise<Product | null> {
    const product = await this.productRepository.findById(id);
    if (!product) return null;
    
    product.price = newPrice;
    await this.productRepository.save(product);
    return product;
  }
  
  async removeProduct(id: string): Promise<boolean> {
    return await this.productRepository.delete(id);
  }
  
  private generateId(): string {
    return Date.now().toString(36) + Math.random().toString(36).substring(2);
  }
}

// Adapter - Adaptador secundario (MongoDB Repository)
class MongoDBProductRepository implements ProductRepository {
  constructor(private readonly mongoClient: any /* Tipo MongoDB */) {}
  
  async findById(id: string): Promise<Product | null> {
    const productData = await this.mongoClient.collection('products').findOne({ id });
    if (!productData) return null;
    
    return new Product(
      productData.id,
      productData.name,
      productData.price,
      productData.description
    );
  }
  
  async findAll(): Promise<Product[]> {
    const productsData = await this.mongoClient.collection('products').find().toArray();
    return productsData.map(data => new Product(
      data.id,
      data.name,
      data.price,
      data.description
    ));
  }
  
  async save(product: Product): Promise<void> {
    await this.mongoClient.collection('products').updateOne(
      { id: product.id },
      { $set: { ...product } },
      { upsert: true }
    );
  }
  
  async delete(id: string): Promise<boolean> {
    const result = await this.mongoClient.collection('products').deleteOne({ id });
    return result.deletedCount > 0;
  }
}

// Adapter - Adaptador primario (Express Controller)
class ProductController {
  constructor(private productService: ProductService) {}
  
  async getProduct(req: any, res: any) {
    try {
      const product = await this.productService.getProductById(req.params.id);
      if (!product) {
        return res.status(404).json({ message: 'Producto no encontrado' });
      }
      return res.json(product);
    } catch (error) {
      console.error('Error al obtener producto:', error);
      return res.status(500).json({ message: 'Error interno del servidor' });
    }
  }
  
  async listProducts(req: any, res: any) {
    try {
      const products = await this.productService.getAllProducts();
      return res.json(products);
    } catch (error) {
      console.error('Error al listar productos:', error);
      return res.status(500).json({ message: 'Error interno del servidor' });
    }
  }
  
  async createProduct(req: any, res: any) {
    try {
      const { name, price, description } = req.body;
      const newProduct = await this.productService.createProduct({
        name,
        price,
        description
      });
      return res.status(201).json(newProduct);
    } catch (error) {
      console.error('Error al crear producto:', error);
      return res.status(500).json({ message: 'Error interno del servidor' });
    }
  }
}

// Configuración y conexión (composición)
const setupApi = async () => {
  // Configuración de la conexión a MongoDB
  const mongoClient = await connectToMongoDB();
  
  // Creación de repositorio
  const productRepository = new MongoDBProductRepository(mongoClient);
  
  // Creación del servicio de aplicación
  const productService = new ProductServiceImpl(productRepository);
  
  // Creación del controlador
  const productController = new ProductController(productService);
  
  // Configuración de rutas Express
  const app = express();
  app.use(express.json());
  
  app.get('/products', (req, res) => productController.listProducts(req, res));
  app.get('/products/:id', (req, res) => productController.getProduct(req, res));
  app.post('/products', (req, res) => productController.createProduct(req, res));
  
  return app;
};
```

#### Relación con Domain-Driven Design (DDD)

La Arquitectura Hexagonal mantiene una relación simbiótica con Domain-Driven Design (DDD), creando una poderosa combinación que maximiza la expresividad del dominio y su aislamiento de las preocupaciones técnicas. Ambos enfoques comparten la filosofía de situar la lógica de negocio en el corazón del sistema y protegerla de influencias externas. Las principales sinergias entre estos enfoques son:

1. **Dominio Aislado**: Ambos buscan un dominio bien definido y aislado de detalles técnicos.
   - En DDD: A través de entidades, objetos de valor y agregados.
   - En Hexagonal: Mediante puertos y adaptadores que protegen el dominio.

2. **Lenguaje Ubicuo**: La separación clara entre dominio e infraestructura facilita el uso del lenguaje del dominio.

3. **Bounded Contexts**: Los límites del contexto en DDD pueden implementarse eficientemente usando la arquitectura hexagonal para definir interfaces claras entre subsistemas.

4. **Servicios de Dominio**: En DDD, los servicios de dominio contienen lógica que no pertenece a entidades específicas, similar a cómo la lógica de aplicación se organiza en la capa de aplicación en Hexagonal.

Ejemplo conceptual de integración DDD-Hexagonal:

```typescript
// Domain Layer (DDD) - Entidad y objeto de valor
class ProductId {
  constructor(public readonly value: string) {
    if (!value) throw new Error('ID del producto es requerido');
  }
}

class Price {
  constructor(public readonly amount: number, public readonly currency: string = 'EUR') {
    if (amount < 0) throw new Error('El precio no puede ser negativo');
  }
}

class Product {
  private constructor(
    public readonly id: ProductId,
    private name: string,
    private price: Price,
    private description: string
  ) {}
  
  
  // Factory method (patrón DDD)
  public static create(name: string, price: number, description: string): Product {
    if (!name || name.trim().length === 0) throw new Error('Nombre de producto inválido');
    if (!description) throw new Error('Descripción del producto requerida');
    
    return new Product(
      new ProductId(Date.now().toString(36)),
      name,
      new Price(price),
      description
    );
  }
  
  // Comportamiento de dominio
  public applyDiscount(percentage: number): Product {
    const discountAmount = (this.price.amount * percentage) / 100;
    const newPrice = new Price(this.price.amount - discountAmount, this.price.currency);
    return new Product(this.id, this.name, newPrice, this.description);
  }
}

// Puertos hexagonales (interfaces secundarias en DDD)
interface ProductRepository {
  findById(id: ProductId): Promise<Product | null>;
  save(product: Product): Promise<void>;
  // ... otros métodos
}

// Servicio de dominio (DDD) + Puerto primario (Hexagonal)
interface ApplyDiscountService {
  applyDiscount(productId: ProductId, discountPercentage: number): Promise<Product | null>;
}

// Implementación del servicio (capa de aplicación hexagonal)
class DiscountService implements ApplyDiscountService {
  constructor(private productRepository: ProductRepository) {}
  
  async applyDiscount(productId: ProductId, discountPercentage: number): Promise<Product | null> {
    const product = await this.productRepository.findById(productId);
    if (!product) return null;
    
    const discountedProduct = product.applyDiscount(discountPercentage);
    await this.productRepository.save(discountedProduct);
    
    return discountedProduct;
  }
}
```

#### Relación con Clean Architecture

La Arquitectura Hexagonal y Clean Architecture, propuesta por Robert C. Martin (Uncle Bob), comparten una profunda afinidad filosófica y conceptual. Ambas surgieron como respuestas al desafío fundamental de crear sistemas de software que sean resistentes al cambio, fáciles de probar y que mantengan la lógica de negocio protegida de los detalles técnicos. Aunque tienen orígenes diferentes, estos enfoques convergen en muchos de sus principios fundamentales, mientras mantienen algunas distinciones conceptuales importantes:

1. **Capas Concéntricas vs Hexágono**: 
   - Clean Architecture define capas concéntricas (Entidades → Casos de Uso → Adaptadores de Interfaz → Frameworks)
   - Hexagonal utiliza el concepto de núcleo con puertos y adaptadores conectados

2. **Regla de Dependencia**: 
   - En Clean Architecture, las dependencias siempre apuntan hacia adentro
   - En Hexagonal, los adaptadores dependen de los puertos (interfaces), no al revés

3. **Terminología**:
   - Clean: Entidades, Casos de Uso, Controladores, Presentadores, Gateways
   - Hexagonal: Dominio, Puertos, Adaptadores Primarios y Secundarios

Comparación visual:

```
// Clean Architecture
┌─────────────────────────────┐
│     Frameworks/Drivers      │
│ ┌─────────────────────────┐ │
│ │  Interface Adapters     │ │
│ │ ┌─────────────────────┐ │ │
│ │ │    Use Cases        │ │ │
│ │ │ ┌───────────────┐   │ │ │
│ │ │ │   Entities    │   │ │ │
│ │ │ └───────────────┘   │ │ │
│ │ └─────────────────────┘ │ │
│ └─────────────────────────┘ │
└─────────────────────────────┘

// Hexagonal Architecture
┌─────────────────────────────┐
│   Adaptador     │  Adaptador │
│   Primario      │ Secundario │
│      │          │     │      │
│      ▼          │     ▼      │
│    Puerto       │   Puerto   │
│   Primario      │ Secundario │
│      │          │     ▲      │
│      ▼          │     │      │
│ ┌─────────────────────────┐ │
│ │      Aplicación /       │ │
│ │        Dominio          │ │
│ └─────────────────────────┘ │
└─────────────────────────────┘
```

Implementación inspirada en Clean Architecture:

```typescript
// 1. Entidades (núcleo del dominio)
class Product {
  constructor(
    public id: string,
    public name: string,
    public price: number,
    public description: string
  ) {}
}

// 2. Caso de Uso (aplicación)
class GetProductUseCase {
  constructor(private productRepository: ProductRepository) {}
  
  async execute(productId: string): Promise<GetProductResponse> {
    try {
      const product = await this.productRepository.getById(productId);
      if (!product) {
        return { success: false, error: 'Producto no encontrado' };
      }
      return { success: true, data: product };
    } catch (error) {
      return { success: false, error: 'Error al obtener el producto' };
    }
  }
}

// Tipos para la respuesta del caso de uso
type GetProductResponse = 
  | { success: true; data: Product } 
  | { success: false; error: string };

// 3. Puerto (interfaz de salida)
interface ProductRepository {
  getById(id: string): Promise<Product | null>;
  getAll(): Promise<Product[]>;
  save(product: Product): Promise<void>;
  delete(id: string): Promise<boolean>;
}

// 4. Adaptador (implementación concreta del repositorio)
class MongoProductRepository implements ProductRepository {
  // Implementación para MongoDB
  async getById(id: string): Promise<Product | null> {
    // Implementación real aquí
    return null;
  }
  
  // Implementación de otros métodos...
}

// 5. Controlador (adaptador de entrada)
class ProductController {
  constructor(private getProductUseCase: GetProductUseCase) {}
  
  async getProduct(req: any, res: any): Promise<void> {
    const productId = req.params.id;
    const result = await this.getProductUseCase.execute(productId);
    
    if (result.success) {
      res.status(200).json(result.data);
    } else {
      res.status(404).json({ error: result.error });
    }
  }
}

// 6. Configuración (composición de dependencias)
function setupProductEndpoints(app: any): void {
  const repository = new MongoProductRepository();
  const useCase = new GetProductUseCase(repository);
  const controller = new ProductController(useCase);
  
  app.get('/products/:id', (req: any, res: any) => controller.getProduct(req, res));
}
```

La elección entre Hexagonal o Clean Architecture a menudo depende de las preferencias del equipo y del contexto específico del proyecto, pero ambas logran objetivos similares de separación de preocupaciones y facilidad para probar el código.

## Conclusiones sobre Arquitecturas Modernas

Al finalizar este recorrido por las arquitecturas modernas de software, es evidente que no existe una solución única que sea óptima para todos los escenarios. La elección arquitectónica debe estar guiada por factores como:

1. **Contexto del negocio**: Complejidad del dominio, velocidad de cambio de los requisitos y criticidad del sistema.

2. **Escalabilidad requerida**: Tanto en términos de carga de trabajo como de equipos de desarrollo.

3. **Madurez organizacional**: Capacidades técnicas, procesos de desarrollo y cultura organizacional.

4. **Consideraciones operativas**: Infraestructura disponible, recursos de monitoreo y estrategias de despliegue.

Las arquitecturas presentadas en este capítulo —microservicios, serverless, monolito modular y hexagonal— ofrecen diferentes compromisos entre simplicidad operativa, velocidad de desarrollo, mantenibilidad y escalabilidad. En la práctica, muchos sistemas exitosos adoptan enfoques híbridos, combinando elementos de distintas arquitecturas según las necesidades específicas de cada componente del sistema.

Lo fundamental es que la arquitectura elegida debe servir a los objetivos del negocio y evolucionar con él, recordando siempre que la mejor arquitectura es aquella que facilita el cambio con el menor costo posible, permitiendo que el software siga siendo relevante en un entorno empresarial dinámico.

## 4.2 Patrones de Diseño Aplicados

Los patrones de diseño representan soluciones probadas y refinadas a problemas recurrentes en el desarrollo de software. Estas "recetas" arquitectónicas, popularizadas por el libro "Design Patterns: Elements of Reusable Object-Oriented Software" (conocido como el libro "Gang of Four" o GoF), permiten aprovechar la experiencia colectiva de la industria para construir sistemas más mantenibles, flexibles y robustos.

La implementación efectiva de patrones de diseño requiere no solo conocer su estructura, sino también comprender cuándo y cómo aplicarlos adecuadamente. Un patrón mal aplicado puede introducir complejidad innecesaria, mientras que el mismo patrón aplicado en el contexto correcto puede simplificar enormemente la arquitectura de un sistema.

### 4.2.1 Patrones Creacionales

Los patrones creacionales abordan el desafío fundamental de la creación de objetos en sistemas orientados a objetos. Estos patrones encapsulan el conocimiento sobre qué clases concretas utiliza el sistema, cómo se crean las instancias y cómo se relacionan entre sí, promoviendo la flexibilidad al desacoplar el sistema de cómo sus objetos son creados, compuestos y representados.

Al implementar patrones creacionales, un sistema puede funcionar con tipos abstractos, independizándose de los detalles concretos de instanciación y permitiendo que la configuración de objetos varíe sin afectar el código cliente. Esto resulta particularmente valioso cuando la creación de objetos involucra procesos complejos, decisiones en tiempo de ejecución o la necesidad de mantener coherencia en todo el sistema.

#### Patrón Factory:

Proporciona una interfaz para crear objetos sin especificar sus clases concretas.

```typescript
// Patrón Factory en TypeScript

// Interfaz común para los productos
interface Button {
  render(): string;
  onClick(): void;
}

// Implementaciones concretas
class WindowsButton implements Button {
  render(): string {
    return '<button class="windows-button">Click me</button>';
  }
  
  onClick(): void {
    console.log('Windows button clicked');
  }
}

class MacOSButton implements Button {
  render(): string {
    return '<button class="macos-button">Click me</button>';
  }
  
  onClick(): void {
    console.log('MacOS button clicked');
  }
}

// Factory abstracta
abstract class ButtonFactory {
  abstract createButton(): Button;
  
  // Método que utiliza la fábrica
  renderButton(): string {
    const button = this.createButton();
    return button.render();
  }
}

// Fábricas concretas
class WindowsButtonFactory extends ButtonFactory {
  createButton(): Button {
    return new WindowsButton();
  }
}

class MacOSButtonFactory extends ButtonFactory {
  createButton(): Button {
    return new MacOSButton();
  }
}

// Uso
function createUI(factory: ButtonFactory): void {
  const buttonHtml = factory.renderButton();
  document.body.innerHTML = buttonHtml;
}

// Determinamos qué fábrica usar según el sistema operativo
const os = getOperatingSystem(); // Función ficticia que detecta el SO
let factory: ButtonFactory;

if (os === 'Windows') {
  factory = new WindowsButtonFactory();
} else {
  factory = new MacOSButtonFactory();
}

createUI(factory);
```

#### Patrón Singleton:

Garantiza que una clase tenga una única instancia y proporciona un punto de acceso global a ella.

```typescript
// Patrón Singleton en TypeScript

class Database {
  private static instance: Database;
  private connection: any;
  
  // Constructor privado para prevenir instanciación directa
  private constructor() {
    this.connection = {
      host: 'localhost',
      port: 5432,
      username: 'user',
      password: 'password'
    };
    console.log('Conexión a la base de datos establecida');
  }
  
  // Método para obtener la única instancia
  public static getInstance(): Database {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    
    return Database.instance;
  }
  
  // Métodos de ejemplo
  public query(sql: string): any {
    console.log(`Ejecutando consulta: ${sql}`);
    return { results: [], affectedRows: 0 };
  }
  
  public close(): void {
    console.log('Cerrando conexión a la base de datos');
    this.connection = null;
  }
}

// Uso
const db1 = Database.getInstance();
const db2 = Database.getInstance();

console.log('¿Son la misma instancia?', db1 === db2); // true

db1.query('SELECT * FROM users');
db2.close(); // Ambas variables refieren a la misma instancia
```

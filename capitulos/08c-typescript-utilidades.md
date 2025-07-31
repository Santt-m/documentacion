# 8.3 Utilidades de Tipo y Patrones Avanzados en TypeScript

## Introducción

A medida que un proyecto TypeScript crece en complejidad, la gestión eficiente de los tipos se vuelve cada vez más importante. En este capítulo, exploraremos las capacidades avanzadas del sistema de tipos de TypeScript que permiten crear código más robusto, reutilizable y expresivo.

TypeScript ofrece un conjunto de herramientas sofisticadas para manipular y transformar tipos, desde utilidades incorporadas hasta técnicas avanzadas como los mapped types y conditional types. Estas características permiten a los desarrolladores implementar patrones elegantes que aprovechan al máximo el sistema de tipado estático, haciendo que el código sea no sólo más seguro sino también más expresivo y mantenible.

En este capítulo, profundizaremos en el universo de las utilidades de tipo y los patrones avanzados de TypeScript, desde los guardias de tipo (type guards) que mejoran el control de flujo hasta los decoradores que añaden metaprogramación elegante. Estas técnicas representan el verdadero poder de TypeScript como un superconjunto de JavaScript orientado a tipos.

## 8.3.1 Type Guards y Narrowing

Los type guards (guardias de tipo) permiten refinar el tipo de una variable dentro de bloques condicionales, facilitando el trabajo con uniones de tipos.

### Type Guards con operadores de tipo

```typescript
function processValue(value: string | number) {
  // Type guard usando typeof
  if (typeof value === "string") {
    // TypeScript sabe que aquí 'value' es de tipo string
    return value.toUpperCase();
  } else {
    // TypeScript sabe que aquí 'value' es de tipo number
    return value.toFixed(2);
  }
}

// Type guard con arrays
function processItems(items: string | string[]) {
  // Type guard usando Array.isArray
  if (Array.isArray(items)) {
    // TypeScript sabe que aquí 'items' es string[]
    return items.join(", ");
  } else {
    // TypeScript sabe que aquí 'items' es string
    return items.trim();
  }
}
```

### Type Guards con instancias de clases

```typescript
class Customer {
  id: number;
  name: string;
  
  constructor(id: number, name: string) {
    this.id = id;
    this.name = name;
  }
  
  placeOrder() {
    console.log(`${this.name} placed an order`);
  }
}

class Employee {
  id: number;
  name: string;
  department: string;
  
  constructor(id: number, name: string, department: string) {
    this.id = id;
    this.name = name;
    this.department = department;
  }
  
  assignTask(task: string) {
    console.log(`${this.name} from ${this.department} assigned to "${task}"`);
  }
}

function processUser(user: Customer | Employee) {
  // Type guard usando instanceof
  if (user instanceof Customer) {
    // TypeScript sabe que aquí 'user' es Customer
    user.placeOrder();
  } else {
    // TypeScript sabe que aquí 'user' es Employee
    user.assignTask("Review code");
  }
}
```

### Type Guards personalizados (user-defined type guards)

```typescript
// Definición de interfaces
interface Bird {
  fly(): void;
  layEggs(): void;
}

interface Fish {
  swim(): void;
  layEggs(): void;
}

// Type guard personalizado usando is
function isFish(pet: Bird | Fish): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function handlePet(pet: Bird | Fish) {
  if (isFish(pet)) {
    // TypeScript sabe que aquí 'pet' es de tipo Fish
    pet.swim();
  } else {
    // TypeScript sabe que aquí 'pet' es de tipo Bird
    pet.fly();
  }
}

// Type guard con aserciones
function assertIsNumber(val: any): asserts val is number {
  if (typeof val !== "number") {
    throw new Error("Value must be a number");
  }
}

function calculateArea(width: any, height: any): number {
  assertIsNumber(width);
  assertIsNumber(height);
  
  // Después de las aserciones, TypeScript sabe que width y height son números
  return width * height;
}
```

### Discriminated Unions

```typescript
// Uniones discriminadas con propiedades literales
type Circle = {
  kind: "circle";
  radius: number;
};

type Rectangle = {
  kind: "rectangle";
  width: number;
  height: number;
};

type Triangle = {
  kind: "triangle";
  base: number;
  height: number;
};

type Shape = Circle | Rectangle | Triangle;

// TypeScript puede hacer narrowing basado en la propiedad discriminadora "kind"
function calculateArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      // TypeScript sabe que shape es Circle
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      // TypeScript sabe que shape es Rectangle
      return shape.width * shape.height;
    case "triangle":
      // TypeScript sabe que shape es Triangle
      return (shape.base * shape.height) / 2;
    default:
      // Exhaustiveness checking
      const _exhaustiveCheck: never = shape;
      throw new Error(`Unhandled shape kind: ${_exhaustiveCheck}`);
  }
}
```

## 8.3.2 Utilidades de tipo incorporadas

TypeScript incluye varios tipos utilitarios que facilitan transformaciones de tipos comunes.

```typescript
// Partial: hace todas las propiedades opcionales
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

function updateUser(user: User, updates: Partial<User>): User {
  return { ...user, ...updates };
}

const user: User = {
  id: 1,
  name: "John",
  email: "john@example.com",
  age: 30
};

// Solo necesitamos proporcionar las propiedades que queremos actualizar
const updatedUser = updateUser(user, { age: 31, name: "Johnny" });

// Required: hace todas las propiedades requeridas
interface BlogPostDraft {
  title?: string;
  content?: string;
  tags?: string[];
  authorId?: number;
}

function publishPost(post: Required<BlogPostDraft>) {
  // Ahora podemos estar seguros que todas las propiedades existen
  console.log(`Publishing "${post.title}" by author ${post.authorId}`);
}

// Readonly: hace todas las propiedades de solo lectura
const frozenUser: Readonly<User> = {
  id: 2,
  name: "Alice",
  email: "alice@example.com",
  age: 25
};

// frozenUser.age = 26; // Error: No se puede asignar a 'age' porque es una propiedad de solo lectura

// Record: crea un tipo con las propiedades especificadas de un tipo
type UserRole = "admin" | "user" | "guest";
type RoleDescription = Record<UserRole, string>;

const roleDescriptions: RoleDescription = {
  admin: "Full system access",
  user: "Limited access to resources",
  guest: "Read-only access"
};

// Pick: selecciona un conjunto de propiedades de un tipo
type UserCredentials = Pick<User, "email" | "id">;

function login(credentials: UserCredentials) {
  // credentials solo tiene email e id
}

// Omit: excluye propiedades de un tipo
type PublicUser = Omit<User, "id" | "email">;

function getPublicInfo(user: User): PublicUser {
  const { id, email, ...publicInfo } = user;
  return publicInfo;
}

// Exclude: excluye tipos de una unión
type NumericOrString = number | string | boolean;
type JustStrings = Exclude<NumericOrString, number | boolean>; // string

// Extract: extrae tipos de una unión que son asignables a otro tipo
type StringOrNumber = Extract<NumericOrString, string | number>; // string | number

// NonNullable: excluye null y undefined de un tipo
type NullableString = string | null | undefined;
type DefinitelyString = NonNullable<NullableString>; // string

// ReturnType: obtiene el tipo de retorno de una función
function createUser(name: string, age: number): User {
  return { id: Date.now(), name, email: `${name}@example.com`, age };
}

type NewUser = ReturnType<typeof createUser>; // User

// Parameters: obtiene los tipos de parámetros de una función como una tupla
type CreateUserParams = Parameters<typeof createUser>; // [string, number]
```

## 8.3.3 Mapped Types

Los mapped types permiten transformar cada propiedad de un tipo de manera uniforme, creando un tipo derivado.

```typescript
// Básico mapped type
type Nullable<T> = { [P in keyof T]: T[P] | null };

const nullableUser: Nullable<User> = {
  id: 1,
  name: null,
  email: "email@example.com",
  age: null
};

// Con modificadores de propiedad
type Optional<T> = { [P in keyof T]?: T[P] };
type ReadonlyType<T> = { readonly [P in keyof T]: T[P] };

// Removiendo modificadores existentes
type Mutable<T> = {
  -readonly [P in keyof T]: T[P]
};

type Required<T> = {
  [P in keyof T]-?: T[P]
};

// Mapped types con transformación de nombres
type PrefixedProps<T> = {
  [P in keyof T as `prefix_${string & P}`]: T[P]
};

interface Person {
  name: string;
  age: number;
}

// { prefix_name: string; prefix_age: number }
type PrefixedPerson = PrefixedProps<Person>;

// Mapped types condicionales
type OptionalIfString<T> = {
  [P in keyof T]: T[P] extends string ? T[P] | undefined : T[P]
};

// Filtrando propiedades por tipo
type OnlyStringProps<T> = {
  [P in keyof T as T[P] extends string ? P : never]: T[P]
};

interface Product {
  id: number;
  name: string;
  price: number;
  description: string;
  tags: string[];
}

// { name: string; description: string }
type ProductStrings = OnlyStringProps<Product>;

// Mapped types con genéricos
type FlagsWithPrefix<Type, Prefix extends string> = {
  [Property in keyof Type as `${Prefix}${string & Property}`]: boolean;
};

type Features = {
  darkMode: () => void;
  newUserProfile: () => void;
  userManagement: () => void;
};

// { enableDarkMode: boolean; enableNewUserProfile: boolean; enableUserManagement: boolean }
type FeatureFlags = FlagsWithPrefix<Features, 'enable'>;
```

## 8.3.4 Conditional Types

Los tipos condicionales permiten expresar relaciones complejas basadas en condiciones sobre otros tipos.

```typescript
// Sintaxis básica de tipos condicionales
// T extends U ? X : Y
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false

// Conditional types con uniones
type ToArray<T> = T extends any ? T[] : never;

type StringOrNumberArray = ToArray<string | number>;  // string[] | number[]

// Tipo condicional con inferencia usando infer
type ReturnTypeCustom<T> = T extends (...args: any[]) => infer R ? R : never;

function greet() { return "hello"; }
function multiply(x: number, y: number) { return x * y; }

type GreetReturn = ReturnTypeCustom<typeof greet>;  // string
type MultiplyReturn = ReturnTypeCustom<typeof multiply>;  // number

// Inferencia en posiciones múltiples
type FirstArgument<T> = T extends (first: infer F, ...args: any[]) => any ? F : never;

type MultiplyFirstArg = FirstArgument<typeof multiply>;  // number

// Distribución de tipos condicionales
type BoxedValue<T> = { value: T };
type Boxed<T> = T extends any ? BoxedValue<T> : never;

// Se convierte en BoxedValue<string> | BoxedValue<number>
type BoxedStringOrNumber = Boxed<string | number>;

// Evitando distribución con corchetes
type NoDistribute<T> = [T] extends [any] ? BoxedValue<T> : never;

// Se convierte en BoxedValue<string | number>
type BoxedUnion = NoDistribute<string | number>;

// Tipos condicionales recursivos
type Flatten<T> = T extends Array<infer Item> 
  ? Flatten<Item> 
  : T;

type NestedArray = number[][][];
type FlatArray = Flatten<NestedArray>;  // number

// Validación de parámetros
type RequireAtLeastOne<T> = { 
  [K in keyof T]: 
    { [L in K]: T[L] } & 
    { [L in Exclude<keyof T, K>]?: T[L] }
}[keyof T];

interface UpdateParams {
  name?: string;
  age?: number;
  email?: string;
}

// Debe tener al menos una propiedad
function updateUserProfile(params: RequireAtLeastOne<UpdateParams>) {
  // Implementación...
}

// OK
updateUserProfile({ name: "John" });

// OK
updateUserProfile({ age: 30, email: "john@example.com" });

// Error: Debe proporcionar al menos una propiedad
// updateUserProfile({});
```

## 8.3.5 Decoradores

Los decoradores proporcionan una forma de añadir anotaciones y metaprogramación a clases y miembros de clase. 

> Nota: Los decoradores son una característica experimental que requiere configuración específica en `tsconfig.json` (`"experimentalDecorators": true`).

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES5",
    "experimentalDecorators": true
  }
}
```

### Decoradores básicos

```typescript
// Decorador simple para una clase
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class BugReport {
  type = "report";
  title: string;
  
  constructor(title: string) {
    this.title = title;
  }
}

// Decorador de método que registra llamadas a métodos
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyKey} with args: ${JSON.stringify(args)}`);
    const result = originalMethod.apply(this, args);
    console.log(`Result: ${result}`);
    return result;
  };
  
  return descriptor;
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}

const calc = new Calculator();
calc.add(1, 2); 
// Logs:
// "Calling add with args: [1,2]"
// "Result: 3"
```

### Decoradores de fábrica (factory)

```typescript
// Decorador de fábrica que acepta parámetros
function format(formatString: string) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const original = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
      const result = original.apply(this, args);
      return formatString.replace('%s', result.toString());
    };
    
    return descriptor;
  };
}

class Greeter {
  @format('¡Hola, %s!')
  sayHello(name: string) {
    return name;
  }
}

const greeter = new Greeter();
console.log(greeter.sayHello('Mundo')); // "¡Hola, Mundo!"
```

### Decoradores de propiedades

```typescript
// Decorador de propiedad
function required(target: any, propertyKey: string) {
  let value: any;
  
  const getter = function() {
    return value;
  };
  
  const setter = function(newValue: any) {
    if (newValue === undefined || newValue === null) {
      throw new Error(`Property ${propertyKey} is required`);
    }
    value = newValue;
  };
  
  Object.defineProperty(target, propertyKey, {
    get: getter,
    set: setter,
    enumerable: true,
    configurable: true
  });
}

class User {
  @required
  username: string;
  
  constructor(username: string) {
    this.username = username;
  }
}

const validUser = new User("john_doe"); // OK
// const invalidUser = new User(null); // Error: Property username is required
```

### Decoradores de parámetros

```typescript
// Decorador de parámetro para validación
function validateLength(min: number, max: number) {
  return function(target: any, propertyKey: string, parameterIndex: number) {
    const originalMethod = target[propertyKey];
    
    target[propertyKey] = function(...args: any[]) {
      const value = args[parameterIndex];
      if (typeof value === 'string' && (value.length < min || value.length > max)) {
        throw new Error(`Parameter at index ${parameterIndex} must be between ${min} and ${max} characters`);
      }
      return originalMethod.apply(this, args);
    };
  };
}

class Registration {
  register(
    @validateLength(3, 20) username: string,
    email: string
  ) {
    console.log(`Registering ${username} with email ${email}`);
    return true;
  }
}

const registration = new Registration();
registration.register("john_doe", "john@example.com"); // OK
// registration.register("jo", "john@example.com"); // Error: must be between 3 and 20 characters
```

### Combinando múltiples decoradores

```typescript
function first() {
  console.log("first decorator");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("first(): called");
  };
}

function second() {
  console.log("second decorator");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("second(): called");
  };
}

class Example {
  @first()
  @second()
  method() {}
}

// Output:
// "second decorator"
// "first decorator"
// "second(): called"
// "first(): called"
```

## 8.3.6 Patrones avanzados

### Builder Pattern

```typescript
// Builder Pattern con interfaces fluidas y tipado avanzado
class ProductBuilder {
  // El objeto que se está construyendo
  private product: { 
    name?: string; 
    price?: number; 
    category?: string;
    features?: string[];
    isAvailable?: boolean;
  } = {};
  
  // Métodos fluidos para configurar propiedades
  setName(name: string): ProductBuilder {
    this.product.name = name;
    return this;
  }
  
  setPrice(price: number): ProductBuilder {
    this.product.price = price;
    return this;
  }
  
  setCategory(category: string): ProductBuilder {
    this.product.category = category;
    return this;
  }
  
  addFeature(feature: string): ProductBuilder {
    if (!this.product.features) {
      this.product.features = [];
    }
    this.product.features.push(feature);
    return this;
  }
  
  setAvailability(isAvailable: boolean): ProductBuilder {
    this.product.isAvailable = isAvailable;
    return this;
  }
  
  // Método para finalizar y validar el producto
  build(): Required<typeof this.product> {
    // Validar que todas las propiedades requeridas están presentes
    if (!this.product.name || !this.product.price || !this.product.category) {
      throw new Error("Product must have name, price, and category");
    }
    
    // Proporcionar valores predeterminados para propiedades opcionales
    const finalProduct = {
      ...this.product,
      features: this.product.features || [],
      isAvailable: this.product.isAvailable !== undefined ? 
                  this.product.isAvailable : true
    } as Required<typeof this.product>;
    
    return finalProduct;
  }
}

// Uso del builder
const product = new ProductBuilder()
  .setName("Laptop")
  .setPrice(999)
  .setCategory("Electronics")
  .addFeature("16GB RAM")
  .addFeature("512GB SSD")
  .build();
```

### Factory Pattern

```typescript
// Factory Pattern con discriminated unions
type PaymentMethod = "credit" | "debit" | "paypal";

// Base interface
interface Payment {
  method: PaymentMethod;
  amount: number;
}

// Specific payment types
interface CreditCardPayment extends Payment {
  method: "credit";
  cardNumber: string;
  securityCode: string;
  expiryDate: string;
}

interface DebitCardPayment extends Payment {
  method: "debit";
  cardNumber: string;
  pin: string;
}

interface PayPalPayment extends Payment {
  method: "paypal";
  email: string;
  password: string;
}

// Discriminated union of payment types
type PaymentTypes = CreditCardPayment | DebitCardPayment | PayPalPayment;

// Factory function
function createPayment(paymentInfo: PaymentTypes): PaymentTypes {
  // Validation logic could be added here
  return paymentInfo;
}

// Usage
const creditPayment = createPayment({
  method: "credit",
  amount: 100,
  cardNumber: "4111-1111-1111-1111",
  securityCode: "123",
  expiryDate: "12/25"
});

// TypeScript knows this is a CreditCardPayment
console.log(creditPayment.securityCode);

// Usage with type guards
function processPayment(payment: PaymentTypes) {
  switch (payment.method) {
    case "credit":
      console.log(`Processing ${payment.amount} with credit card ${payment.cardNumber}`);
      break;
    case "debit":
      console.log(`Processing ${payment.amount} with debit card ${payment.cardNumber}`);
      break;
    case "paypal":
      console.log(`Processing ${payment.amount} with PayPal account ${payment.email}`);
      break;
  }
}
```

### Singleton con TypeScript

```typescript
class DatabaseConnection {
  private static instance: DatabaseConnection | null = null;
  private connectionString: string;
  
  // Constructor privado
  private constructor(connectionString: string) {
    this.connectionString = connectionString;
    // Simulando conexión a base de datos
    console.log(`Connected to database with: ${connectionString}`);
  }
  
  // Método estático para obtener la instancia
  public static getInstance(connectionString: string): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection(connectionString);
    }
    
    return DatabaseConnection.instance;
  }
  
  // Métodos de la clase
  public query(sql: string): any[] {
    console.log(`Executing query: ${sql}`);
    return [{ id: 1, name: "Example" }];
  }
  
  public close(): void {
    console.log("Connection closed");
    // Lógica para cerrar la conexión
  }
}

// Uso del singleton
const db1 = DatabaseConnection.getInstance("mongodb://localhost:27017");
const db2 = DatabaseConnection.getInstance("mongodb://localhost:27017");

console.log(db1 === db2); // true, misma instancia
```

## Conclusión

Las utilidades de tipo y los patrones avanzados de TypeScript representan un nivel superior de abstracción y seguridad en el desarrollo de aplicaciones JavaScript. A través de estas características, TypeScript brinda la posibilidad de expresar relaciones de tipo complejas que capturan con precisión la intención del código, detectando errores potenciales en tiempo de compilación que serían difíciles de encontrar en JavaScript puro.

La maestría en el uso de type guards, mapped types, conditional types y decoradores permite a los desarrolladores implementar patrones de diseño robustos y crear abstracciones poderosas que mejoran significativamente la mantenibilidad y escalabilidad del código. Estas herramientas avanzadas son particularmente valiosas en proyectos grandes donde la claridad del código y la detección temprana de errores tienen un impacto directo en la productividad y la calidad del software.

Sin embargo, es importante recordar que con gran poder viene gran responsabilidad. Las características avanzadas del sistema de tipos de TypeScript deben usarse juiciosamente, teniendo en cuenta el equilibrio entre la complejidad del código y los beneficios que aportan. Un uso excesivo de técnicas avanzadas puede hacer que el código sea difícil de entender para otros desarrolladores.

En el próximo capítulo, exploraremos cómo aplicar estos conocimientos de TypeScript en el contexto de frameworks populares como React, Angular y Node.js, viendo cómo el tipado estático mejora el desarrollo de aplicaciones en estos ecosistemas.

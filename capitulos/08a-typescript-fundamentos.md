# 8.1 Fundamentos y Sistema de Tipos de TypeScript

## Introducción

En el universo del desarrollo web moderno, JavaScript ha sido el lenguaje dominante en el frontend durante décadas. Sin embargo, a medida que las aplicaciones web se vuelven más complejas y los equipos de desarrollo crecen, las limitaciones inherentes de JavaScript —particularmente su naturaleza dinámicamente tipada— se convierten en desafíos significativos.

TypeScript surge como respuesta a estos desafíos. Desarrollado por Microsoft y lanzado por primera vez en 2012, TypeScript se ha convertido en una herramienta esencial en el arsenal de los desarrolladores modernos. No es simplemente una extensión de JavaScript; es un superconjunto que añade un sistema de tipos estáticos opcional, manteniendo total compatibilidad con el código JavaScript existente.

En este capítulo, exploraremos los fundamentos de TypeScript, desde su sistema de tipos hasta sus características avanzadas, y entenderemos por qué es considerado "el superhéroe del desarrollo JavaScript" en el mundo del desarrollo web actual.

## 8.1.1 ¿Qué es TypeScript?

TypeScript es un lenguaje de programación de código abierto desarrollado y mantenido por Microsoft. Como superconjunto de JavaScript, todo código JavaScript válido también es código TypeScript válido, pero TypeScript añade características adicionales, siendo la más destacada su sistema de tipos estáticos.

### El problema que resuelve TypeScript

JavaScript, siendo un lenguaje dinámicamente tipado, ofrece flexibilidad pero también introduce vulnerabilidades:

- **Errores en tiempo de ejecución**: Muchos errores solo se descubren cuando el código se ejecuta, a menudo en producción
- **Falta de documentación estructurada**: El propósito y uso de variables y funciones puede ser ambiguo
- **Dificultad para refactorizar**: Sin conocimiento explícito de tipos, cambiar código existente es arriesgado
- **Herramientas de desarrollo limitadas**: Los IDEs no pueden proporcionar asistencia completa sin información de tipos

TypeScript aborda estos problemas añadiendo un sistema de tipos que permite detectar errores durante el desarrollo, antes de la ejecución del código.

### Filosofía de diseño

El equipo de TypeScript sigue algunos principios clave:

1. **Compatibilidad con JavaScript**: Todo código JavaScript es código TypeScript válido
2. **Tipado gradual**: Permite adoptar TypeScript incrementalmente, añadiendo tipos donde se necesiten
3. **Tipo como documentación**: El sistema de tipos sirve como documentación viva del código
4. **Tipado estructural**: Los tipos se basan en la estructura (forma) de los objetos, no en su nombre
5. **Inferencia de tipos**: TypeScript puede deducir tipos automáticamente en muchos casos

## 8.1.2 Ventajas de TypeScript

La adopción de TypeScript ofrece numerosos beneficios que van más allá de la simple detección de errores:

### Detección temprana de errores

El sistema de tipos estáticos permite identificar problemas comunes durante el desarrollo:

```typescript
// JavaScript permitiría esto sin advertencias
const user = { name: "John", email: "john@example.com" };
console.log(user.phone); // undefined en JavaScript, error en TypeScript
```

### Documentación integrada

Los tipos actúan como documentación que siempre está actualizada:

```typescript
// El tipo documenta claramente qué espera la función
function sendEmail(options: { 
  to: string; 
  subject: string; 
  body: string; 
  attachments?: string[] 
}): Promise<boolean> {
  // Implementación...
}
```

### Mejora en herramientas de desarrollo

Los editores e IDEs pueden proporcionar:

- **Autocompletado inteligente**: Sugerencias precisas basadas en tipos
- **Refactorización segura**: Cambios automatizados con verificación de tipos
- **Navegación de código mejorada**: Saltar a definiciones, encontrar referencias
- **Detección de errores en tiempo real**: Resaltado de problemas mientras se escribe

### Mantenimiento mejorado

Para bases de código grandes y equipos:

- **Cambios más seguros**: Los cambios incompatibles se identifican inmediatamente
- **Mejor comunicación**: Los contratos de tipo establecen expectativas claras entre componentes
- **Confianza al refactorizar**: Mayor seguridad al modificar código existente
- **Onboarding más rápido**: Nuevos desarrolladores pueden entender el código más rápidamente

## 8.1.3 Configuración básica

Para comenzar con TypeScript, se requiere una configuración mínima. El archivo `tsconfig.json` es el corazón de cualquier proyecto TypeScript.

```json
// tsconfig.json - Configuración básica de TypeScript
{
  "compilerOptions": {
    "target": "es5",           // Versión de ECMAScript de salida
    "module": "commonjs",      // Sistema de módulos
    "strict": true,            // Habilitar todas las comprobaciones estrictas de tipo
    "esModuleInterop": true,   // Permitir imports de CommonJS de forma más natural
    "skipLibCheck": true,      // Saltarse la comprobación de tipos en archivos .d.ts
    "forceConsistentCasingInFileNames": true, // Asegurar consistencia en nombres de archivos
    "outDir": "./dist",        // Directorio de salida
    "rootDir": "./src"         // Directorio raíz
  },
  "include": ["src/**/*"],     // Archivos a incluir
  "exclude": ["node_modules", "**/*.test.ts"] // Archivos a excluir
}
```

### Opciones principales de configuración

- **target**: Especifica la versión de ECMAScript para la salida compilada
- **module**: Define el sistema de módulos para la salida (commonjs, esnext, umd, etc.)
- **strict**: Activa todas las comprobaciones estrictas de tipo
- **outDir/rootDir**: Especifica los directorios de entrada y salida
- **lib**: Define qué librerías de definición incluir (DOM, ES2020, etc.)
- **include/exclude**: Controla qué archivos procesa el compilador

### Compilación y ejecución

Para compilar código TypeScript:

```bash
# Instalación global
npm install -g typescript

# Compilar un archivo
tsc archivo.ts

# Compilar el proyecto según tsconfig.json
tsc

# Compilar y observar cambios
tsc --watch
```

## 8.1.4 Sistema de Tipos

El sistema de tipos es la característica central de TypeScript. Proporciona una manera de describir la forma y el comportamiento de los objetos.

### Tipos básicos

TypeScript incluye tipos primitivos similares a JavaScript, pero con anotaciones explícitas:

```typescript
// Tipos primitivos
let isDone: boolean = false;
let decimal: number = 6;
let color: string = "blue";
let bigNumber: bigint = 100n;
let symbol: Symbol = Symbol("unique");

// Arrays
let list: number[] = [1, 2, 3];
let names: Array<string> = ["Alice", "Bob"]; // Sintaxis genérica

// Tuples (arrays con número fijo de elementos de tipos específicos)
let tuple: [string, number] = ["hello", 10];

// Enums (conjunto de constantes nombradas)
enum Color {Red, Green, Blue}
let c: Color = Color.Green;  // 1
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT"
}

// Any (evitar cuando sea posible)
let notSure: any = 4;
notSure = "maybe a string instead";

// Unknown (tipo más seguro que any)
let userInput: unknown = getUserInput();
// Requiere verificación antes de usarlo con tipos específicos
if (typeof userInput === "string") {
  let length: number = userInput.length;
}

// Void (para funciones sin retorno)
function warnUser(): void {
  console.log("This is a warning message");
}

// Null y Undefined
let u: undefined = undefined;
let n: null = null;

// Never (para funciones que nunca retornan o siempre lanzan excepciones)
function error(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {}
}

// Object (cualquier valor que no sea un tipo primitivo)
let obj: object = { key: "value" };

// Literal types (valores específicos)
let status: "ready" | "loading" | "error";
status = "loading"; // Correcto
// status = "waiting"; // Error: Type '"waiting"' is not assignable to type '"ready" | "loading" | "error"'
```

### Type Assertions (Aserciones de tipo)

Las aserciones de tipo permiten al desarrollador indicar al compilador que trate un valor como un tipo específico:

```typescript
// Usando "as"
let someValue: unknown = "hello world";
let strLength: number = (someValue as string).length;

// Usando sintaxis de "<>"
let someValue2: unknown = "hello world";
let strLength2: number = (<string>someValue2).length;
```

## 8.1.5 Interfaces

Las interfaces son una herramienta poderosa en TypeScript para definir contratos dentro de tu código y con código externo.

```typescript
// Definición básica
interface User {
  id: number;
  name: string;
  email: string;
  age?: number; // Propiedad opcional con "?"
  readonly createdAt: Date; // Propiedad de solo lectura
}

// Implementación
const newUser: User = {
  id: 1,
  name: "John Doe",
  email: "john@example.com",
  createdAt: new Date()
};

// Propiedades opcionales
interface Configuration {
  debug?: boolean;
  timeoutInMs?: number;
}

// Propiedades de solo lectura
interface Point {
  readonly x: number;
  readonly y: number;
}

// Interfaces para funciones
interface SearchFunction {
  (source: string, subString: string): boolean;
}

const mySearch: SearchFunction = (src, sub) => src.includes(sub);

// Interfaces indexables
interface StringArray {
  [index: number]: string;
}

const myArray: StringArray = ["Bob", "Alice"];

interface Dictionary {
  [key: string]: number;
}

const scores: Dictionary = {
  "John": 89,
  "Jane": 95
};

// Extensión de interfaces
interface Employee extends User {
  employeeId: string;
  department: string;
}

// Múltiple herencia de interfaces
interface Manager extends Employee, Admin {
  managedTeams: string[];
}

// Implementando interfaces en clases
interface ClockInterface {
  currentTime: Date;
  setTime(d: Date): void;
}

class Clock implements ClockInterface {
  currentTime: Date = new Date();
  
  setTime(d: Date) {
    this.currentTime = d;
  }
}
```

## 8.1.6 Types vs. Interfaces

TypeScript ofrece dos formas principales de definir tipos: `interface` y `type`. Aunque son similares, tienen algunas diferencias importantes.

```typescript
// Alias de tipo básico
type ID = string | number;

// Tipo objeto
type Point = {
  x: number;
  y: number;
};

// Union types
type Result = "success" | "failure" | "error";

// Intersection types
type Admin = User & { permissions: string[] };

// Type con genéricos
type Container<T> = { value: T };

// Diferencia clave con interfaces: los types no se pueden extender después de ser definidos
```

### Comparativa entre Types e Interfaces

| Característica | Type | Interface |
|--------------|------|-----------|
| Declaración | `type Point = { x: number, y: number }` | `interface Point { x: number, y: number }` |
| Uniones/Intersecciones | ✅ Soporta `type A = B \| C` | ❌ No directamente |
| Primitivos | ✅ `type ID = string \| number` | ❌ Solo objetos |
| Declaration merging | ❌ No se pueden extender después | ✅ Se pueden extender con más declaraciones |
| Rendimiento | Ligeramente mejor para tipos complejos | Ligeramente mejor para objetos simples |

### ¿Cuándo usar cada uno?

- **Usa interfaces cuando**:
  - Defines una API pública o objetos que serán extendidos
  - Necesitas la característica de "declaration merging"
  - Trabajas con clases que implementarán tu tipo
  
- **Usa types cuando**:
  - Necesitas unions o intersecciones
  - Trabajas con tipos primitivos
  - Necesitas mapeos de tipos o tipos condicionales
  - El tipo no será extendido

## Conclusión

TypeScript proporciona una capa de seguridad y documentación a JavaScript que resulta invaluable en proyectos de mediano y gran tamaño. Sus fundamentos y sistema de tipos permiten detectar errores tempranamente, mejoran la legibilidad del código y facilitan el trabajo en equipo.

La adopción de TypeScript implica una curva de aprendizaje, pero los beneficios a largo plazo superan significativamente el costo inicial. La capacidad de detectar errores antes de la ejecución, la mejora en las herramientas de desarrollo y la claridad que aporta al código hacen que sea una elección estratégica para equipos que buscan construir aplicaciones robustas y mantenibles.

En los siguientes capítulos, profundizaremos en aspectos más avanzados de TypeScript como funciones, clases, genéricos y patrones de diseño que potencian aún más las capacidades de este "superhéroe" del desarrollo moderno.

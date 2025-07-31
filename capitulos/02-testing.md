# Capítulo 2: Testing: Asegurando la Calidad del Software

El testing es un componente crucial del desarrollo de software moderno que garantiza la calidad, fiabilidad y robustez de las aplicaciones. Este capítulo explora los diferentes tipos de pruebas, herramientas y mejores prácticas para implementar una estrategia de testing efectiva.

## Pruebas Unitarias: Validando la Lógica Atómica

Las pruebas unitarias se centran en comprobar que las unidades individuales de código (funciones, métodos, clases) funcionan correctamente de forma aislada. Son la base de la pirámide de pruebas y proporcionan feedback rápido a los desarrolladores.

### Ejemplo de una Función Simple con TypeScript:

```typescript
// src/utils/math.ts
export function sum(a: number, b: number): number {
  return a + b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}

// tests/unit/math.test.ts (usando Jest/Vitest)
import { sum, multiply } from '../src/utils/math';

describe('Math Utilities', () => {
  test('sum should correctly add two numbers', () => {
    expect(sum(1, 2)).toBe(3);
    expect(sum(-1, 1)).toBe(0);
    expect(sum(0, 0)).toBe(0);
  });

  test('multiply should correctly multiply two numbers', () => {
    expect(multiply(2, 3)).toBe(6);
    expect(multiply(-2, 3)).toBe(-6);
    expect(multiply(0, 5)).toBe(0);
  });
});
```

### Características de las Pruebas Unitarias:

- **Rápidas:** Deben ejecutarse en milisegundos, permitiendo una retroalimentación instantánea.
- **Aisladas:** No dependen de servicios externos, bases de datos o sistemas de archivos.
- **Deterministas:** Siempre producen el mismo resultado si el código no cambia.
- **Automatizadas:** Pueden ejecutarse con un solo comando, típicamente como parte del flujo de CI/CD.

### Herramientas comunes: Jest, Vitest, Mocha + Chai, Jasmine.

### Mocking y Stubbing: Aislando Dependencias

El mocking y stubbing son técnicas esenciales para aislar el código bajo prueba de sus dependencias externas, permitiendo pruebas más rápidas, confiables y enfocadas.

#### ¿Qué es Mocking?

El mocking crea objetos simulados (mocks) que imitan el comportamiento de objetos reales de forma controlada. Los mocks:

- Verifican las interacciones: Registran cómo, cuándo y cuántas veces son llamados
- Validan argumentos: Comprueban que los parámetros pasados son correctos
- Controlan retornos: Devuelven valores predefinidos cuando son invocados

**Ejemplo de Mocking con Jest:**

```typescript
// src/services/userService.ts
export class UserService {
  async fetchUser(id: string) {
    // En una implementación real, esto haría una llamada API
    const response = await fetch(`https://api.example.com/users/${id}`); 
    if (!response.ok) throw new Error('User not found');
    return response.json();
  }
}

// src/components/UserProfile.ts
import { UserService } from '../services/userService';

export class UserProfile {
  private userService: UserService;
  
  constructor(userService: UserService) {
    this.userService = userService;
  }
  
  async displayUserInfo(userId: string) {
    try {
      const user = await this.userService.fetchUser(userId);
      return `Nombre: ${user.name}, Email: ${user.email}`;
    } catch (error) {
      return 'Error: Usuario no encontrado';
    }
  }
}

// tests/unit/UserProfile.test.ts
import { UserProfile } from '../src/components/UserProfile';
import { UserService } from '../src/services/userService';

// Mock completo del servicio
jest.mock('../src/services/userService');

describe('UserProfile', () => {
  // Limpiar todos los mocks después de cada prueba
  afterEach(() => {
    jest.clearAllMocks();
  });

  test('should display user info correctly', async () => {
    // Configurar el mock para devolver datos específicos
    const mockFetchUser = jest.fn().mockResolvedValue({
      name: 'Ana García',
      email: 'ana@example.com'
    });
    
    // Asignar la función mock al método del servicio
    (UserService as jest.MockedClass<typeof UserService>)
      .prototype.fetchUser = mockFetchUser;
    
    // Crear instancia con el servicio mockeado
    const userProfile = new UserProfile(new UserService());
    const result = await userProfile.displayUserInfo('123');
    
    // Verificar que el servicio fue llamado correctamente
    expect(mockFetchUser).toHaveBeenCalledWith('123');
    expect(mockFetchUser).toHaveBeenCalledTimes(1);
    
    // Verificar el resultado
    expect(result).toBe('Nombre: Ana García, Email: ana@example.com');
  });

  test('should handle errors correctly', async () => {
    // Configurar el mock para simular un error
    const mockFetchUser = jest.fn().mockRejectedValue(
      new Error('User not found')
    );
    
    (UserService as jest.MockedClass<typeof UserService>)
      .prototype.fetchUser = mockFetchUser;
    
    const userProfile = new UserProfile(new UserService());
    const result = await userProfile.displayUserInfo('999');
    
    expect(mockFetchUser).toHaveBeenCalledWith('999');
    expect(result).toBe('Error: Usuario no encontrado');
  });
});
```

#### ¿Qué es Stubbing?

El stubbing crea implementaciones simplificadas (stubs) que reemplazan funcionalidades complejas. Los stubs:

- Proporcionan respuestas predefinidas: Devuelven datos fijos sin lógica compleja
- No verifican interacciones: A diferencia de los mocks, no validan cómo son usados
- Simplifican dependencias: Reemplazan componentes complejos con alternativas simples

**Ejemplo de Stubbing:**

```typescript
// Stub manual de una API de pago
class PaymentGatewayStub {
  processPayment(amount: number, cardDetails: any) {
    // Simplemente devuelve una respuesta exitosa sin procesar realmente el pago
    return Promise.resolve({
      success: true,
      transactionId: 'stub-transaction-123',
      timestamp: new Date().toISOString()
    });
  }
  
  // Podemos añadir un método para simular fallos cuando sea necesario
  failPayment(amount: number, cardDetails: any) {
    return Promise.resolve({
      success: false,
      error: 'Insufficient funds',
      errorCode: 'ERR-FUNDS'
    });
  }
}

// Usamos el stub en nuestras pruebas
test('should process order successfully when payment succeeds', async () => {
  const paymentStub = new PaymentGatewayStub();
  const orderProcessor = new OrderProcessor(paymentStub);
  
  const result = await orderProcessor.processOrder({
    items: [{id: '1', quantity: 2}],
    total: 199.99,
    payment: {cardNumber: '4111111111111111', expiry: '12/25', cvv: '123'}
  });
  
  expect(result.status).toBe('completed');
  expect(result.orderId).toBeDefined();
});
```

#### Diferencias Clave entre Mocks y Stubs

| Aspecto | Mocks | Stubs |
| --- | --- | --- |
| Propósito | Verificar comportamiento | Proporcionar datos |
| Verificación | Comprueban cómo se usan | No verifican interacciones |
| Complejidad | Más complejos de configurar | Generalmente más simples |
| Uso típico | Validar que se llama correctamente a dependencias | Simular respuestas de sistemas externos |

#### Cuándo Usar Cada Técnica

**Usa Mocks cuando:**
- Necesites verificar que ciertos métodos son llamados con argumentos específicos
- Quieras comprobar el número exacto de llamadas a un método
- Debas simular comportamientos complejos o condiciones de error

**Usa Stubs cuando:**
- Solo necesites que una dependencia devuelva datos específicos
- No te importe cómo se interactúa con la dependencia
- Quieras simplificar una integración compleja para pruebas

Estas técnicas son fundamentales para crear pruebas unitarias verdaderamente aisladas y confiables.

## Pruebas de Componentes: Validando la UI

Las pruebas de componentes verifican que los elementos de la interfaz de usuario funcionen correctamente, incluyendo su renderizado y respuesta a interacciones del usuario.

### Ejemplo con React y React Testing Library:

```typescript
// src/components/UserDisplay.tsx
import React from 'react';

interface UserDisplayProps {
  user: {
    name: string;
    email: string;
  };
}

const UserDisplay: React.FC<UserDisplayProps> = ({ user }) => {
  return (
    <div className="user-display">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
};

export default UserDisplay;

// tests/integration/UserDisplay.test.tsx (usando React Testing Library)
import React from 'react';
import { render, screen } from '@testing-library/react';
import UserDisplay from '../src/components/UserDisplay';

describe('UserDisplay Component', () => {
  test('displays user information correctly', () => {
    const user = { name: 'Test User', email: 'test@example.com' };
    
    render(<UserDisplay user={user} />);
    
    expect(screen.getByText('Test User')).toBeInTheDocument();
    expect(screen.getByText('test@example.com')).toBeInTheDocument();
  });
});
```

### Características de las Pruebas de Componentes:

- **Enfoque en la UI:** Verifican que los componentes se rendericen correctamente y respondan apropiadamente a los eventos del usuario.
- **Simulación de Interacciones:** Prueban cómo reaccionan los componentes a clics, entradas de texto y otros eventos del usuario.
- **Aserciones Basadas en el DOM:** Comprueban la estructura y el contenido del DOM resultante.

### Herramientas comunes: React Testing Library, Vue Testing Library, Angular Testing Library, Enzyme.

## Pruebas de Integración: Validando la Colaboración entre Componentes

Las pruebas de integración verifican que diferentes partes de la aplicación (módulos, servicios, componentes) funcionen correctamente juntas. Detectan problemas en las interfaces y en la comunicación entre componentes.

### Ejemplo de Prueba de Integración para una API:

```typescript
// src/app.js (servidor Express simple)
const express = require('express');
const app = express();

app.use(express.json());
app.get('/api/users/:id', (req, res) => {
  if (req.params.id === '1') {
    return res.status(200).json({ id: 1, name: 'Test User', email: 'test@example.com' });
  }
  res.status(404).json({ message: 'User not found' });
});

describe('User API Integration', () => {
  test('GET /api/users/:id should return a user if found', async () => {
    const res = await request(app).get('/api/users/1');
    expect(res.statusCode).toEqual(200);
    expect(res.body).toHaveProperty('name', 'Test User');
  });
  
  test('GET /api/users/:id should return 404 if not found', async () => {
    const res = await request(app).get('/api/users/999');
    expect(res.statusCode).toEqual(404);
    expect(res.body).toHaveProperty('message', 'User not found');
  });
});
```

### Características de las Pruebas de Integración:

- **Múltiples Componentes:** Prueban cómo interactúan varios componentes entre sí.
- **Mayor Alcance:** Abarcan más código que las pruebas unitarias, pero menos que las E2E.
- **Posible Uso de Servicios Reales:** A veces utilizan bases de datos reales, API, etc., aunque pueden usar mocks o stubs.

### Herramientas comunes: Supertest, Jest, Cypress (para pruebas de integración frontend).

## Pruebas End-to-End (E2E): Validando Flujos Completos

Las pruebas E2E simulan el comportamiento real del usuario interactuando con la aplicación como lo haría un usuario final. Son las pruebas más completas pero también las más lentas y frágiles.

### Ejemplo de una Prueba E2E para un Flujo de Login:

```typescript
// tests/e2e/login.spec.ts (usando Playwright)
import { test, expect } from '@playwright/test';

test.describe('Login Flow', () => {
  test('should allow a user to log in successfully', async ({ page }) => {
    // Navegar a la página de login
    await page.goto('http://localhost:3000/login');

    // Rellenar el formulario de login
    await page.fill('input[name="email"]', 'user@example.com');
    await page.fill('input[name="password"]', 'password123');
    
    // Enviar el formulario
    await page.click('button[type="submit"]');
    
    // Verificar que el usuario fue redirigido al dashboard
    await expect(page).toHaveURL('http://localhost:3000/dashboard');
    
    // Verificar que se muestra el nombre del usuario
    await expect(page.locator('text=Bienvenido, Usuario')).toBeVisible();
  });
  
  test('should show error message for invalid credentials', async ({ page }) => {
    // Navegar a la página de login
    await page.goto('http://localhost:3000/login');
    
    // Rellenar el formulario de login con credenciales incorrectas
    await page.fill('input[name="email"]', 'wrong@example.com');
    await page.fill('input[name="password"]', 'wrongpassword');
    
    // Enviar el formulario
    await page.click('button[type="submit"]');
    
    // Verificar que se muestra un mensaje de error
    await expect(page.locator('text=Credenciales inválidas')).toBeVisible();
    // Asegurarse de que no se redirigió al dashboard
    await expect(page).toHaveURL('http://localhost:3000/login');
  });
});
```

### Características de las Pruebas E2E:

- **Simulan Interacción Real:** Automatizan acciones de usuario como navegación, clics y entrada de datos.
- **Prueban el Sistema Completo:** Verifican que todos los componentes de la aplicación funcionen correctamente juntos.
- **Son Lentas:** Requieren arrancar un navegador real y simular acciones completas.
- **Sensibles a Cambios:** Pueden romperse fácilmente con cambios en la UI.

### Herramientas comunes: Playwright, Cypress, Selenium.

## Pruebas de Rendimiento/Carga

Las pruebas de rendimiento y carga evalúan la capacidad de respuesta, estabilidad, escalabilidad y uso de recursos de una aplicación bajo una carga de trabajo específica.

### Tipos de Pruebas de Rendimiento:

- **Pruebas de Carga:** Evalúan el comportamiento del sistema bajo una carga esperada o pico.
- **Pruebas de Estrés:** Determinan los límites del sistema sometiéndolo a cargas extremas.
- **Pruebas de Escalabilidad:** Verifican cómo se comporta el sistema al aumentar la carga gradualmente.
- **Pruebas de Resistencia:** Verifican la estabilidad del sistema durante largos periodos de tiempo bajo carga constante.

### Métricas Clave:

- **Tiempo de Respuesta:** Tiempo que tarda el sistema en responder a una solicitud.
- **Throughput:** Número de transacciones por segundo que puede manejar el sistema.
- **Utilización de Recursos:** CPU, memoria, disco y uso de red.
- **Concurrencia:** Número máximo de usuarios simultáneos que puede soportar el sistema.

### Herramientas comunes: JMeter, k6, Locust, Gatling, Artillery.

## La Pirámide de Pruebas: Estrategia de Testing Eficiente

La pirámide de pruebas es un concepto que ilustra la proporción ideal de diferentes tipos de pruebas en una estrategia de testing efectiva.

### Estructura de la Pirámide:

- **Base (más grande):** Pruebas Unitarias: Rápidas, aisladas, cubren el mayor número de casos posibles.
- **Medio:** Pruebas de Integración/Componentes: Verifican la interacción entre componentes, con un alcance medio.
- **Cima (más pequeña):** Pruebas End-to-End: El menor número de pruebas deben ser E2E. Son las más lentas y costosas, y se reservan para los flujos de usuario más críticos.

Esta pirámide enfatiza la importancia de la automatización y la detección temprana de errores, ya que los bugs son más baratos de corregir cuanto antes se detectan.

## Ice Cream Cone Anti-Pattern

El "Ice Cream Cone" es un anti-patrón común donde la mayoría de las pruebas son E2E y manuales, con pocas pruebas de integración y aún menos pruebas unitarias. Esto conduce a:

- Detección tardía de bugs
- Ciclos de feedback lentos
- Alta fragilidad de las pruebas
- Costes de mantenimiento elevados
- Desarrollo más lento

## Integración Continua y Testing

Integrar las pruebas en un sistema de Integración Continua (CI) garantiza que todas las pruebas se ejecuten automáticamente cuando hay cambios en el código, proporcionando feedback inmediato a los desarrolladores.

### Beneficios:

- **Detección Temprana de Problemas:** Los errores se detectan inmediatamente después de introducirse.
- **Integración Frecuente:** Fomenta que los desarrolladores integren su código con frecuencia.
- **Calidad Consistente:** Asegura que el código siempre cumpla con los estándares de calidad.
- **Menor Tiempo de Resolución:** Los bugs son más fáciles de corregir cuando se detectan cerca de su introducción.

### Herramientas de CI: GitHub Actions, GitLab CI, Jenkins, CircleCI.

## Mejores Prácticas en Testing

- **Escribir Pruebas Claras y Legibles:** Las pruebas deben ser fáciles de entender, casi como documentación del código.

- **Probar Casos de Éxito y de Error:** No solo pruebes el camino feliz; asegúrate de que tu código maneje correctamente los errores y las entradas inválidas.

- **Usar Fixtures y Factories:** Crea datos de prueba consistentes y reutilizables para tus pruebas.

- **Evitar Pruebas Frágiles:** Diseña tus pruebas para que sean resilientes a cambios menores en la implementación.

- **Aislar las Pruebas:** Cada prueba debe ser independiente y no depender del estado dejado por otras pruebas.

- **Mantener las Pruebas Rápidas:** Especialmente las unitarias, que deberían ejecutarse en segundos, no en minutos.

- **Monitorear la Cobertura:** La cobertura de código proporciona una métrica útil, aunque no perfecta, para evaluar la efectividad de tus pruebas.

- **Revisar y Refactorizar las Pruebas:** Como cualquier código, las pruebas deben ser refactorizadas para mantener su calidad.

- **Integrar en el Flujo de Trabajo:** Las pruebas deben ejecutarse automáticamente en CI/CD y antes de hacer push a repositorios compartidos.

- **Usar Test-Driven Development (TDD):** Escribir pruebas antes que el código de producción puede mejorar el diseño y la calidad general.

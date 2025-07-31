# Capítulo 6: React y Next.js (Continuación)

## 6.3 Características Avanzadas de Next.js

### 6.3.1 Optimización de Imágenes

Next.js incluye un componente `Image` optimizado que mejora automáticamente el rendimiento.

```jsx
import Image from 'next/image';

export default function ProfilePage() {
  return (
    <div>
      <h1>Mi Perfil</h1>
      <Image
        src="/profile.jpg"     // Ruta relativa a /public
        alt="Foto de perfil"
        width={300}
        height={300}
        priority        // Carga prioritaria
        quality={85}    // Calidad de la imagen (1-100)
      />
      
      {/* Imagen remota (requiere configuración en next.config.js) */}
      <Image
        src="https://example.com/profile.jpg"
        alt="Foto remota"
        width={300}
        height={300}
      />
    </div>
  );
}
```

Configuración en `next.config.js` para imágenes remotas:

```javascript
module.exports = {
  images: {
    domains: ['example.com'], // Dominios permitidos
    // O usar un loader personalizado:
    loader: 'custom',
    loaderFile: './my-loader.js',
  },
}
```

### 6.3.2 Optimización de Fuentes

Next.js 13+ incluye el componente `next/font` que optimiza la carga de fuentes.

```jsx
// app/layout.js
import { Inter } from 'next/font/google';

// Cargar fuente Inter y definir subconjuntos
const inter = Inter({ 
  subsets: ['latin'],
  display: 'swap',
  weight: ['400', '700'],
});

export default function RootLayout({ children }) {
  return (
    <html lang="es" className={inter.className}>
      <body>{children}</body>
    </html>
  );
}
```

Uso de fuentes locales:

```jsx
import localFont from 'next/font/local';

// Carga de una fuente local
const myFont = localFont({
  src: [
    {
      path: '../public/fonts/my-font.woff2',
      weight: '400',
      style: 'normal',
    },
    {
      path: '../public/fonts/my-font-bold.woff2',
      weight: '700',
      style: 'normal',
    },
  ],
});

export default function MyComponent() {
  return (
    <div className={myFont.className}>
      Texto con mi fuente personalizada
    </div>
  );
}
```

### 6.3.3 Internacionalización (i18n)

#### Configuración de i18n en Pages Router

```javascript
// next.config.js
module.exports = {
  i18n: {
    // Idiomas soportados
    locales: ['es', 'en', 'fr'],
    // Idioma por defecto
    defaultLocale: 'es',
    // Rutas de detección automática de dominio
    domains: [
      {
        domain: 'example.es',
        defaultLocale: 'es',
      },
      {
        domain: 'example.com',
        defaultLocale: 'en',
      },
      {
        domain: 'example.fr',
        defaultLocale: 'fr',
      },
    ],
  },
}
```

Uso de traducciones:

```jsx
// pages/index.js
import { useRouter } from 'next/router';

// Diccionario simple de traducciones
const translations = {
  es: {
    greeting: 'Hola mundo',
    description: 'Bienvenido a mi sitio web',
  },
  en: {
    greeting: 'Hello world',
    description: 'Welcome to my website',
  },
  fr: {
    greeting: 'Bonjour le monde',
    description: 'Bienvenue sur mon site web',
  },
};

export default function Home() {
  const router = useRouter();
  const { locale } = router;
  const t = translations[locale];

  return (
    <div>
      <h1>{t.greeting}</h1>
      <p>{t.description}</p>
      
      <div>
        <button onClick={() => router.push('/', '/', { locale: 'es' })}>
          Español
        </button>
        <button onClick={() => router.push('/', '/', { locale: 'en' })}>
          English
        </button>
        <button onClick={() => router.push('/', '/', { locale: 'fr' })}>
          Français
        </button>
      </div>
    </div>
  );
}
```

#### i18n en App Router

Con App Router, se recomienda usar directorios para idiomas:

```
app/
├── [lang]/              # Parámetro dinámico para el idioma
│   ├── layout.js        # Layout con detección de idioma
│   ├── page.js          # Página principal
│   └── about/
│       └── page.js      # Página "Acerca de"
├── api/
└── ...
```

Ejemplo de implementación:

```jsx
// app/[lang]/layout.js
export async function generateStaticParams() {
  // Define los idiomas soportados
  return [
    { lang: 'es' },
    { lang: 'en' },
    { lang: 'fr' },
  ];
}

// Diccionario de traducciones
const dictionaries = {
  es: () => import('../dictionaries/es.json').then(m => m.default),
  en: () => import('../dictionaries/en.json').then(m => m.default),
  fr: () => import('../dictionaries/fr.json').then(m => m.default),
};

export default async function Layout({ children, params }) {
  // Carga el diccionario para el idioma actual
  const dict = await dictionaries[params.lang]();
  
  return (
    <html lang={params.lang}>
      <body>
        {/* Pasa el diccionario a los componentes hijos */}
        {children}
      </body>
    </html>
  );
}

// app/[lang]/page.js
export default function Home({ params }) {
  // Obtiene traducciones en el componente
  const dict = useTranslations(params.lang);
  
  return (
    <div>
      <h1>{dict.greeting}</h1>
      <p>{dict.description}</p>
    </div>
  );
}
```

### 6.3.4 Middleware

El middleware permite ejecutar código antes de que se complete una solicitud.

```javascript
// middleware.js (en la raíz del proyecto)
import { NextResponse } from 'next/server';

export function middleware(request) {
  // Ejemplo 1: Redirección
  if (request.nextUrl.pathname === '/old-page') {
    return NextResponse.redirect(new URL('/new-page', request.url));
  }
  
  // Ejemplo 2: Reescritura (muestra una página pero mantiene la URL original)
  if (request.nextUrl.pathname === '/blog') {
    return NextResponse.rewrite(new URL('/blog-temp-down', request.url));
  }
  
  // Ejemplo 3: Verificación de autenticación
  const authCookie = request.cookies.get('authToken');
  if (
    request.nextUrl.pathname.startsWith('/dashboard') && 
    !authCookie
  ) {
    // Redirecciona al login y guarda la URL original como param
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('from', request.nextUrl.pathname);
    return NextResponse.redirect(loginUrl);
  }
  
  // Continúa con la solicitud normal
  return NextResponse.next();
}

// Configurar qué rutas deben usar el middleware
export const config = {
  matcher: ['/dashboard/:path*', '/blog/:path*', '/old-page'],
};
```

### 6.3.5 Edge Runtime

Next.js permite ejecutar código en el edge (cerca del usuario) para mejor rendimiento.

```javascript
// app/api/edge/route.js
export const runtime = 'edge';

export async function GET(request) {
  const { searchParams } = new URL(request.url);
  const name = searchParams.get('name') || 'World';
  
  return new Response(`Hello, ${name}!`, {
    headers: {
      'content-type': 'text/plain',
    },
  });
}
```

Componentes Edge:

```jsx
// app/edge-page/page.js
export const runtime = 'edge';

export default async function Page() {
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();
  
  return (
    <div>
      <h1>Edge Page</h1>
      <p>Datos desde el edge: {data.message}</p>
    </div>
  );
}
```

### 6.3.6 Streaming y Componentes Suspense

Next.js 13+ permite usar Streaming SSR y Suspense para renderizado progresivo.

```jsx
// app/streaming/page.js
import { Suspense } from 'react';

// Componente que carga datos lentamente
async function SlowDataComponent() {
  // Simula una carga lenta
  await new Promise(resolve => setTimeout(resolve, 2000));
  
  const data = await fetch('https://api.example.com/data');
  const json = await data.json();
  
  return <div>{json.message}</div>;
}

export default function StreamingPage() {
  return (
    <div>
      <h1>Página con Streaming</h1>
      
      {/* Contenido que se muestra inmediatamente */}
      <p>Este contenido aparece al instante</p>
      
      {/* Contenido que se carga progresivamente */}
      <Suspense fallback={<div>Cargando datos...</div>}>
        <SlowDataComponent />
      </Suspense>
      
      {/* Otro componente lento */}
      <Suspense fallback={<div>Cargando comentarios...</div>}>
        <SlowComments />
      </Suspense>
    </div>
  );
}
```

### 6.3.7 Manejo Avanzado de Estado con Context API y Redux

#### Context API en Next.js

```jsx
// contexts/ThemeContext.js
'use client'; // Necesario en App Router para componentes cliente

import { createContext, useContext, useState } from 'react';

// Crear contexto
const ThemeContext = createContext();

// Hook personalizado para usar el contexto
export function useTheme() {
  return useContext(ThemeContext);
}

// Proveedor del contexto
export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prevTheme => (prevTheme === 'light' ? 'dark' : 'light'));
  };
  
  const value = {
    theme,
    toggleTheme
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```

Uso en App Router:

```jsx
// app/layout.js
import { ThemeProvider } from '../contexts/ThemeContext';

export default function RootLayout({ children }) {
  return (
    <html lang="es">
      <body>
        <ThemeProvider>
          {children}
        </ThemeProvider>
      </body>
    </html>
  );
}

// app/page.js
'use client'; // Necesario para usar hooks

import { useTheme } from '../contexts/ThemeContext';

export default function Home() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <div className={theme === 'dark' ? 'dark-mode' : ''}>
      <h1>Tema: {theme}</h1>
      <button onClick={toggleTheme}>Cambiar tema</button>
    </div>
  );
}
```

#### Redux con Next.js

```jsx
// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';
import userReducer from './userSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer,
    user: userReducer,
  },
});

// store/counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: state => {
      state.value += 1;
    },
    decrement: state => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
  },
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export default counterSlice.reducer;
```

Integración en Next.js con App Router:

```jsx
// app/StoreProvider.jsx
'use client';

import { Provider } from 'react-redux';
import { store } from '../store';

export default function StoreProvider({ children }) {
  return <Provider store={store}>{children}</Provider>;
}

// app/layout.js
import StoreProvider from './StoreProvider';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <StoreProvider>
          {children}
        </StoreProvider>
      </body>
    </html>
  );
}

// app/counter/page.js
'use client';

import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement } from '../../store/counterSlice';

export default function CounterPage() {
  const count = useSelector(state => state.counter.value);
  const dispatch = useDispatch();
  
  return (
    <div>
      <h1>Contador: {count}</h1>
      <button onClick={() => dispatch(increment())}>
        Incrementar
      </button>
      <button onClick={() => dispatch(decrement())}>
        Decrementar
      </button>
    </div>
  );
}
```

### 6.3.8 Testing en Next.js

#### Pruebas Unitarias con Jest y React Testing Library

```javascript
// __tests__/Home.test.js
import { render, screen } from '@testing-library/react';
import Home from '../pages/index';

describe('Home page', () => {
  it('renders a heading', () => {
    render(<Home />);
    
    const heading = screen.getByRole('heading', {
      name: /welcome to next\.js/i,
    });
    
    expect(heading).toBeInTheDocument();
  });
  
  it('renders a button', () => {
    render(<Home />);
    
    const button = screen.getByRole('button', {
      name: /get started/i,
    });
    
    expect(button).toBeInTheDocument();
  });
});
```

Configuración en `jest.config.js`:

```javascript
const nextJest = require('next/jest');

const createJestConfig = nextJest({
  // Ruta al directorio de Next.js
  dir: './',
});

// Configuración personalizada de Jest
const customJestConfig = {
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  testEnvironment: 'jest-environment-jsdom',
  moduleNameMapper: {
    '^@/components/(.*)$': '<rootDir>/components/$1',
    '^@/pages/(.*)$': '<rootDir>/pages/$1',
  },
};

// exporta esta configuración
module.exports = createJestConfig(customJestConfig);
```

#### Pruebas End-to-End con Cypress

```javascript
// cypress/e2e/home.cy.js
describe('Home Page', () => {
  beforeEach(() => {
    cy.visit('http://localhost:3000');
  });

  it('displays the heading', () => {
    cy.get('h1').contains('Welcome to Next.js');
  });

  it('navigates to about page', () => {
    cy.get('a[href="/about"]').click();
    cy.url().should('include', '/about');
    cy.get('h1').contains('About Us');
  });

  it('submits the form correctly', () => {
    cy.get('input[name="email"]').type('test@example.com');
    cy.get('button[type="submit"]').click();
    cy.get('.success-message').should('be.visible');
  });
});
```

### 6.3.9 Despliegue de Aplicaciones Next.js

#### Vercel (Plataforma Nativa)

```bash
# Instalar CLI de Vercel
npm i -g vercel

# Login en Vercel
vercel login

# Despliegue de desarrollo
vercel

# Despliegue de producción
vercel --prod
```

Configuración con `vercel.json`:

```json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/next"
    }
  ],
  "routes": [
    {
      "src": "/api/legacy/(.*)",
      "dest": "/api/v1/$1"
    }
  ],
  "env": {
    "CUSTOM_ENV_VAR": "production-value"
  }
}
```

#### Despliegue en AWS (Amazon Web Services)

```bash
# Construcción de la aplicación
npm run build

# Configuración básica en AWS Amplify
amplify init
amplify add hosting
amplify publish
```

#### Despliegue con Docker

```dockerfile
# Dockerfile
FROM node:18-alpine AS base

# Instalar dependencias
FROM base AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# Construir la aplicación
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Imagen de producción
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

Configuración en `next.config.js` para Docker:

```javascript
module.exports = {
  output: 'standalone',
};
```

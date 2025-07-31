# Capítulo 6: React y Next.js (Continuación)

## 6.2 Next.js: El Framework de React

Next.js es un framework de React que proporciona estructura, características y optimizaciones para aplicaciones web. Creado por Vercel, Next.js simplifica el desarrollo de aplicaciones React complejas con funcionalidades como renderizado del lado del servidor (SSR), generación estática (SSG), y enrutamiento basado en sistema de archivos. Este capítulo cubre Next.js 15, la versión más reciente.

### 6.2.1 Introducción a Next.js

Next.js resuelve muchos desafíos comunes en el desarrollo web moderno:

- **Renderizado**: Ofrece diferentes estrategias de renderizado adaptadas a cada página.
- **Enrutamiento**: Sistema de enrutamiento basado en archivos, simple e intuitivo.
- **Compilación**: Optimización automática del código para producción.
- **Performance**: Optimizaciones como división de código, precarga de rutas y más.
- **Desarrollo**: Recarga en caliente y mejores mensajes de error para el desarrollo.

### 6.2.2 Configuración Inicial

#### Creación de un Proyecto Next.js

```bash
# Usando create-next-app (recomendado)
npx create-next-app mi-proyecto
# o
yarn create next-app mi-proyecto

# Con TypeScript
npx create-next-app@latest --typescript mi-proyecto
```

#### Estructura Básica de un Proyecto

```
mi-proyecto/
├── .next/               # Directorio de compilación (generado automáticamente)
├── node_modules/        # Dependencias
├── public/              # Archivos estáticos
│   ├── favicon.ico
│   └── images/
├── pages/               # Páginas de la aplicación
│   ├── _app.js          # Componente principal de la aplicación
│   ├── _document.js     # Personalización del documento HTML
│   ├── index.js         # Página principal
│   └── api/             # Rutas de API
│       └── hello.js
├── styles/              # Estilos CSS
├── components/          # Componentes reutilizables
├── .gitignore
├── next.config.js       # Configuración de Next.js
├── package.json
└── README.md
```

#### Estructura de Directorios en Next.js 15

En Next.js 15, la estructura recomendada y predeterminada usa el directorio `app` para el nuevo App Router:

```
mi-proyecto/
│── app/                 # Directorio principal para el App Router (predeterminado en Next.js 15)
│   │── layout.js        # Diseño compartido entre páginas
│   │── page.js          # Página principal
│   │── about/
│   │   └── page.js      # Ruta /about
│   │── dashboard/
│   │   │── layout.js    # Diseño específico para el dashboard
│   │   │── page.js      # Página principal del dashboard
│   │   └── [id]/        # Ruta dinámica
│   │       └── page.js  # Página de detalle con parámetro dinámico
│   └── api/             # Rutas API con el App Router
│       └── route.js     # Manejador de ruta API
│── components/          # Componentes reutilizables
│── public/              # Archivos estáticos
└── ...                  # Otros archivos de configuración
```

### 6.2.3 Sistema de Enrutamiento: App Router vs Pages Router

#### Comparación entre App Router y Pages Router

Next.js 15 ofrece dos sistemas de enrutamiento diferentes, cada uno con sus propias características y casos de uso. El App Router es la opción recomendada para nuevos proyectos, mientras que Pages Router se mantiene por razones de compatibilidad.

| Característica | App Router (app/) | Pages Router (pages/) |
|---------------|------------------|----------------------|
| Modelo de renderizado | React Server Components (RSC) | Client-side components |
| Componentes por defecto | Componentes de servidor | Componentes de cliente |
| Archivo de definición | page.js | [nombre].js |
| Layouts | Sistema jerárquico con layout.js | Componentes personalizados en _app.js |
| Rutas dinámicas | [param]/page.js o [...param]/page.js | [param].js o [...param].js |
| Rutas API | route.js con métodos HTTP | api/[ruta].js con handler(req, res) |
| Data Fetching | fetch() con opciones de cache/revalidate | getServerSideProps/getStaticProps |
| Streaming | Suspense y streaming nativo | No soportado nativamente |
| Metadata | Exports de metadata y generateMetadata | Componente Head de next/head |
| Interacciones cliente | Necesita directiva 'use client' | Todo es cliente por defecto |

##### Ventajas del App Router

- **React Server Components**: Reduce el JavaScript enviado al cliente
- **Layouts anidados**: Más fácil crear interfaces complejas con layouts compartidos
- **Fetching de datos colocado**: Los datos se pueden obtener directamente en los componentes
- **Streaming y carga progresiva**: Mejor UX durante cargas de datos
- **Estructura intuitiva**: Los archivos page.js, layout.js, loading.js tienen roles claros
- **Mejoras de caché**: Control granular sobre el caché y revalidación

##### Ventajas del Pages Router

- **Madurez**: Sistema más probado y con más recursos/ejemplos disponibles
- **Simplicidad**: Para aplicaciones pequeñas puede ser más directo
- **Compatibilidad**: Algunas librerías legacy funcionan mejor con Pages Router
- **Familiaridad**: Desarrolladores con experiencia en Next.js ya lo conocen

#### Migración de Pages Router a App Router

Next.js 15 permite la coexistencia de ambos sistemas, facilitando la migración gradual:

```
mi-proyecto/
├── app/                 # App Router (nuevas funcionalidades)
│   └── nueva-seccion/
│       └── page.js
└── pages/               # Pages Router (código existente)
    └── index.js
```

#### Enrutamiento Basado en Sistema de Archivos (Pages Router)

En Next.js tradicional, las rutas se definen por la estructura de archivos en el directorio `pages`.

```
pages/
├── index.js             # Ruta: /
├── about.js             # Ruta: /about
└── blog/
    ├── index.js         # Ruta: /blog
    └── [slug].js        # Ruta dinámica: /blog/:slug
```

Ejemplo de página básica:

```jsx
// pages/index.js
export default function Home() {
  return (
    <div>
      <h1>Bienvenido a mi sitio</h1>
    </div>
  );
}
```

#### Rutas Dinámicas

Las rutas dinámicas permiten crear páginas que pueden manejar diferentes datos basados en parámetros en la URL.

```jsx
// pages/blog/[slug].js
import { useRouter } from 'next/router';

export default function BlogPost() {
  const router = useRouter();
  const { slug } = router.query;

  return (
    <div>
      <h1>Post: {slug}</h1>
    </div>
  );
}
```

#### Enrutamiento con App Router (Next.js 13+)

El nuevo App Router utiliza convenciones de carpetas y archivos para definir rutas:

```
app/
├── page.js              # Ruta: /
├── about/
│   └── page.js          # Ruta: /about
└── blog/
    ├── page.js          # Ruta: /blog
    └── [slug]/
        └── page.js      # Ruta dinámica: /blog/:slug
```

Ejemplo con el App Router:

```jsx
// app/page.js
export default function Home() {
  return (
    <div>
      <h1>Bienvenido a mi sitio con App Router</h1>
    </div>
  );
}
```

Manejo de rutas dinámicas:

```jsx
// app/blog/[slug]/page.js
export default function BlogPost({ params }) {
  // Los parámetros se pasan como props
  const { slug } = params;

  return (
    <div>
      <h1>Post: {slug}</h1>
    </div>
  );
}
```

#### Navegación entre Páginas

Next.js proporciona el componente `Link` para navegación del lado del cliente sin recargar la página.

```jsx
import Link from 'next/link';

export default function Navigation() {
  return (
    <nav>
      <ul>
        <li>
          <Link href="/">Inicio</Link>
        </li>
        <li>
          <Link href="/about">Acerca de</Link>
        </li>
        <li>
          {/* Ruta dinámica con parámetro */}
          <Link href="/blog/hello-world">
            Blog Post
          </Link>
        </li>
        <li>
          {/* Con objeto de configuración */}
          <Link
            href={{
              pathname: '/blog/[slug]',
              query: { slug: 'hello-world' },
            }}
          >
            Blog Post (Objeto)
          </Link>
        </li>
      </ul>
    </nav>
  );
}
```

#### Navegación Programática

También es posible navegar usando el hook `useRouter`.

```jsx
import { useRouter } from 'next/router';

export default function LoginForm() {
  const router = useRouter();
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Lógica de login...
    const success = await loginUser(/* credenciales */);
    
    if (success) {
      // Redirecciona al dashboard después del login
      router.push('/dashboard');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Campos del formulario */}
      <button type="submit">Iniciar sesión</button>
    </form>
  );
}
```

Con App Router, se usa `useRouter` de `next/navigation`:

```jsx
'use client';

import { useRouter } from 'next/navigation';

export default function LoginForm() {
  const router = useRouter();
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    // Lógica de login...
    router.push('/dashboard');
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Formulario de login */}
    </form>
  );
}
```

### 6.2.4 Estrategias de Renderizado

#### Estrategias de Renderizado en Next.js 15

Next.js 15 ofrece varias estrategias de renderizado que se pueden elegir según los requisitos de cada ruta de la aplicación:

| Estrategia | Descripción | Uso Recomendado |
|-----------|-------------|------------------|
| Static Rendering (predeterminado) | Renderizado en tiempo de build | Contenido que no cambia frecuentemente |
| Dynamic Rendering | Renderizado en tiempo de solicitud | Contenido personalizado o dependiente de cookies/headers |
| Streaming | Renderizado progresivo | Interfaces complejas con distintas prioridades de carga |

Con el App Router, la elección entre estas estrategias es más granular y puede aplicarse a nivel de componente o ruta.

#### Renderizado del Lado del Servidor (SSR)

Las páginas se generan en el servidor en cada solicitud.

```jsx
// En Pages Router (pages/ssr-page.js)
export default function Page({ data }) {
  return <div>Datos: {data}</div>;
}

// Esta función se ejecuta en cada solicitud
export async function getServerSideProps() {
  // Fetch de datos en el servidor
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();
  
  // Pasar los datos como props
  return {
    props: { data },
  };
}
```

En App Router, el SSR se realiza de forma nativa usando la función `fetch` con la opción `{ cache: 'no-store' }`:

```jsx
// En App Router (app/ssr-page/page.js)
async function getData() {
  // No caché - se ejecuta en cada solicitud (equivalente a getServerSideProps)
  const res = await fetch('https://api.example.com/data', { cache: 'no-store' });
  if (!res.ok) throw new Error('Failed to fetch data');
  return res.json();
}

// Los componentes de página en app/ son de servidor por defecto
export default async function Page() {
  // Componentes de servidor pueden ser async
  const data = await getData();
  
  return <div>Datos: {data}</div>;
}
```

#### Generación Estática (SSG)

El contenido se genera durante la compilación y se sirve como HTML estático. En Next.js 15, esta es la estrategia de renderizado predeterminada para la mayoria de los casos.

##### En Pages Router:

```jsx
// pages/ssg-page.js
export default function Page({ data }) {
  return <div>Datos estáticos: {data}</div>;
}

// Esta función se ejecuta SOLO en tiempo de construcción
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();
  
  return {
    props: { data },
  };
}
```

##### En App Router:

En App Router, la generación estática es el comportamiento predeterminado. Cualquier componente que no use datos dinámicos o caché específico se genera estáticamente durante la compilación:

```jsx
// app/ssg-page/page.js
async function getData() {
  // Por defecto, fetch() cachea automáticamente en tiempo de build
  const res = await fetch('https://api.example.com/data');
  if (!res.ok) throw new Error('Failed to fetch data');
  return res.json();
}

export default async function Page() {
  const data = await getData();
  
  return <div>Datos estáticos: {data}</div>;
}
```

#### Revalidación Incremental Estática (ISR)

##### En Pages Router:

ISR combina las ventajas de SSG y SSR, permitiendo regenerar páginas estáticas en segundo plano.

```jsx
// pages/isr-page.js
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();
  
  return {
    props: { data },
    // Revalidación incremental: actualizar cada 60 segundos
    revalidate: 60,
  };
}
```

##### En App Router (Next.js 15):

```jsx
// app/isr-page/page.js
async function getData() {
  // Revalidación incremental con la opción next.revalidate
  const res = await fetch('https://api.example.com/data', {
    next: { revalidate: 60 }, // Revalidar cada 60 segundos
  });
  if (!res.ok) throw new Error('Failed to fetch data');
  return res.json();
}

export default async function Page() {
  const data = await getData();
  return <div>Datos revalidados: {data.time}</div>;
}
```

Esta implementación es más granular en Next.js 15, permitiendo configurar diferentes tiempos de revalidación para distintas solicitudes dentro del mismo componente.

#### Revalidación bajo Demanda

Permite revalidar manualmente una página generada estáticamente cuando ocurren eventos específicos como actualizaciones en una base de datos.

##### En Pages Router:

```jsx
// pages/api/revalidate.js
export default async function handler(req, res) {
  try {
    // Verificar secreto para prevenir revalidación no autorizada
    if (req.query.secret !== process.env.REVALIDATION_TOKEN) {
      return res.status(401).json({ message: 'Invalid token' });
    }

    const path = req.query.path;
    
    // Revalidar la ruta
    await res.revalidate(path);
    
    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).json({ message: 'Error revalidating' });
  }
}
```

##### En App Router (Next.js 15):

Next.js 15 proporciona las APIs `revalidatePath` y `revalidateTag` para un control más preciso de la revalidación:

```jsx
// app/api/revalidate/route.js
import { revalidatePath, revalidateTag } from 'next/cache';
import { NextResponse } from 'next/server';

export async function POST(request) {
  const { secret, path, tag } = await request.json();
  
  if (secret !== process.env.REVALIDATION_TOKEN) {
    return NextResponse.json({ message: 'Invalid token' }, { status: 401 });
  }
  
  if (path) {
    // Revalidar por ruta
    revalidatePath(path);
    return NextResponse.json({ revalidated: true, path });
  } else if (tag) {
    // Revalidar por etiqueta de caché
    revalidateTag(tag);
    return NextResponse.json({ revalidated: true, tag });
  }
  
  return NextResponse.json(
    { message: 'Path or tag parameter missing' },
    { status: 400 }
  );
}
```

### 6.2.5 Middleware y Edge Runtime en Next.js 15

Next.js 15 mejora significativamente las capacidades de middleware y Edge Runtime para aplicaciones modernas.

#### Middleware

El middleware permite ejecutar código antes de que una solicitud se complete, ideal para autenticación, redirecciones, reescritura de rutas, A/B testing y más.

```jsx
// middleware.js - se ejecuta en todas las rutas
import { NextResponse } from 'next/server';

export function middleware(request) {
  // Obtener la ruta solicitada
  const path = request.nextUrl.pathname;
  
  // Verificar si el usuario está autenticado (ejemplo con cookie)
  const isAuthenticated = request.cookies.has('authToken');
  
  // Redireccionar a login si intenta acceder a rutas protegidas
  if (path.startsWith('/dashboard') && !isAuthenticated) {
    // Redirección con preservación de la URL original
    const url = new URL('/login', request.url);
    url.searchParams.set('from', path);
    return NextResponse.redirect(url);
  }
  
  // Añadir encabezados personalizados
  if (path.startsWith('/api')) {
    const response = NextResponse.next();
    response.headers.set('x-api-version', '1.0.0');
    return response;
  }
  
  // Continuar con la solicitud normal
  return NextResponse.next();
}

// Configuración para aplicar middleware solo a rutas específicas
export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
};
```

#### Edge Runtime

Next.js 15 mejora el soporte para Edge Runtime, permitiendo ejecutar código más cerca de los usuarios para menor latencia:

```jsx
// app/api/edge/route.js
export const runtime = 'edge'; // Usar Edge Runtime en lugar de Node.js

export async function GET() {
  const data = {
    timestamp: new Date().toISOString(),
    region: process.env.VERCEL_REGION || 'unknown',
  };
  
  return new Response(JSON.stringify(data), {
    status: 200,
    headers: { 'Content-Type': 'application/json' },
  });
}
```

#### Generación Estática con Rutas Dinámicas

```jsx
// pages/posts/[id].js
export default function Post({ post }) {
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}

// Define qué rutas generar en tiempo de construcción
export async function getStaticPaths() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();
  
  // Generar rutas para cada post
  const paths = posts.map((post) => ({
    params: { id: post.id.toString() },
  }));
  
  return {
    paths,
    fallback: 'blocking', // 'blocking', false, o true
  };
}

export async function getStaticProps({ params }) {
  const res = await fetch(`https://api.example.com/posts/${params.id}`);
  const post = await res.json();
  
  return {
    props: { post },
    revalidate: 60, // ISR: revalidar después de 60 segundos
  };
}
```

En App Router:

```jsx
// app/posts/[id]/page.js
// Genera rutas estáticas
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts').then(res => 
    res.json()
  );
  
  return posts.map((post) => ({
    id: post.id.toString(),
  }));
}

async function getPost(id) {
  const post = await fetch(`https://api.example.com/posts/${id}`).then(res =>
    res.json()
  );
  return post;
}

export default async function Post({ params }) {
  const post = await getPost(params.id);
  
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}
```

#### Revalidación Incremental Estática (ISR)

ISR permite regenerar páginas estáticas después de un tiempo especificado.

```jsx
// pages/isr-page.js
export default function Page({ data, time }) {
  return (
    <div>
      <p>Datos: {data}</p>
      <p>Generado en: {time}</p>
    </div>
  );
}

export async function getStaticProps() {
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();
  
  return {
    props: {
      data,
      time: new Date().toISOString(),
    },
    // Revalidar cada 60 segundos
    revalidate: 60,
  };
}
```

En App Router:

```jsx
// app/isr-page/page.js
async function getData() {
  // Por defecto, fetch en App Router hace caché y se comporta como SSG
  const res = await fetch('https://api.example.com/data', {
    next: { revalidate: 60 }, // Revalidar cada 60 segundos
  });
  
  if (!res.ok) throw new Error('Failed to fetch data');
  return res.json();
}

export default async function Page() {
  const data = await getData();
  const time = new Date().toISOString();
  
  return (
    <div>
      <p>Datos: {data}</p>
      <p>Generado en: {time}</p>
    </div>
  );
}
```

#### Revalidación bajo demanda

Next.js permite revalidar páginas específicas mediante una API.

```jsx
// pages/api/revalidate.js
export default async function handler(req, res) {
  // Verifica alguna forma de autenticación
  if (req.query.secret !== process.env.REVALIDATION_TOKEN) {
    return res.status(401).json({ message: 'Invalid token' });
  }

  try {
    // Revalidar la ruta especificada
    const path = req.query.path;
    await res.revalidate(path);
    
    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).json({ message: 'Error revalidating' });
  }
}
```

En App Router:

```jsx
// app/api/revalidate/route.js
import { revalidatePath } from 'next/cache';
import { NextResponse } from 'next/server';

export async function POST(request) {
  const { secret, path } = await request.json();
  
  if (secret !== process.env.REVALIDATION_TOKEN) {
    return NextResponse.json({ message: 'Invalid token' }, { status: 401 });
  }
  
  if (!path) {
    return NextResponse.json(
      { message: 'Path parameter missing' },
      { status: 400 }
    );
  }
  
  try {
    revalidatePath(path);
    return NextResponse.json({ revalidated: true, now: Date.now() });
  } catch (err) {
    return NextResponse.json(
      { message: 'Error revalidating' },
      { status: 500 }
    );
  }
}
```

### 6.2.5 Manejo de Datos

#### API Routes

Next.js permite crear APIs dentro del mismo proyecto.

```jsx
// pages/api/users.js
export default function handler(req, res) {
  if (req.method === 'GET') {
    // Manejar solicitud GET
    res.status(200).json({ users: ['Juan', 'María', 'Pedro'] });
  } else if (req.method === 'POST') {
    // Manejar solicitud POST
    const { name } = req.body;
    // Lógica para crear usuario
    res.status(201).json({ message: `Usuario ${name} creado` });
  } else {
    // Método no permitido
    res.setHeader('Allow', ['GET', 'POST']);
    res.status(405).end(`Method ${req.method} Not Allowed`);
  }
}
```

En App Router:

```jsx
// app/api/users/route.js
import { NextResponse } from 'next/server';

export async function GET() {
  // Manejar solicitud GET
  return NextResponse.json({ users: ['Juan', 'María', 'Pedro'] });
}

export async function POST(request) {
  // Manejar solicitud POST
  const { name } = await request.json();
  // Lógica para crear usuario
  return NextResponse.json(
    { message: `Usuario ${name} creado` },
    { status: 201 }
  );
}
```

#### SWR para Gestión de Estado y Caché

SWR es una librería creada por el equipo de Next.js para el manejo de datos.

```jsx
import useSWR from 'swr';

// Función para hacer fetch de datos
const fetcher = (...args) => fetch(...args).then(res => res.json());

function Profile() {
  const { data, error, isLoading } = useSWR('/api/user', fetcher);

  if (error) return <div>Error al cargar datos</div>;
  if (isLoading) return <div>Cargando...</div>;
  
  return (
    <div>
      <h1>Perfil de {data.name}</h1>
      <p>Correo: {data.email}</p>
    </div>
  );
}
```

#### Mutación y Revalidación con SWR

```jsx
import useSWR, { mutate } from 'swr';

function Profile() {
  const { data, error } = useSWR('/api/user', fetcher);
  
  async function updateUser(newData) {
    // Actualizar el dato local inmediatamente (UI optimista)
    mutate('/api/user', { ...data, ...newData }, false);
    
    // Enviar solicitud al servidor
    await fetch('/api/user', {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(newData),
    });
    
    // Trigger revalidation (recargar datos)
    mutate('/api/user');
  }
  
  return (
    <div>
      {/* Interfaz de usuario */}
      <button
        onClick={() => updateUser({ name: 'Nuevo Nombre' })}
      >
        Actualizar nombre
      </button>
    </div>
  );
}
```

### 6.2.6 Server Actions y Gestión de Estado en Next.js 15

Next.js 15 introduce Server Actions, una característica que permite ejecutar funciones asíncronas directamente en el servidor desde el cliente, simplificando operaciones como mutaciones de datos y formularios.

#### Server Actions

```jsx
// app/actions.js
'use server';

import { revalidatePath } from 'next/cache';

// Esta función se ejecuta en el servidor cuando se llama desde el cliente
export async function createTodo(formData) {
  const title = formData.get('title');
  
  // Validar en el servidor
  if (!title || title.length < 3) {
    return { error: 'El título debe tener al menos 3 caracteres' };
  }
  
  try {
    // Crear nuevo todo en base de datos
    await db.todos.create({ title, completed: false });
    
    // Revalidar la página para mostrar el nuevo todo
    revalidatePath('/todos');
    
    return { success: true };
  } catch (error) {
    return { error: 'Error al crear el todo' };
  }
}
```

#### Uso de Server Actions en formularios

```jsx
// app/todos/page.js
import { createTodo } from '../actions';

export default function TodosPage() {
  return (
    <div>
      <h1>Mis Tareas</h1>
      
      {/* El formulario envía los datos directamente al servidor */}
      <form action={createTodo}>
        <input type="text" name="title" placeholder="Nueva tarea..." />
        <button type="submit">Añadir</button>
      </form>
      
      {/* Lista de tareas */}
    </div>
  );
}
```

#### Optimistic UI con useOptimistic

```jsx
'use client';
import { useOptimistic } from 'react';
import { createTodo } from '../actions';

export default function TodoForm({ todos }) {
  // Estado optimista para actualización inmediata de UI
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo) => [...state, newTodo]
  );
  
  async function action(formData) {
    // Crear una versión optimista de la nueva tarea
    const newTodo = {
      id: 'temp-' + Date.now(),
      title: formData.get('title'),
      completed: false,
      pending: true, // Indicador visual de que está en proceso
    };
    
    // Actualizar UI inmediatamente
    addOptimisticTodo(newTodo);
    
    // Enviar al servidor
    await createTodo(formData);
  }
  
  return (
    <div>
      <form action={action}>
        <input type="text" name="title" />
        <button type="submit">Añadir</button>
      </form>
      
      <ul>
        {optimisticTodos.map(todo => (
          <li key={todo.id} className={todo.pending ? 'pending' : ''}>
            {todo.title}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

#### Caching y Revalidación Inteligente

Next.js 15 proporciona un sistema avanzado de caché y revalidación:

```jsx
// app/products/[id]/page.js
async function getProduct(id, storeId) {
  const res = await fetch(`https://api.store.com/products/${id}`, {
    // Options para control granular de caché
    next: { 
      revalidate: 3600, // Revalidar cada hora
      tags: [`product-${id}`, `store-${storeId}`], // Tags para revalidación selectiva
    },
  });
  return res.json();
}

export default async function ProductPage({ params }) {
  const product = await getProduct(params.id, 'main-store');
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>Precio: ${product.price}</p>
    </div>
  );
}
```

La revalidación se puede disparar por tag específico cuando sea necesario:

```jsx
// app/api/revalidate/route.js
import { revalidateTag } from 'next/cache';
import { NextResponse } from 'next/server';

export async function POST(request) {
  const { productId } = await request.json();
  
  // Solo revalidar los componentes que usan este producto específico
  revalidateTag(`product-${productId}`);
  
  return NextResponse.json({ revalidated: true });
}
```

### 6.2.7 Layouts y Componentes Compartidos en Next.js 15

La gestión de layouts y componentes compartidos ha evolucionado significativamente en Next.js 15, especialmente con la introducción del App Router.

#### Layouts en Pages Router (Enfoque Tradicional)

```jsx
// components/Layout.js
import Head from 'next/head';
import Navbar from './Navbar';
import Footer from './Footer';

export default function Layout({ children, title = 'Mi Sitio' }) {
  return (
    <>
      <Head>
        <title>{title}</title>
        <meta name="description" content="Mi sitio web con Next.js" />
        <link rel="icon" href="/favicon.ico" />
      </Head>
      
      <div className="container">
        <Navbar />
        <main>{children}</main>
        <Footer />
      </div>
    </>
  );
}

// pages/index.js
import Layout from '../components/Layout';

export default function Home() {
  return (
    <Layout title="Inicio - Mi Sitio">
      <h1>Bienvenido a mi sitio</h1>
    </Layout>
  );
}
```

#### Layouts en App Router (Next.js 15)

En App Router, Next.js 15 ofrece un sistema de layouts integrado basado en archivos de convención, lo que elimina la necesidad de importar layouts manualmente en cada página:

```jsx
// app/layout.js - Layout raíz para toda la aplicación
import Navbar from '../components/Navbar';
import Footer from '../components/Footer';
import './globals.css';

// Metadata API - para SEO y social sharing
export const metadata = {
  title: {
    template: '%s | Mi Sitio',
    default: 'Mi Sitio',
  },
  description: 'Mi sitio web con Next.js',
};

export default function RootLayout({ children }) {
  return (
    <html lang="es">
      <body>
        <div className="container">
          <Navbar />
          <main>{children}</main>
          <Footer />
        </div>
      </body>
    </html>
  );
}

// Layouts anidados - Una de las características principales de App Router en Next.js 15
// app/dashboard/layout.js
export default function DashboardLayout({ children }) {
  return (
    <div className="dashboard-layout">
      <aside>
        {/* Sidebar del dashboard */}
        <nav>
          <h3>Panel de Control</h3>
          <ul>
            <li><a href="/dashboard/analytics">Análisis</a></li>
            <li><a href="/dashboard/settings">Configuración</a></li>
            <li><a href="/dashboard/users">Usuarios</a></li>
          </ul>
        </nav>
      </aside>
      <div className="dashboard-content">
        {children}
      </div>
    </div>
  );
}
```

En Next.js 15, los layouts anidados permiten mantener el estado entre navegaciones en la misma sección, lo que mejora considerablemente la experiencia de usuario.

#### Templates vs. Layouts

Next.js 15 introduce también los **templates** como una alternativa a los layouts. A diferencia de los layouts que preservan su estado y permanecen montados entre navegaciones, los templates se montan desde cero en cada navegación:

```jsx
// app/productos/template.js
export default function ProductsTemplate({ children }) {
  // Este componente se monta de nuevo en cada navegación
  return (
    <div className="products-container">
      {/* Útil para reiniciar animaciones o estados en cada navegación */}
      <div key={Math.random()} className="animate-in">
        <ProductFilters />
      </div>
      {children}
    </div>
  );
}
```

#### Estados de Carga (Loading UI)

Next.js 15 proporciona una forma sencilla de mostrar estados de carga durante las navegaciones o mientras se cargan datos:

```jsx
// app/dashboard/loading.js
export default function DashboardLoading() {
  // Esta UI se muestra mientras se carga la página o los datos
  return (
    <div className="loading-container">
      <div className="spinner"></div>
      <p>Cargando panel de control...</p>
    </div>
  );
}
```

El componente de loading se muestra automáticamente durante:

1. La navegación entre rutas
2. La carga de datos en componentes React Server Components
3. La carga de datos en `useEffect` o cuando un cliente llama a una función asíncrona

#### Manejo de Errores (Error Boundaries)

Next.js 15 integra manejo de errores para manejar fallas de forma elegante:

```jsx
// app/dashboard/error.js
'use client'; // Los componentes de error deben ser Client Components

import { useEffect } from 'react';

export default function DashboardError({ error, reset }) {
  useEffect(() => {
    // Opcional: registrar el error en un servicio de monitoreo
    console.error('Error en Dashboard:', error);
  }, [error]);

  return (
    <div className="error-container">
      <h2>¡Algo salió mal en el panel!</h2>
      <p>{error.message || 'Ha ocurrido un error inesperado'}</p>
      <button
        onClick={() => reset()} // Intenta volver a renderizar el segmento
        className="retry-button"
      >
        Intentar de nuevo
      </button>
    </div>
  );
}
```

Los componentes de error pueden ser específicos para segmentos de la aplicación, permitiendo un control preciso del manejo de errores.

### 6.2.8 Optimizaciones y Mejores Prácticas en Next.js 15

#### Internacionalización (i18n)

Next.js 15 facilita la creación de aplicaciones multiidioma con un enfoque basado en directorios:

```jsx
// app/[lang]/layout.js
import { getDictionary } from '../dictionaries';

export async function generateStaticParams() {
  return [{ lang: 'es' }, { lang: 'en' }, { lang: 'fr' }];
}

export default async function Layout({ children, params }) {
  const { lang } = params;
  const dictionary = await getDictionary(lang);
  
  return (
    <html lang={lang}>
      <body>
        <LocaleContext.Provider value={{ lang, dictionary }}>
          {children}
        </LocaleContext.Provider>
      </body>
    </html>
  );
}
```

#### Optimización de Imágenes

El componente Image de Next.js 15 ofrece optimizaciones automáticas:

```jsx
import Image from 'next/image';

export default function ProductView() {
  return (
    <div>
      <Image
        src="/products/coffee-machine.jpg"
        alt="Máquina de café"
        width={500}
        height={300}
        priority={true} // Carga prioritaria para LCP
        placeholder="blur" // Efecto blur durante la carga
        sizes="(max-width: 768px) 100vw, 50vw" // Responsive
      />
    </div>
  );
}
```

#### Scripts Optimizados

```jsx
import Script from 'next/script';

export default function Analytics() {
  return (
    <>
      {/* Carga cuando la página se vuelve interactiva */}
      <Script id="analytics" strategy="afterInteractive">
        {`(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':new Date().getTime(),event:'gtm.js'});`}
      </Script>
      
      {/* Se carga antes que cualquier otro script */}
      <Script
        src="https://cdn.example.com/critical.js"
        strategy="beforeInteractive"
      />
      
      {/* Se carga cuando el usuario hace scroll */}
      <Script
        src="https://cdn.example.com/lazy.js"
        strategy="lazyOnload"
      />
    </>
  );
}
```

#### Conclusión

Next.js 15 representa un avance significativo en el desarrollo de aplicaciones web modernas, proporcionando herramientas poderosas para construir experiencias rápidas, confiables y escalables. La combinación del App Router con React Server Components ofrece un modelo de desarrollo que optimiza tanto la experiencia del desarrollador como el rendimiento de la aplicación final. Las mejoras en caché, optimizaciones automáticas y patrones de composición facilitan la creación de aplicaciones sofisticadas manteniendo un código claro y mantenible.

# Capítulo 6: React y Next.js (Continuación)

## 6.2 Next.js: El Framework de React

Next.js es un framework de React que proporciona estructura, características y optimizaciones para aplicaciones web. Creado por Vercel, Next.js simplifica el desarrollo de aplicaciones React complejas con funcionalidades como renderizado del lado del servidor (SSR), generación estática (SSG), y enrutamiento basado en sistema de archivos.

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

#### Estructura de Directorios en Next.js 13+

En versiones más recientes (Next.js 13+), la estructura recomendada cambia usando el directorio `app`:

```
mi-proyecto/
├── app/                 # Directorio principal para el App Router
│   ├── layout.js        # Diseño compartido entre páginas
│   ├── page.js          # Página principal (reemplaza a pages/index.js)
│   ├── about/
│   │   └── page.js      # Ruta /about
│   └── api/             # Rutas API con el App Router
│       └── route.js     # Manejador de ruta API
├── components/          # Componentes reutilizables
├── public/              # Archivos estáticos
└── ...                  # Otros archivos de configuración
```

### 6.2.3 Sistema de Enrutamiento

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
  
  // ...
}
```

### 6.2.4 Estrategias de Renderizado

Next.js ofrece múltiples estrategias de renderizado adaptadas a diferentes casos de uso.

#### Renderizado del Lado del Servidor (SSR)

El contenido se genera en cada solicitud en el servidor.

```jsx
// pages/ssr-page.js
export default function Page({ data }) {
  return <div>Datos: {data}</div>;
}

// Esta función se ejecuta en cada solicitud
export async function getServerSideProps() {
  // Fetch de datos en el servidor
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();
  
  // Pasar los datos como props
  return { props: { data } };
}
```

En App Router:

```jsx
// app/ssr-page/page.js
async function getData() {
  const res = await fetch('https://api.example.com/data', { cache: 'no-store' });
  if (!res.ok) throw new Error('Failed to fetch data');
  return res.json();
}

export default async function Page() {
  // Componentes de servidor pueden ser async
  const data = await getData();
  
  return <div>Datos: {data}</div>;
}
```

#### Generación Estática (SSG)

El contenido se genera durante la compilación y se sirve como HTML estático.

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
    // Opcional: revalidación incremental (ISR)
    revalidate: 60, // segundos
  };
}
```

En App Router:

```jsx
// app/ssg-page/page.js
async function getData() {
  // Por defecto, fetch en App Router hace caché y se comporta como SSG
  const res = await fetch('https://api.example.com/data');
  if (!res.ok) throw new Error('Failed to fetch data');
  return res.json();
}

export default async function Page() {
  const data = await getData();
  return <div>Datos estáticos: {data}</div>;
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

### 6.2.6 Layouts y Componentes Compartidos

#### Layouts en Pages Router

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

#### Layouts en App Router

```jsx
// app/layout.js
import Navbar from '../components/Navbar';
import Footer from '../components/Footer';
import './globals.css';

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

// Layouts anidados
// app/dashboard/layout.js
export default function DashboardLayout({ children }) {
  return (
    <div className="dashboard-layout">
      <aside>
        {/* Sidebar del dashboard */}
        <nav>{/* Enlaces de navegación */}</nav>
      </aside>
      <div className="dashboard-content">
        {children}
      </div>
    </div>
  );
}
```

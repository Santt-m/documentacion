# Capítulo 5: Sistemas de Diseño (Design Systems)

## 5.1 Fundamentos de los Sistemas de Diseño

Un sistema de diseño es una colección de componentes reutilizables, guiados por estándares claros, que pueden ensamblarse para construir cualquier número de aplicaciones. Va más allá de una simple biblioteca de componentes o guía de estilo, representando un enfoque holístico para el diseño de productos digitales.

### 5.1.1 Componentes Clave de un Sistema de Diseño

1. **Principios de Diseño**: Valores fundamentales que guían las decisiones de diseño.
2. **Tokens de Diseño**: Variables que almacenan valores visuales como colores, espaciado, tipografía.
3. **Componentes**: Elementos de UI reutilizables que implementan los tokens y siguen los principios.
4. **Patrones**: Soluciones probadas a problemas recurrentes de UI/UX.
5. **Documentación**: Guías sobre uso, implementación y desarrollo.

### 5.1.2 Beneficios de los Sistemas de Diseño

- **Consistencia**: Experiencia de usuario coherente a través de productos y plataformas.
- **Eficiencia**: Reducción significativa del tiempo de desarrollo.
- **Escalabilidad**: Facilita el crecimiento del producto y del equipo.
- **Calidad**: Componentes probados y refinados que cumplen estándares de accesibilidad y usabilidad.
- **Colaboración**: Mejora la comunicación entre diseñadores y desarrolladores.

### 5.1.3 Ejemplos de Sistemas de Diseño Populares

- **Material Design (Google)**: Enfoque en el diseño basado en papel con animaciones y efectos de profundidad.
- **Human Interface Guidelines (Apple)**: Principios para crear interfaces que funcionan armoniosamente con los dispositivos Apple.
- **Fluent Design System (Microsoft)**: Énfasis en movimiento, profundidad, materiales y luz.
- **Atlassian Design System**: Centrado en la claridad y la eficiencia para productos empresariales.
- **Lightning Design System (Salesforce)**: Componentes empresariales con accesibilidad incorporada.

## 5.2 Tokens de Diseño

Los tokens de diseño son variables con nombre que almacenan valores visuales y se utilizan en lugar de valores hardcodeados. Actúan como la "fuente única de verdad" para los valores de diseño.

### 5.2.1 Tipos de Tokens

```javascript
// tokens.js - Ejemplo de definición de tokens de diseño

// 1. Tokens Globales (Primitivos)
const globalTokens = {
  // Paleta de colores
  colors: {
    blue100: '#DAECFF',
    blue200: '#B6DAFF',
    blue300: '#84CAFF',
    blue400: '#53B1FD',
    blue500: '#2E90FA',
    blue600: '#1570EF',
    blue700: '#175CD3',
    blue800: '#1849A9',
    
    gray100: '#F2F4F7',
    gray200: '#E4E7EC',
    gray300: '#D0D5DD',
    gray400: '#98A2B3',
    gray500: '#667085',
    gray600: '#475467',
    gray700: '#344054',
    gray800: '#1D2939',
    
    white: '#FFFFFF',
    black: '#000000',
  },
  
  // Tipografía
  fontFamilies: {
    sans: "'Inter', -apple-system, BlinkMacSystemFont, sans-serif",
    serif: "Georgia, serif",
    mono: "'IBM Plex Mono', monospace",
  },
  
  // Tamaños de fuente (en rem para mejor accesibilidad)
  fontSizes: {
    xs: '0.75rem',    // 12px
    sm: '0.875rem',   // 14px
    md: '1rem',       // 16px
    lg: '1.125rem',   // 18px
    xl: '1.25rem',    // 20px
    '2xl': '1.5rem',  // 24px
    '3xl': '1.875rem', // 30px
    '4xl': '2.25rem', // 36px
    '5xl': '3rem',    // 48px
  },
  
  // Pesos de fuente
  fontWeights: {
    regular: 400,
    medium: 500,
    semiBold: 600,
    bold: 700,
  },
  
  // Espaciado
  spacing: {
    '0': '0',
    '1': '0.25rem',  // 4px
    '2': '0.5rem',   // 8px
    '3': '0.75rem',  // 12px
    '4': '1rem',     // 16px
    '5': '1.25rem',  // 20px
    '6': '1.5rem',   // 24px
    '8': '2rem',     // 32px
    '10': '2.5rem',  // 40px
    '12': '3rem',    // 48px
    '16': '4rem',    // 64px
    '20': '5rem',    // 80px
  },
  
  // Radios de borde
  borderRadius: {
    none: '0',
    sm: '0.125rem',  // 2px
    md: '0.375rem',  // 6px
    lg: '0.5rem',    // 8px
    xl: '0.75rem',   // 12px
    '2xl': '1rem',   // 16px
    full: '9999px',
  },
  
  // Sombras
  shadows: {
    sm: '0 1px 2px 0 rgba(0, 0, 0, 0.05)',
    md: '0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06)',
    lg: '0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05)',
    xl: '0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04)',
  },
};

// 2. Tokens Semánticos (Alias que referencian a los globales)
const semanticTokens = {
  colors: {
    // Colores de marca
    primary: globalTokens.colors.blue600,
    primaryLight: globalTokens.colors.blue400,
    primaryDark: globalTokens.colors.blue800,
    
    // Colores de interfaz
    background: globalTokens.colors.white,
    backgroundAlt: globalTokens.colors.gray100,
    backgroundInverse: globalTokens.colors.gray800,
    
    // Colores de texto
    text: globalTokens.colors.gray800,
    textMuted: globalTokens.colors.gray500,
    textInverse: globalTokens.colors.white,
    
    // Colores de estado
    success: '#12B76A',
    warning: '#F79009',
    error: '#F04438',
    info: '#53B1FD',
  },
  
  // Configuraciones específicas de componentes
  buttons: {
    primary: {
      background: semanticTokens.colors.primary,
      text: globalTokens.colors.white,
      hoverBackground: globalTokens.colors.blue700,
      borderRadius: globalTokens.borderRadius.md,
    },
    secondary: {
      background: globalTokens.colors.white,
      text: semanticTokens.colors.primary,
      border: semanticTokens.colors.primary,
      hoverBackground: globalTokens.colors.blue100,
      borderRadius: globalTokens.borderRadius.md,
    },
  }
};

export { globalTokens, semanticTokens };
```

### 5.2.2 Implementación con CSS Variables

```css
/* tokens.css - Implementación con variables CSS */
:root {
  /* Colores */
  --color-blue-100: #DAECFF;
  --color-blue-500: #2E90FA;
  --color-blue-700: #175CD3;
  
  --color-gray-100: #F2F4F7;
  --color-gray-500: #667085;
  --color-gray-800: #1D2939;
  
  /* Colores semánticos */
  --color-primary: var(--color-blue-500);
  --color-background: #FFFFFF;
  --color-text: var(--color-gray-800);
  
  /* Espaciado */
  --space-1: 0.25rem;
  --space-4: 1rem;
  --space-8: 2rem;
  
  /* Tipografía */
  --font-family-base: 'Inter', -apple-system, sans-serif;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  
  /* Componentes */
  --button-padding: var(--space-1) var(--space-4);
  --button-border-radius: 0.375rem;
}

/* Uso de variables */
.button-primary {
  background-color: var(--color-primary);
  color: var(--color-background);
  padding: var(--button-padding);
  border-radius: var(--button-border-radius);
  font-family: var(--font-family-base);
}

.button-primary:hover {
  background-color: var(--color-blue-700);
}
```

### 3.4.3 Integración en el Ciclo de Desarrollo

Para ser efectivo, el modelado de amenazas y los principios de seguridad deben integrarse en todo el ciclo de desarrollo:

1. **Fase de Diseño**: Realizar modelado de amenazas STRIDE para identificar riesgos temprano.
2. **Fase de Desarrollo**: Aplicar principios de Secure by Default y Least Privilege.
3. **Fase de Pruebas**: Incluir pruebas de seguridad específicas para las amenazas identificadas.
4. **Fase de Despliegue**: Verificar configuraciones seguras y aplicar hardening.
5. **Fase de Mantenimiento**: Monitoreo continuo, actualizaciones de seguridad y revisión periódica de amenazas.

Este enfoque proactivo y continuo asegura que la seguridad no sea un componente añadido al final, sino una consideración integral a lo largo de todo el proceso de desarrollo de software.

#### Implementación en Metodologías Ágiles

El modelado de amenazas debe adaptarse a las metodologías ágiles de desarrollo:

```typescript
// Ejemplo de integración de análisis de seguridad en el proceso de desarrollo
interface UserStory {
  id: string;
  title: string;
  description: string;
  acceptanceCriteria: string[];
  securityRequirements: SecurityRequirement[];
  estimatedPoints: number;
}

interface SecurityRequirement {
  id: string;
  description: string;
  threatCategory: 'S' | 'T' | 'R' | 'I' | 'D' | 'E'; // STRIDE
  priority: 'critical' | 'high' | 'medium' | 'low';
  testCriteria: string;
  implementationNotes: string;
}

// Ejemplo de user story con requisitos de seguridad integrados
const userStoryWithSecurity: UserStory = {
  id: 'US-123',
  title: 'Como usuario quiero poder restablecer mi contraseña',
  description: 'El usuario debe poder solicitar un restablecimiento de contraseña mediante correo electrónico',
  acceptanceCriteria: [
    'Debe enviar un correo con un enlace único de restablecimiento',
    'El enlace debe expirar en 1 hora',
    'Debe requerir verificación de la contraseña actual antes de cambiarla'
  ],
  securityRequirements: [
    {
      id: 'SEC-01',
      description: 'Prevenir enumeración de usuarios',
      threatCategory: 'I', // Information Disclosure
      priority: 'high',
      testCriteria: 'La respuesta al solicitar restablecimiento debe ser idéntica para usuarios existentes y no existentes',
      implementationNotes: 'Usar tiempo constante para la respuesta independientemente de si el usuario existe'
    },
    {
      id: 'SEC-02',
      description: 'Protección contra ataques de fuerza bruta',
      threatCategory: 'S', // Spoofing
      priority: 'critical',
      testCriteria: 'Verificar que se implementa rate limiting por IP y por cuenta de usuario',
      implementationNotes: 'Implementar throttling con incremental backoff después de 3 intentos fallidos'
    },
    {
      id: 'SEC-03',
      description: 'Tokens de restablecimiento seguros',
      threatCategory: 'E', // Elevation of Privilege
      priority: 'critical',
      testCriteria: 'El token debe ser aleatorio, de al menos 128 bits de entropía y específico para un solo uso',
      implementationNotes: 'Usar crypto.randomBytes(16) y almacenar hash del token, no el token mismo'
    }
  ],
  estimatedPoints: 5
};

// Integración con la definición de "Done"
const securityDefinitionOfDone = [
  'Todas las securityRequirements están implementadas y probadas',
  'Se ha realizado análisis estático de código con herramientas especializadas',
  'Se han ejecutado pruebas dinámicas para las amenazas identificadas',
  'El código ha pasado por una revisión de seguridad por pares',
  'Se han documentado las decisiones de seguridad tomadas'
];

// Ejemplo de integración con pruebas automatizadas
describe('Password Reset Feature', () => {
  // Prueba normal de funcionalidad
  it('should send password reset email to valid user', async () => {
    // Prueba de funcionalidad estándar
  });
  
  // Prueba específica para SEC-01: Prevenir enumeración de usuarios
  it('should return same response and timing for non-existent users (SEC-01)', async () => {
    const start1 = performance.now();
    const response1 = await request(app)
      .post('/api/reset-password')
      .send({ email: 'existing@example.com' });
    const time1 = performance.now() - start1;
    
    const start2 = performance.now();
    const response2 = await request(app)
      .post('/api/reset-password')
      .send({ email: 'nonexistent@example.com' });
    const time2 = performance.now() - start2;
    
    // Verificar que las respuestas son idénticas
    expect(response1.status).toEqual(response2.status);
    expect(response1.body).toEqual(response2.body);
    
    // Verificar que los tiempos son similares (dentro de un margen razonable)
    expect(Math.abs(time1 - time2)).toBeLessThan(100); // Menos de 100ms de diferencia
  });
  
  // Prueba para SEC-02: Protección contra fuerza bruta
  it('should implement rate limiting after multiple attempts (SEC-02)', async () => {
    const attempts = [];
    
    // Realizar múltiples intentos
    for (let i = 0; i < 5; i++) {
      const response = await request(app)
        .post('/api/reset-password')
        .send({ email: 'test@example.com' })
        .set('X-Forwarded-For', '192.168.1.1'); // Simular misma IP
      
      attempts.push(response.status);
    }
    
    // Verificar que después de cierto número de intentos se limita el acceso
    expect(attempts.some(status => status === 429)).toBe(true);
  });
  
  // Prueba para SEC-03: Tokens seguros de restablecimiento
  it('should generate secure, one-time reset tokens (SEC-03)', async () => {
    // Solicitar dos restablecimientos para el mismo usuario
    await request(app)
      .post('/api/reset-password')
      .send({ email: 'user@example.com' });
    
    // Extraer el primer token desde la base de datos de prueba
    const firstToken = await testDb.getResetToken('user@example.com');
    
    await request(app)
      .post('/api/reset-password')
      .send({ email: 'user@example.com' });
    
    // Extraer el segundo token
    const secondToken = await testDb.getResetToken('user@example.com');
    
    // Verificar que los tokens son diferentes
    expect(firstToken).not.toEqual(secondToken);
    
    // Verificar que el primer token ya no es válido
    const resetResult = await request(app)
      .post('/api/reset-password/confirm')
      .send({ token: firstToken, newPassword: 'NewSecure123!' });
    
    expect(resetResult.status).toEqual(400);
  });
});
```

#### Integración en Revisiones de Código

Las revisiones de código deben incluir consideraciones específicas de seguridad:

```typescript
// Ejemplo de plantilla para revisión de código con enfoque en seguridad
interface SecurityCodeReview {
  pullRequestId: string;
  reviewer: string;
  securityChecklist: SecurityCheckItem[];
  vulnerabilitiesFound: Vulnerability[];
  approved: boolean;
  comments: string;
}

interface SecurityCheckItem {
  category: string;
  description: string;
  compliant: boolean;
  evidence?: string;
  mitigation?: string;
}

interface Vulnerability {
  description: string;
  severity: 'critical' | 'high' | 'medium' | 'low';
  cweId?: string; // Common Weakness Enumeration ID
  location: string;
  suggestedFix: string;
}

// Ejemplo de una revisión de seguridad completada
const securityReview: SecurityCodeReview = {
  pullRequestId: 'PR-456',
  reviewer: 'security-engineer@company.com',
  securityChecklist: [
    {
      category: 'Input Validation',
      description: '¿Todas las entradas del usuario son validadas adecuadamente?',
      compliant: true,
      evidence: 'Se utiliza Zod para validación de esquemas en líneas 45-67'
    },
    {
      category: 'Authentication',
      description: '¿Se implementan prácticas seguras de autenticación?',
      compliant: true,
      evidence: 'Se usa JWT con expiration y algoritmo RS256'
    },
    {
      category: 'Authorization',
      description: '¿Se verifican los permisos antes de operaciones sensibles?',
      compliant: false,
      mitigation: 'Añadir verificación de roles en el endpoint de eliminación'
    },
    {
      category: 'Data Protection',
      description: '¿Los datos sensibles están protegidos en tránsito y reposo?',
      compliant: true,
      evidence: 'Se usa HTTPS y cifrado AES-256-GCM para datos en la base de datos'
    },
    {
      category: 'Error Handling',
      description: '¿El manejo de errores evita la exposición de información sensible?',
      compliant: false,
      mitigation: 'Sanitizar mensajes de error en líneas 120-125'
    }
  ],
  vulnerabilitiesFound: [
    {
      description: 'Falta de verificación de autorización en endpoint de eliminación',
      severity: 'high',
      cweId: 'CWE-285',
      location: 'src/controllers/user.controller.ts:deleteUser (línea 87)',
      suggestedFix: 'Añadir middleware requirePermission("delete:user") antes del controlador'
    },
    {
      description: 'Exposición de información sensible en mensajes de error',
      severity: 'medium',
      cweId: 'CWE-209',
      location: 'src/middlewares/error-handler.ts (líneas 120-125)',
      suggestedFix: 'Implementar sanitizeErrorResponse() para filtrar detalles sensibles'
    }
  ],
  approved: false,
  comments: 'No se puede aprobar hasta solucionar las vulnerabilidades de autorización y exposición de información en errores.'
};

// Automatización de las revisiones de seguridad
class SecurityReviewAutomation {
  async performAutomatedChecks(pullRequestId: string): Promise<SecurityCodeReview> {
    // Obtener cambios del pull request
    const changes = await this.gitService.getPullRequestChanges(pullRequestId);
    
    // Ejecutar herramientas de análisis estático
    const staticAnalysisResults = await Promise.all([
      this.runStaticAnalysisTool('eslint-security', changes),
      this.runStaticAnalysisTool('sonarqube', changes),
      this.runStaticAnalysisTool('snyk-code', changes)
    ]);
    
    // Convertir resultados a formato de revisión de seguridad
    const vulnerabilities = this.convertToVulnerabilities(staticAnalysisResults);
    
    // Generar checklist basado en los cambios detectados
    const checklist = this.generateSecurityChecklist(changes, vulnerabilities);
    
    // Determinar si se aprueba automáticamente
    const criticalOrHighVulnerabilities = vulnerabilities.filter(
      v => v.severity === 'critical' || v.severity === 'high'
    );
    
    return {
      pullRequestId,
      reviewer: 'security-automation-bot',
      securityChecklist: checklist,
      vulnerabilitiesFound: vulnerabilities,
      approved: criticalOrHighVulnerabilities.length === 0,
      comments: criticalOrHighVulnerabilities.length > 0
        ? 'Se requiere revisión manual debido a vulnerabilidades críticas o altas detectadas'
        : 'Revisión automática aprobada, sin vulnerabilidades críticas o altas'
    };
  }
}
```

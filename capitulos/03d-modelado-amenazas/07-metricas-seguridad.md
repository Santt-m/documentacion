### 3.4.7 Métricas y Monitoreo de Seguridad

Para evaluar la efectividad del modelado de amenazas y mantener un nivel de seguridad adecuado, es fundamental establecer métricas y sistemas de monitoreo que proporcionen visibilidad sobre la postura de seguridad del sistema.

#### Métricas Clave para el Modelado de Amenazas

Las métricas permiten cuantificar el valor y efectividad del proceso de modelado de amenazas:

```typescript
// Implementación de métricas para el modelado de amenazas
class ThreatModelingMetrics {
  // Recopila métricas del proceso de modelado de amenazas
  async collectMetrics(model: ThreatModel): Promise<ModelingMetrics> {
    const threats = this.getAllThreats(model);
    
    // Métricas de cobertura
    const coverageMetrics = {
      totalComponents: model.components.length,
      componentsModeled: model.components.filter(c => c.threatsMapped).length,
      coveragePercentage: 0,
      avgThreatsPerComponent: 0,
    };
    
    coverageMetrics.coveragePercentage = 
      (coverageMetrics.componentsModeled / coverageMetrics.totalComponents) * 100;
      
    coverageMetrics.avgThreatsPerComponent = 
      threats.length / coverageMetrics.componentsModeled;
    
    // Métricas de mitigación
    const mitigationMetrics = {
      totalThreats: threats.length,
      mitigatedThreats: threats.filter(t => t.mitigated).length,
      mitigationRate: 0,
      highRiskUnmitigated: threats.filter(t => 
        t.riskLevel === 'HIGH' && !t.mitigated
      ).length,
      criticalRiskUnmitigated: threats.filter(t => 
        t.riskLevel === 'CRITICAL' && !t.mitigated
      ).length,
    };
    
    mitigationMetrics.mitigationRate = 
      (mitigationMetrics.mitigatedThreats / mitigationMetrics.totalThreats) * 100;
    
    // Métricas de esfuerzo
    const effortMetrics = {
      averageMitigationEffort: this.calculateAverageMitigationEffort(threats),
      totalSecurityDebt: this.calculateSecurityDebt(threats),
      priorityRatio: this.calculatePriorityRatio(threats),
    };
    
    // Métricas de tiempo
    const timeMetrics = {
      meanTimeToMitigate: await this.calculateMTTM(threats),
      riskExposureDuration: await this.calculateRiskExposureDuration(threats),
    };
    
    return {
      coverage: coverageMetrics,
      mitigation: mitigationMetrics,
      effort: effortMetrics,
      time: timeMetrics,
      timestamp: new Date().toISOString()
    };
  }
  
  // Genera un informe visual de las métricas
  async generateMetricsReport(metrics: ModelingMetrics): Promise<ReportOutput> {
    // Generar gráficas y visualizaciones
    const charts = await this.createVisualization(metrics);
    
    // Detectar tendencias (comparando con métricas anteriores)
    const trends = await this.analyzeTrends(metrics);
    
    // Crear informe con evaluaciones y recomendaciones
    const report = {
      summary: this.createExecutiveSummary(metrics),
      details: metrics,
      charts,
      trends,
      recommendations: this.generateRecommendations(metrics, trends)
    };
    
    return report;
  }
  
  // Métodos auxiliares
  private getAllThreats(model: ThreatModel): Threat[] {
    return model.components.flatMap(c => c.threats || []);
  }
  
  private calculateAverageMitigationEffort(threats: Threat[]): number {
    const mitigatedThreats = threats.filter(t => t.mitigated && t.mitigationEffort);
    if (mitigatedThreats.length === 0) return 0;
    
    const totalEffort = mitigatedThreats.reduce(
      (sum, threat) => sum + threat.mitigationEffort, 0
    );
    
    return totalEffort / mitigatedThreats.length;
  }
  
  private calculateSecurityDebt(threats: Threat[]): number {
    // Calcular "deuda" de seguridad basada en amenazas sin mitigar
    const unmitigatedThreats = threats.filter(t => !t.mitigated);
    
    return unmitigatedThreats.reduce((debt, threat) => {
      const riskFactor = this.getRiskFactor(threat.riskLevel);
      const estimatedEffort = threat.estimatedMitigationEffort || this.estimateEffort(threat);
      return debt + (riskFactor * estimatedEffort);
    }, 0);
  }
  
  private getRiskFactor(riskLevel: string): number {
    switch(riskLevel) {
      case 'CRITICAL': return 10;
      case 'HIGH': return 5;
      case 'MEDIUM': return 2;
      case 'LOW': return 1;
      default: return 1;
    }
  }
  
  private calculatePriorityRatio(threats: Threat[]): number {
    const highPriorityThreats = threats.filter(t => 
      t.riskLevel === 'CRITICAL' || t.riskLevel === 'HIGH'
    );
    
    if (highPriorityThreats.length === 0) return 1; // Proporción perfecta
    
    const mitigatedHighPriority = highPriorityThreats.filter(t => t.mitigated).length;
    return mitigatedHighPriority / highPriorityThreats.length;
  }
  
  private async calculateMTTM(threats: Threat[]): Promise<number> {
    // Calcular tiempo medio de mitigación desde identificación hasta implementación
    const mitigatedThreats = threats.filter(
      t => t.mitigated && t.identifiedDate && t.mitigatedDate
    );
    
    if (mitigatedThreats.length === 0) return 0;
    
    const totalDays = mitigatedThreats.reduce((sum, threat) => {
      const identifiedDate = new Date(threat.identifiedDate);
      const mitigatedDate = new Date(threat.mitigatedDate);
      const days = (mitigatedDate.getTime() - identifiedDate.getTime()) / (1000 * 3600 * 24);
      return sum + days;
    }, 0);
    
    return totalDays / mitigatedThreats.length;
  }
  
  // Métodos adicionales...
}

// Ejemplo de uso
const metricsCollector = new ThreatModelingMetrics();
const currentMetrics = await metricsCollector.collectMetrics(threatModel);
const report = await metricsCollector.generateMetricsReport(currentMetrics);

console.log(`
Resumen de Métricas de Modelado de Amenazas:
----------------------------------------
Cobertura del Modelo: ${currentMetrics.coverage.coveragePercentage.toFixed(1)}%
Tasa de Mitigación: ${currentMetrics.mitigation.mitigationRate.toFixed(1)}%
Amenazas críticas sin mitigar: ${currentMetrics.mitigation.criticalRiskUnmitigated}
Deuda de seguridad estimada: ${currentMetrics.effort.totalSecurityDebt} puntos
Tiempo medio de mitigación: ${currentMetrics.time.meanTimeToMitigate.toFixed(1)} días
`);
```

#### Sistema de Monitoreo de Seguridad

Un sistema de monitoreo efectivo permite detectar posibles brechas de seguridad en tiempo real:

```typescript
// Implementación de un sistema de monitoreo de seguridad
class SecurityMonitoring {
  private alertSystem: AlertSystem;
  private logAnalyzer: LogAnalyzer;
  private metricsCollector: MetricsCollector;
  private thresholds: SecurityThresholds;
  
  constructor(config: MonitoringConfig) {
    this.alertSystem = new AlertSystem(config.alertEndpoints);
    this.logAnalyzer = new LogAnalyzer(config.logSources);
    this.metricsCollector = new MetricsCollector(config.metricsSources);
    this.thresholds = config.thresholds;
  }
  
  // Iniciar el monitoreo continuo
  async startMonitoring(): Promise<void> {
    console.log('Iniciando monitoreo de seguridad...');
    
    // Iniciar análisis de logs en tiempo real
    this.logAnalyzer.startRealTimeAnalysis(async (securityEvent) => {
      await this.processSecurityEvent(securityEvent);
    });
    
    // Programar recolección periódica de métricas
    setInterval(async () => {
      const metrics = await this.metricsCollector.collectCurrentMetrics();
      await this.analyzeMetrics(metrics);
    }, this.thresholds.metricsInterval);
    
    // Programar verificaciones de integridad
    setInterval(async () => {
      const integrityReport = await this.performIntegrityCheck();
      if (!integrityReport.passed) {
        await this.handleIntegrityFailure(integrityReport);
      }
    }, this.thresholds.integrityCheckInterval);
  }
  
  // Procesar eventos de seguridad detectados
  private async processSecurityEvent(event: SecurityEvent): Promise<void> {
    // Correlacionar con otros eventos para detectar patrones
    const correlatedEvents = await this.logAnalyzer.correlateWithPreviousEvents(event);
    
    // Evaluar el nivel de riesgo del evento
    const riskLevel = this.evaluateRiskLevel(event, correlatedEvents);
    
    // Si supera el umbral, generar alerta
    if (riskLevel >= this.thresholds.alertThreshold) {
      await this.alertSystem.triggerAlert({
        title: `[${this.getRiskLevelName(riskLevel)}] ${event.type}`,
        description: event.description,
        sourceIp: event.sourceIp,
        targetComponent: event.targetComponent,
        timestamp: event.timestamp,
        correlatedEvents: correlatedEvents.length,
        suggestedActions: this.getSuggestedActions(event)
      });
    }
    
    // Registrar el evento para análisis posterior
    await this.logAnalyzer.storeProcessedEvent({
      ...event,
      riskLevel,
      processed: true,
      correlatedCount: correlatedEvents.length
    });
  }
  
  // Analizar métricas para detectar tendencias anómalas
  private async analyzeMetrics(metrics: SecurityMetrics): Promise<void> {
    // Detectar anomalías en las métricas
    const anomalies = this.metricsCollector.detectAnomalies(metrics);
    
    for (const anomaly of anomalies) {
      // Si es una anomalía significativa, alertar
      if (anomaly.significanceScore > this.thresholds.anomalyThreshold) {
        await this.alertSystem.triggerAlert({
          title: `Anomalía detectada: ${anomaly.metricName}`,
          description: `Valor anómalo de ${anomaly.actualValue} (esperado ${anomaly.expectedRange.min}-${anomaly.expectedRange.max})`,
          anomalyScore: anomaly.significanceScore,
          metricHistory: anomaly.recentHistory,
          timestamp: new Date().toISOString(),
          suggestedActions: this.getSuggestedActionsForAnomaly(anomaly)
        });
      }
    }
    
    // Almacenar métricas para análisis histórico
    await this.metricsCollector.storeMetricsSnapshot(metrics);
  }
  
  // Realizar verificación de integridad del sistema
  private async performIntegrityCheck(): Promise<IntegrityReport> {
    const checks = [
      await this.checkFileIntegrity(),
      await this.checkConfigurationIntegrity(),
      await this.checkDatabaseIntegrity()
      // Otros checks...
    ];
    
    return {
      timestamp: new Date().toISOString(),
      passed: checks.every(check => check.passed),
      checks,
      summary: checks.filter(check => !check.passed).map(check => check.failReason)
    };
  }
  
  // Manejar fallos de integridad
  private async handleIntegrityFailure(report: IntegrityReport): Promise<void> {
    // Generar alerta de alta prioridad
    await this.alertSystem.triggerAlert({
      title: 'Fallo de verificación de integridad',
      description: `Detectados ${report.checks.filter(c => !c.passed).length} fallos de integridad`,
      details: report.summary,
      timestamp: report.timestamp,
      priority: 'HIGH',
      suggestedActions: [
        'Revisar registros de auditoría',
        'Verificar accesos no autorizados',
        'Comparar con versiones respaldadas',
        'Iniciar protocolo de respuesta a incidentes si se confirma compromiso'
      ]
    });
    
    // Registrar para análisis forense
    await this.logAnalyzer.logForensicData({
      type: 'INTEGRITY_FAILURE',
      timestamp: report.timestamp,
      details: report
    });
  }
  
  // Métodos de utilidad
  private evaluateRiskLevel(event: SecurityEvent, correlatedEvents: SecurityEvent[]): number {
    // Lógica para evaluar el nivel de riesgo basado en características del evento
    // y correlación con otros eventos
    // ...
    return riskLevel;
  }
  
  private getRiskLevelName(level: number): string {
    if (level >= 8) return 'CRITICAL';
    if (level >= 6) return 'HIGH';
    if (level >= 4) return 'MEDIUM';
    return 'LOW';
  }
  
  private getSuggestedActions(event: SecurityEvent): string[] {
    // Sugerir acciones basadas en el tipo de evento
    // ...
    return actions;
  }
  
  private getSuggestedActionsForAnomaly(anomaly: MetricAnomaly): string[] {
    // Sugerir acciones basadas en el tipo de anomalía
    // ...
    return actions;
  }
}

// Implementación de analizador de logs
class LogAnalyzer {
  private logSources: LogSource[];
  private eventDatabase: SecurityEventDatabase;
  private patterns: DetectionPattern[];
  
  constructor(logSources: LogSource[]) {
    this.logSources = logSources;
    this.eventDatabase = new SecurityEventDatabase();
    this.patterns = this.loadDetectionPatterns();
  }
  
  // Iniciar análisis en tiempo real
  async startRealTimeAnalysis(callback: (event: SecurityEvent) => Promise<void>): Promise<void> {
    for (const source of this.logSources) {
      // Configurar streaming de logs
      await source.connect();
      
      source.onLogEntry(async (entry) => {
        // Analizar cada entrada de log
        const detectedEvents = this.analyzeLogEntry(entry);
        
        // Procesar eventos detectados
        for (const event of detectedEvents) {
          await callback(event);
        }
      });
    }
  }
  
  // Analizar una entrada de log
  private analyzeLogEntry(entry: LogEntry): SecurityEvent[] {
    const detectedEvents = [];
    
    // Aplicar patrones de detección
    for (const pattern of this.patterns) {
      if (pattern.matches(entry)) {
        detectedEvents.push({
          type: pattern.eventType,
          description: pattern.createDescription(entry),
          sourceIp: entry.sourceIp || 'unknown',
          targetComponent: entry.component || 'unknown',
          timestamp: entry.timestamp,
          severity: pattern.severity,
          rawLog: entry
        });
      }
    }
    
    return detectedEvents;
  }
  
  // Correlacionar con eventos anteriores
  async correlateWithPreviousEvents(event: SecurityEvent): Promise<SecurityEvent[]> {
    // Buscar eventos relacionados en la base de datos
    const relatedEvents = await this.eventDatabase.findRelatedEvents({
      sourceIp: event.sourceIp,
      targetComponent: event.targetComponent,
      timeWindow: 3600, // 1 hora en segundos
      maxResults: 50
    });
    
    return relatedEvents;
  }
  
  // Cargar patrones de detección
  private loadDetectionPatterns(): DetectionPattern[] {
    // Cargar patrones desde configuración
    // ...
    return patterns;
  }
  
  // Almacenar evento procesado
  async storeProcessedEvent(event: ProcessedSecurityEvent): Promise<void> {
    await this.eventDatabase.storeEvent(event);
  }
  
  // Registrar datos forenses
  async logForensicData(data: ForensicData): Promise<void> {
    await this.eventDatabase.storeForensicData(data);
  }
}

// Ejemplo de configuración del monitoreo
const monitoringConfig: MonitoringConfig = {
  alertEndpoints: [
    {
      type: 'email',
      recipients: ['security-team@example.com'],
      minSeverity: 'MEDIUM'
    },
    {
      type: 'slack',
      webhook: process.env.SLACK_SECURITY_WEBHOOK,
      channel: '#security-alerts',
      minSeverity: 'HIGH'
    },
    {
      type: 'sms',
      phoneNumbers: ['+1234567890'],
      minSeverity: 'CRITICAL'
    }
  ],
  logSources: [
    {
      type: 'syslog',
      path: '/var/log/system.log',
      format: 'syslog'
    },
    {
      type: 'application',
      path: '/var/log/application.log',
      format: 'json'
    },
    {
      type: 'api-gateway',
      url: 'http://api-gateway/logs',
      authToken: process.env.GATEWAY_LOG_TOKEN,
      format: 'json'
    }
  ],
  metricsSources: [
    {
      type: 'prometheus',
      url: 'http://prometheus:9090',
      metrics: ['auth_failures', 'request_rate', 'error_rate']
    },
    {
      type: 'cloudwatch',
      region: 'us-west-2',
      namespace: 'SecurityMetrics',
      metrics: ['UnauthorizedAccess', 'DataExfiltration']
    }
  ],
  thresholds: {
    alertThreshold: 6, // nivel de riesgo mínimo para alertar
    anomalyThreshold: 0.75, // puntuación mínima de anomalía para alertar
    metricsInterval: 60 * 1000, // cada 1 minuto
    integrityCheckInterval: 15 * 60 * 1000 // cada 15 minutos
  }
};

// Iniciar el sistema de monitoreo
const securityMonitoring = new SecurityMonitoring(monitoringConfig);
securityMonitoring.startMonitoring()
  .then(() => console.log('Sistema de monitoreo de seguridad iniciado'))
  .catch(error => console.error('Error al iniciar monitoreo:', error));
```

#### Tablero de Seguridad en Tiempo Real

Implementar un tablero para visualizar el estado de seguridad en tiempo real:

```typescript
// Implementación de un tablero de seguridad en tiempo real
class SecurityDashboard {
  private metricsSources: MetricsSource[];
  private alertsSource: AlertsSource;
  private dashboardConfig: DashboardConfig;
  
  constructor(config: DashboardConfig) {
    this.metricsSources = config.metricsSources;
    this.alertsSource = new AlertsSource(config.alertsEndpoint);
    this.dashboardConfig = config;
  }
  
  // Inicializar el tablero
  async initialize(): Promise<void> {
    // Conectar a las fuentes de datos
    await Promise.all([
      ...this.metricsSources.map(source => source.connect()),
      this.alertsSource.connect()
    ]);
    
    // Configurar actualizaciones periódicas
    setInterval(() => this.updateDashboard(), this.dashboardConfig.refreshInterval);
    
    // Escuchar alertas en tiempo real
    this.alertsSource.onNewAlert(alert => this.handleNewAlert(alert));
    
    console.log('Tablero de seguridad inicializado');
  }
  
  // Actualizar datos del tablero
  private async updateDashboard(): Promise<void> {
    try {
      // Recolectar métricas actuales
      const metrics = await this.collectCurrentMetrics();
      
      // Actualizar visualizaciones
      this.updateSecurityScorecard(metrics);
      this.updateThreatMap(metrics);
      this.updateAnomaliesChart(metrics);
      this.updateSecurityTrends(metrics);
      this.updateComplianceStatus(metrics);
      
      // Actualizar timestamp
      document.getElementById('last-update').textContent = 
        new Date().toLocaleString();
    } catch (error) {
      console.error('Error al actualizar tablero:', error);
      this.showUpdateError(error);
    }
  }
  
  // Recolectar métricas de todas las fuentes
  private async collectCurrentMetrics(): Promise<DashboardMetrics> {
    const metricsPromises = this.metricsSources.map(async source => {
      return source.getLatestMetrics();
    });
    
    const metricsResults = await Promise.all(metricsPromises);
    
    // Combinar métricas de diferentes fuentes
    return this.combineMetrics(metricsResults);
  }
  
  // Manejar una nueva alerta
  private handleNewAlert(alert: SecurityAlert): void {
    // Mostrar notificación
    this.showAlertNotification(alert);
    
    // Agregar a la lista de alertas recientes
    this.addToRecentAlerts(alert);
    
    // Si es crítica, resaltar en el tablero
    if (alert.severity === 'CRITICAL' || alert.severity === 'HIGH') {
      this.highlightCriticalAlert(alert);
    }
    
    // Reproducir sonido de alerta si está configurado
    if (this.dashboardConfig.audioAlerts) {
      this.playAlertSound(alert.severity);
    }
  }
  
  // Actualizar tarjeta de puntuación de seguridad
  private updateSecurityScorecard(metrics: DashboardMetrics): void {
    const scoreElement = document.getElementById('security-score');
    const scoreValue = this.calculateSecurityScore(metrics);
    
    // Actualizar valor numérico
    scoreElement.textContent = scoreValue.toFixed(1);
    
    // Actualizar color según el valor
    if (scoreValue >= 8) {
      scoreElement.className = 'score excellent';
    } else if (scoreValue >= 6) {
      scoreElement.className = 'score good';
    } else if (scoreValue >= 4) {
      scoreElement.className = 'score moderate';
    } else {
      scoreElement.className = 'score poor';
    }
    
    // Actualizar tendencia
    this.updateScoreTrend(metrics.securityScoreHistory);
  }
  
  // Calcular puntuación de seguridad general
  private calculateSecurityScore(metrics: DashboardMetrics): number {
    // Ponderación de factores para la puntuación
    const weights = {
      vulnerabilities: 0.3,
      compliance: 0.2,
      incidents: 0.25,
      responseTime: 0.15,
      patching: 0.1
    };
    
    // Calcular cada componente normalizado de 0-10
    const scores = {
      vulnerabilities: this.normalizeVulnerabilityScore(metrics.vulnerabilities),
      compliance: metrics.compliancePercentage / 10,
      incidents: this.normalizeIncidentsScore(metrics.incidents),
      responseTime: this.normalizeResponseTimeScore(metrics.meanTimeToRespond),
      patching: this.normalizePatchingScore(metrics.patchingStatus)
    };
    
    // Calcular puntuación ponderada
    return Object.entries(weights).reduce((score, [key, weight]) => {
      return score + (scores[key] * weight);
    }, 0);
  }
  
  // Métodos auxiliares para normalización de puntuaciones
  private normalizeVulnerabilityScore(vulnerabilities: VulnerabilityMetrics): number {
    // Lógica para normalizar puntuación de vulnerabilidades
    // ...
    return normalizedScore;
  }
  
  private normalizeIncidentsScore(incidents: IncidentMetrics): number {
    // Lógica para normalizar puntuación de incidentes
    // ...
    return normalizedScore;
  }
  
  private normalizeResponseTimeScore(responseTime: number): number {
    // Lógica para normalizar puntuación de tiempo de respuesta
    // ...
    return normalizedScore;
  }
  
  private normalizePatchingScore(patchingStatus: PatchingMetrics): number {
    // Lógica para normalizar puntuación de parcheado
    // ...
    return normalizedScore;
  }
  
  // Otros métodos para actualizar visualizaciones específicas...
}

// Iniciar el tablero
const securityDashboard = new SecurityDashboard({
  metricsSources: [
    {
      name: 'SIEM',
      endpoint: 'https://siem-api.example.com/metrics',
      authToken: 'YOUR_API_TOKEN'
    },
    {
      name: 'Vulnerability Scanner',
      endpoint: 'https://scanner-api.example.com/metrics',
      authToken: 'YOUR_SCANNER_TOKEN'
    }
  ],
  alertsEndpoint: 'wss://alerts.example.com/stream',
  refreshInterval: 30000, // 30 segundos
  audioAlerts: true
});

securityDashboard.initialize()
  .then(() => console.log('Tablero de seguridad en tiempo real iniciado'))
  .catch(error => console.error('Error al iniciar tablero:', error));
```

#### Ejercicios de Implementación

Para poner en práctica los conceptos de métricas y monitoreo:

1. **Definición de KPIs**: Identifica los indicadores clave de rendimiento (KPIs) de seguridad más relevantes para tu organización.

2. **Automatización de Métricas**: Implementa un sistema automatizado para recolectar y visualizar métricas de seguridad.

3. **Establecimiento de Líneas Base**: Determina los valores normales o esperados para las métricas clave como punto de comparación.

4. **Simulaciones de Incidentes**: Realiza simulaciones periódicas para probar la efectividad del sistema de monitoreo.

5. **Revisión Periódica**: Establece un proceso para revisar y ajustar métricas y umbrales según la evolución del sistema.

#### Beneficios de un Sistema Robusto de Métricas y Monitoreo

La implementación de un sistema efectivo de métricas y monitoreo proporciona varios beneficios:

- **Detección temprana**: Identificación de posibles amenazas antes de que causen daño significativo
- **Visibilidad mejorada**: Comprensión clara del estado de seguridad del sistema
- **Toma de decisiones basada en datos**: Priorización informada de las actividades de seguridad
- **Mejora continua**: Identificación de áreas para mejorar el proceso de modelado de amenazas
- **Demostración de valor**: Capacidad para mostrar el impacto de las inversiones en seguridad

Al integrar métricas efectivas y monitoreo continuo con el modelado de amenazas, las organizaciones pueden crear un enfoque proactivo y basado en datos para la seguridad de sus aplicaciones.

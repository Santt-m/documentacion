### 3.4.5 Análisis de Riesgos Estructurado

El análisis de riesgos complementa el modelado de amenazas proporcionando un marco para evaluar el impacto y la probabilidad de las amenazas identificadas.

#### Metodología DREAD

DREAD es una metodología de calificación de amenazas que ayuda a priorizar la mitigación de riesgos según cinco categorías:

| Letra | Categoría | Preguntas para evaluar |
|-------|-----------|------------------------|
| **D** | **D**amage (Daño) | ¿Cuál es el daño potencial si la amenaza se explota? |
| **R** | **R**eproducibility (Reproducibilidad) | ¿Qué tan fácil es reproducir el ataque? |
| **E** | **E**xploitability (Facilidad de explotación) | ¿Qué habilidades o recursos se necesitan para explotar la vulnerabilidad? |
| **A** | **A**ffected users (Usuarios afectados) | ¿Cuántos usuarios serían afectados? |
| **D** | **D**iscoverability (Facilidad de descubrimiento) | ¿Qué tan fácil es descubrir la vulnerabilidad? |

```typescript
// Implementación de evaluación DREAD en una clase auxiliar
class ThreatAssessment {
  // Evalúa una amenaza usando la metodología DREAD
  // Cada categoría se puntúa de 1 (bajo) a 3 (alto)
  assessThreat(threat: Threat): ThreatRisk {
    const scores = {
      damage: this.evaluateDamage(threat),
      reproducibility: this.evaluateReproducibility(threat),
      exploitability: this.evaluateExploitability(threat),
      affectedUsers: this.evaluateAffectedUsers(threat),
      discoverability: this.evaluateDiscoverability(threat),
    };
    
    // Calcular puntuación total (de 5 a 15)
    const totalScore = Object.values(scores).reduce((sum, score) => sum + score, 0);
    
    // Determinar nivel de riesgo basado en puntuación
    let riskLevel: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
    
    if (totalScore <= 7) riskLevel = 'LOW';
    else if (totalScore <= 10) riskLevel = 'MEDIUM';
    else if (totalScore <= 13) riskLevel = 'HIGH';
    else riskLevel = 'CRITICAL';
    
    return {
      threat,
      scores,
      totalScore,
      riskLevel,
      recommendedActions: this.getRecommendedActions(threat, riskLevel)
    };
  }
  
  private evaluateDamage(threat: Threat): number {
    // Ejemplo de lógica para evaluar el daño potencial
    if (threat.impactsConfidentialData) return 3;
    if (threat.impactsSystemAvailability) return 2;
    return 1;
  }
  
  // Métodos similares para otras categorías
  // ...
  
  private getRecommendedActions(threat: Threat, risk: string): string[] {
    const actions = [];
    
    switch (risk) {
      case 'CRITICAL':
        actions.push('Mitigar inmediatamente antes del despliegue');
        actions.push('Asignar equipo dedicado a la resolución');
        break;
      case 'HIGH':
        actions.push('Mitigar en el próximo sprint');
        actions.push('Implementar controles temporales');
        break;
      case 'MEDIUM':
        actions.push('Incluir en el backlog con prioridad normal');
        break;
      case 'LOW':
        actions.push('Documentar y monitorear');
        break;
    }
    
    // Acciones específicas según el tipo de amenaza
    if (threat.type === 'INJECTION') {
      actions.push('Implementar consultas parametrizadas');
      actions.push('Validar todas las entradas de usuario');
    } else if (threat.type === 'XSS') {
      actions.push('Implementar Content Security Policy');
      actions.push('Utilizar bibliotecas de sanitización');
    }
    
    return actions;
  }
}
```

#### Matriz de Probabilidad e Impacto

Otra técnica común para el análisis de riesgos es la matriz de probabilidad e impacto, que permite visualizar y priorizar riesgos.

```typescript
// Implementación de una matriz de riesgo para evaluar amenazas
enum Probability {
  LOW = 1,    // Improbable que ocurra
  MEDIUM = 2, // Podría ocurrir
  HIGH = 3    // Muy probable que ocurra
}

enum Impact {
  LOW = 1,    // Impacto mínimo
  MEDIUM = 2, // Impacto significativo pero manejable
  HIGH = 3    // Impacto grave o catastrófico
}

enum RiskLevel {
  LOW = 'LOW',           // Bajo riesgo
  MEDIUM = 'MEDIUM',     // Riesgo medio
  HIGH = 'HIGH',         // Alto riesgo
  CRITICAL = 'CRITICAL'  // Riesgo crítico
}

class RiskMatrix {
  // Determina el nivel de riesgo basado en probabilidad e impacto
  assessRisk(probability: Probability, impact: Impact): RiskLevel {
    const riskScore = probability * impact;
    
    if (riskScore <= 2) return RiskLevel.LOW;
    if (riskScore <= 4) return RiskLevel.MEDIUM;
    if (riskScore <= 6) return RiskLevel.HIGH;
    return RiskLevel.CRITICAL;
  }
  
  // Genera una representación visual de la matriz de riesgo
  generateMatrix(): RiskLevel[][] {
    const matrix: RiskLevel[][] = [];
    
    // Filas representan el impacto (de mayor a menor)
    for (let impact = Impact.HIGH; impact >= Impact.LOW; impact--) {
      const row: RiskLevel[] = [];
      
      // Columnas representan la probabilidad (de menor a mayor)
      for (let prob = Probability.LOW; prob <= Probability.HIGH; prob++) {
        row.push(this.assessRisk(prob, impact));
      }
      
      matrix.push(row);
    }
    
    return matrix;
  }
  
  // Evalúa una lista de amenazas y las clasifica según su nivel de riesgo
  evaluateThreats(threats: Threat[]): Record<RiskLevel, Threat[]> {
    const categorized: Record<RiskLevel, Threat[]> = {
      [RiskLevel.LOW]: [],
      [RiskLevel.MEDIUM]: [],
      [RiskLevel.HIGH]: [],
      [RiskLevel.CRITICAL]: []
    };
    
    for (const threat of threats) {
      const riskLevel = this.assessRisk(
        threat.probabilityScore as Probability,
        threat.impactScore as Impact
      );
      
      categorized[riskLevel].push(threat);
    }
    
    return categorized;
  }
  
  // Visualizar matriz en consola (utilidad para desarrollo)
  printMatrix(): void {
    const matrix = this.generateMatrix();
    const headers = ['', 'Baja', 'Media', 'Alta'];
    
    console.log('\nMatriz de Riesgo:\n');
    console.log(`${headers[0].padEnd(10)}${headers[1].padEnd(10)}${headers[2].padEnd(10)}${headers[3].padEnd(10)}`);
    
    const impactLabels = ['Alta', 'Media', 'Baja'];
    
    matrix.forEach((row, index) => {
      let rowStr = `${impactLabels[index].padEnd(10)}`;
      
      row.forEach(risk => {
        let cell = risk;
        // Colorear según nivel de riesgo (si se usa en un entorno que soporta colores)
        rowStr += `${cell.padEnd(10)}`;
      });
      
      console.log(rowStr);
    });
    
    console.log('\nLeyenda: Filas = Impacto, Columnas = Probabilidad\n');
  }
}

// Ejemplo de uso
const riskMatrix = new RiskMatrix();
riskMatrix.printMatrix();

// Ejemplo de evaluación de una lista de amenazas
const threats = [
  {
    id: 'T-001',
    name: 'Inyección SQL en formulario de login',
    probabilityScore: Probability.MEDIUM,
    impactScore: Impact.HIGH
  },
  {
    id: 'T-002',
    name: 'XSS en comentarios de usuarios',
    probabilityScore: Probability.HIGH,
    impactScore: Impact.MEDIUM
  },
  {
    id: 'T-003',
    name: 'Fuga de información en logs',
    probabilityScore: Probability.HIGH,
    impactScore: Impact.LOW
  }
];

const categorizedThreats = riskMatrix.evaluateThreats(threats);
console.log('Amenazas críticas:', categorizedThreats[RiskLevel.CRITICAL]);
console.log('Amenazas de alto riesgo:', categorizedThreats[RiskLevel.HIGH]);
```

#### Evaluación Cuantitativa del Riesgo

Para organizaciones con mayor madurez en seguridad, es posible realizar un análisis cuantitativo que exprese el riesgo en términos monetarios.

```typescript
// Implementación de Factor Analysis of Information Risk (FAIR)
class FAIRAnalysis {
  // Calcula la pérdida única esperada (SLE - Single Loss Expectancy)
  calculateSLE(assetValue: number, exposureFactor: number): number {
    return assetValue * exposureFactor;
  }
  
  // Calcula la pérdida anual esperada (ALE - Annual Loss Expectancy)
  calculateALE(singleLossExpectancy: number, annualRateOfOccurrence: number): number {
    return singleLossExpectancy * annualRateOfOccurrence;
  }
  
  // Calcula el retorno de inversión de una contramedida
  calculateROI(annualLossExpectancy: number, controlCost: number, riskReductionRatio: number): number {
    const benefit = annualLossExpectancy * riskReductionRatio;
    return (benefit - controlCost) / controlCost;
  }
  
  // Realiza un análisis completo de riesgo para una amenaza específica
  analyzeRisk(params: {
    assetValue: number;            // Valor del activo en riesgo ($)
    exposureFactor: number;        // Porcentaje del activo expuesto (0-1)
    minFrequency: number;          // Frecuencia mínima de ocurrencia anual
    maxFrequency: number;          // Frecuencia máxima de ocurrencia anual
    controlCost: number;           // Costo anual del control ($)
    estimatedReduction: number;    // Reducción estimada del riesgo (0-1)
  }): FAIRResult {
    const { 
      assetValue,
      exposureFactor,
      minFrequency,
      maxFrequency,
      controlCost,
      estimatedReduction
    } = params;
    
    // Calcular SLE
    const sle = this.calculateSLE(assetValue, exposureFactor);
    
    // Calcular frecuencia media de ocurrencia
    const avgFrequency = (minFrequency + maxFrequency) / 2;
    
    // Calcular ALE
    const ale = this.calculateALE(sle, avgFrequency);
    
    // Calcular ROI del control
    const roi = this.calculateROI(ale, controlCost, estimatedReduction);
    
    // Calcular ALE después de implementar el control
    const aleAfterControl = ale * (1 - estimatedReduction);
    
    // Calcular ahorro anual
    const annualSavings = ale - aleAfterControl - controlCost;
    
    return {
      singleLossExpectancy: sle,
      annualLossExpectancy: ale,
      returnOnInvestment: roi,
      annualLossExpectancyAfterControl: aleAfterControl,
      annualSavings: annualSavings,
      recommendation: roi > 0 ? 'Implementar control' : 'No implementar control'
    };
  }
  
  // Realiza una simulación Monte Carlo para obtener resultados más precisos
  performMonteCarloSimulation(params: {
    assetValue: [number, number];         // [min, max] valor del activo
    exposureFactor: [number, number];     // [min, max] factor de exposición
    annualFrequency: [number, number];    // [min, max] frecuencia anual
    controlCost: [number, number];        // [min, max] costo del control
    controlEffectiveness: [number, number]; // [min, max] efectividad
  }, iterations: number = 10000): MonteCarlResult {
    const results: number[] = [];
    const rois: number[] = [];
    
    for (let i = 0; i < iterations; i++) {
      // Generar valores aleatorios dentro de los rangos especificados
      const assetValue = this.randomBetween(params.assetValue[0], params.assetValue[1]);
      const exposureFactor = this.randomBetween(params.exposureFactor[0], params.exposureFactor[1]);
      const frequency = this.randomBetween(params.annualFrequency[0], params.annualFrequency[1]);
      const controlCost = this.randomBetween(params.controlCost[0], params.controlCost[1]);
      const effectiveness = this.randomBetween(params.controlEffectiveness[0], params.controlEffectiveness[1]);
      
      // Calcular SLE y ALE para esta iteración
      const sle = this.calculateSLE(assetValue, exposureFactor);
      const ale = this.calculateALE(sle, frequency);
      const roi = this.calculateROI(ale, controlCost, effectiveness);
      
      results.push(ale);
      rois.push(roi);
    }
    
    // Ordenar resultados para análisis estadístico
    results.sort((a, b) => a - b);
    rois.sort((a, b) => a - b);
    
    return {
      meanALE: results.reduce((sum, val) => sum + val, 0) / iterations,
      medianALE: results[Math.floor(iterations / 2)],
      percentile95ALE: results[Math.floor(iterations * 0.95)],
      meanROI: rois.reduce((sum, val) => sum + val, 0) / iterations,
      positiveROIProbability: rois.filter(roi => roi > 0).length / iterations,
      recommendation: rois.filter(roi => roi > 0).length / iterations > 0.75 
        ? 'Implementar control (alta confianza)'
        : rois.filter(roi => roi > 0).length / iterations > 0.5
          ? 'Implementar control (confianza moderada)'
          : 'No implementar control'
    };
  }
  
  private randomBetween(min: number, max: number): number {
    return min + Math.random() * (max - min);
  }
}

// Ejemplo de uso del análisis FAIR
const fairAnalysis = new FAIRAnalysis();

const riskAssessment = fairAnalysis.analyzeRisk({
  assetValue: 1000000,     // El sistema vale $1M
  exposureFactor: 0.3,     // Una brecha afectaría al 30% del valor
  minFrequency: 0.1,       // Ocurre entre 0.1 veces al año (una vez cada 10 años)
  maxFrequency: 0.5,       // Y 0.5 veces al año (una vez cada 2 años)
  controlCost: 50000,      // El control cuesta $50K al año
  estimatedReduction: 0.8  // Y reduciría el riesgo en un 80%
});

console.log('Análisis FAIR:', riskAssessment);

// Ejemplo de simulación Monte Carlo
const monteCarloResult = fairAnalysis.performMonteCarloSimulation({
  assetValue: [800000, 1200000],        // Valor entre $800K y $1.2M
  exposureFactor: [0.2, 0.4],           // Exposición entre 20% y 40%
  annualFrequency: [0.05, 0.6],         // Frecuencia entre 0.05 y 0.6 por año
  controlCost: [40000, 60000],          // Costo entre $40K y $60K
  controlEffectiveness: [0.7, 0.9]      // Efectividad entre 70% y 90%
});

console.log('Simulación Monte Carlo:', monteCarloResult);
```

#### Integración con la Gestión de Riesgos Empresariales

El análisis de riesgos de seguridad debe integrarse con la gestión de riesgos empresariales más amplia:

```typescript
// Integración del análisis de seguridad con ERM (Enterprise Risk Management)
class EnterpriseRiskIntegration {
  mapSecurityRiskToBusinessImpact(securityRisk: SecurityRisk): BusinessImpact {
    // Traducir riesgo técnico a impacto empresarial
    const financialImpact = this.calculateFinancialImpact(securityRisk);
    const reputationalImpact = this.calculateReputationalImpact(securityRisk);
    const regulatoryImpact = this.calculateRegulatoryImpact(securityRisk);
    const operationalImpact = this.calculateOperationalImpact(securityRisk);
    
    return {
      financialImpact,
      reputationalImpact,
      regulatoryImpact,
      operationalImpact,
      overall: this.calculateOverallBusinessImpact({
        financialImpact,
        reputationalImpact,
        regulatoryImpact,
        operationalImpact
      })
    };
  }
  
  generateExecutiveSummary(securityRisks: SecurityRisk[]): ExecutiveSummary {
    // Mapear riesgos técnicos a impactos empresariales
    const businessImpacts = securityRisks.map(risk => 
      this.mapSecurityRiskToBusinessImpact(risk)
    );
    
    // Agregar impactos por categoría
    const aggregatedFinancialImpact = this.aggregateFinancialImpact(businessImpacts);
    
    // Generar KRIs (Key Risk Indicators)
    const keyRiskIndicators = this.generateKeyRiskIndicators(securityRisks);
    
    // Generar recomendaciones priorizadas para la dirección ejecutiva
    const recommendations = this.prioritizeRecommendations(securityRisks);
    
    return {
      riskExposure: aggregatedFinancialImpact,
      keyRiskIndicators,
      recommendations,
      complianceStatus: this.assessComplianceStatus(securityRisks)
    };
  }
  
  // Métodos auxiliares para los cálculos
  private calculateFinancialImpact(risk: SecurityRisk): FinancialImpact {
    // Lógica para traducir riesgo de seguridad a impacto financiero
    // ...
    return {
      directCosts: risk.potentialDataLoss * risk.recordCost,
      indirectCosts: risk.businessDisruption * risk.hourlyRevenue,
      remediation: risk.remediationCost,
      total: 0 // Calculado basado en los valores anteriores
    };
  }
  
  // Implementaciones similares para otros métodos
  // ...
}
```

# Sistema de Stress Testing - Análisis de Riesgo de Cartera Multi-Activo

## Descripción del Proyecto

Este proyecto implementa un sistema completo de **stress testing** para gestión de riesgos de carteras multi-activo. El sistema utiliza metodologías avanzadas de análisis cuantitativo para identificar regímenes de mercado, modelar distribuciones de retornos con colas pesadas, capturar dependencias multivariadas mediante cópulas, y generar escenarios sintéticos para evaluación de riesgos.

El análisis está estructurado en **5 fases metodológicas** que van desde la preparación de datos hasta la generación y validación de escenarios de stress testing.

## Estructura del Proyecto

### **Fase 1: Descarga, Preparación y Análisis Exploratorio de Datos**
- Descarga de datos históricos de 8 ETFs representativos de diferentes clases de activos
- Cálculo de retornos logarítmicos diarios
- Limpieza y validación de datos
- Análisis exploratorio completo:
  - Estadísticas descriptivas (media, volatilidad, asimetría, curtosis)
  - Matriz de correlación lineal
  - Serie temporal de retornos acumulados (wealth index)
  - Distribuciones empíricas de retornos
  - Rolling volatility (ventana de 252 días)

### **Fase 2: Identificación de Regímenes de Mercado mediante Hidden Markov Model**
- Implementación de HMM con exactamente 2 estados (Normal y Crisis)
- Identificación de estados históricos mediante algoritmo de Viterbi
- Cálculo de matriz de transición y persistencia de estados
- Estadísticas condicionales de retornos por estado
- Análisis de correlaciones condicionales (efecto contagio en crisis)
- Identificación de períodos históricos de crisis

### **Fase 3: Modelización de Distribuciones de Retornos por Estado de Mercado**
- Ajuste de distribuciones t de Student (no paramétricas) por activo y estado
- Cálculo de métricas de riesgo:
  - Value at Risk (VaR) al 95% y 99%
  - Conditional Value at Risk (CVaR) / Expected Shortfall al 95% y 99%
- Tests de bondad de ajuste (Kolmogorov-Smirnov)
- Análisis comparativo de métricas de riesgo entre estados Normal y Crisis
- Visualización de distribuciones ajustadas

### **Fase 4: Modelización de Dependencia Multivariada mediante Cópulas t de Student**
- Transformación a pseudo-observaciones uniformes
- Ajuste de cópulas t diferenciadas por estado (Normal y Crisis)
- Estimación de matrices de correlación y grados de libertad
- Cálculo de tail dependence (dependencia en colas)
- Análisis de cambios en estructura de dependencia entre estados
- Visualización comparativa de correlaciones

### **Fase 5: Simulación Monte Carlo de Escenarios Sintéticos**
- Implementación de funciones de simulación desde cópula t
- Generación de 10,000 escenarios sintéticos por estado
- Transformación de uniformes a retornos usando distribuciones marginales
- Validación exhaustiva de escenarios:
  - Estadísticas descriptivas (empírico vs sintético)
  - Matrices de correlación
  - VaR y CVaR
  - Tests de bondad de ajuste (Kolmogorov-Smirnov)
- Análisis de coherencia y visualizaciones comparativas

## Cartera de Análisis

El sistema analiza una cartera diversificada de **8 ETFs** que representan las principales clases de activos:

| ETF | Descripción | Clase de Activo |
|-----|-------------|-----------------|
| **SPY** | S&P 500 | Acciones grandes de EE.UU. |
| **IWM** | Russell 2000 | Acciones pequeñas de EE.UU. |
| **EFA** | MSCI EAFE | Acciones desarrolladas ex-EE.UU. |
| **EEM** | MSCI Emerging Markets | Acciones mercados emergentes |
| **TLT** | Treasury Bonds 20+ años | Bonos gubernamentales largos |
| **IEF** | Treasury Bonds 7-10 años | Bonos gubernamentales intermedios |
| **HYG** | High Yield Corporate Bonds | Bonos corporativos de alto rendimiento |
| **GLD** | Gold | Oro físico |

Esta diversificación permite analizar riesgos tanto de acciones como de renta fija, así como diferentes geografías y niveles de riesgo crediticio.

## Requisitos

### Librerías de Python

```python
numpy
pandas
yfinance
matplotlib
seaborn
scipy
scikit-learn
hmmlearn
```

### Instalación

```bash
pip install numpy pandas yfinance matplotlib seaborn scipy scikit-learn hmmlearn
```

## Uso

1. Abrir el notebook `stress_testing_analysis.ipynb`
2. Ejecutar las celdas en orden secuencial, fase por fase
3. Cada fase genera resultados y visualizaciones que se utilizan en las fases posteriores
4. Las variables clave se preservan entre fases para el análisis completo

### Variables Principales Generadas

- `returns`: DataFrame con retornos logarítmicos diarios de todos los activos
- `states`: Array con la secuencia histórica de estados identificados (Normal/Crisis)
- `state_labels`: Diccionario de mapeo de estados
- `transition_matrix`: Matriz de transición entre estados
- `returns_by_state`: Retornos segregados por estado
- `fitted_distributions`: Distribuciones t de Student ajustadas por activo y estado
- `risk_metrics`: Métricas de VaR y CVaR por activo y estado
- `copula_params_normal` / `copula_params_crisis`: Parámetros de cópulas t por estado
- `scenarios_normal` / `scenarios_crisis`: Escenarios sintéticos generados (10,000 por estado)

## Metodología

### Identificación de Regímenes
- **Hidden Markov Model (HMM)** con 2 estados para identificar automáticamente períodos de mercado Normal y Crisis
- Identificación basada en volatilidad observada (Crisis = mayor volatilidad)

### Modelización de Distribuciones
- **Distribución t de Student** en lugar de normal para capturar colas pesadas
- Ajuste por máxima verosimilitud para cada activo en cada estado
- Cálculo de métricas de riesgo robustas (VaR, CVaR)

### Modelización de Dependencia
- **Cópulas t de Student** para capturar dependencia en colas (tail dependence)
- Cópulas diferenciadas por estado de mercado
- Permite modelar dependencias no lineales y asimétricas

### Generación de Escenarios
- **Simulación Monte Carlo** mediante:
  1. Simulación desde cópula t multivariada
  2. Transformación inversa de distribuciones marginales
- Validación exhaustiva contra datos empíricos

## Resultados Clave

El sistema permite:
- Identificar automáticamente períodos históricos de crisis
- Cuantificar diferencias en riesgo entre estados Normal y Crisis
- Capturar dependencias en colas (cuando más se necesita diversificación)
- Generar escenarios sintéticos coherentes para stress testing
- Evaluar impacto de diferentes condiciones de mercado en carteras

## Aplicaciones en Gestión de Riesgos

- **Stress Testing**: Evaluación de carteras bajo escenarios extremos
- **Gestión de Riesgo de Cola**: Identificación y cuantificación de eventos extremos
- **Análisis de Diversificación**: Evaluación de beneficios de diversificación en diferentes regímenes
- **Asignación de Capital**: Cálculo de requerimientos de capital basados en escenarios de stress
- **Reportes a Comités de Riesgo**: Análisis profesional orientado a presentación ejecutiva

## Notas Técnicas

- Los datos históricos se descargan desde 2000-01-01 hasta la fecha actual
- Se utilizan retornos logarítmicos (continuamente compuestos) por sus propiedades estadísticas
- El modelo HMM es compatible con uso en tiempo real para identificación de regímenes
- Los escenarios sintéticos son condicionales a estado (no predicen transiciones entre estados)

## Autor

Sistema desarrollado para análisis cuantitativo de gestión de riesgos.

## Licencia

Este proyecto es parte de un taller de gestión de riesgos.

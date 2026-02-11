# Sistema de Stress Testing - Análisis de Riesgo de Cartera Multi-Activo

Sistema completo de **stress testing** para gestión de riesgos financieros utilizando metodologías avanzadas: Hidden Markov Models (HMM) para detección de regímenes, análisis marginal de riesgo, cópulas para modelización de dependencia multivariante, y simulaciones Monte Carlo para escenarios de estrés.

## 📊 Estructura del Proyecto

```
┌─────────────────────────────────────────────────────────────┐
│                    FASE 0: Setup y Datos                    │
│  • Descarga datos (2006-2026)                                │
│  • 5 indicadores + 18 activos                                │
│  • Retornos logarítmicos                                    │
└───────────────────────┬─────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│          FASE 1: Detección de Regímenes (HMM)                │
│  • HMM Gaussiano 2 estados                                   │
│  • Identifica Calma vs Crisis                                │
│  • Validación histórica (2008, 2020, 2022)                   │
└───────────────────────┬─────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│        FASE 2: Análisis Marginal del Riesgo                  │
│  • Estadísticas por régimen                                  │
│  • Volatilidad, skewness, kurtosis                           │
│  • Análisis específico: GLD, HYG                             │
└───────────────────────┬─────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│     FASE 3: Dependencia y Diversificación (Cópulas)          │
│  • Correlaciones por régimen                                 │
│  • Cópulas Gaussianas multivariantes                         │
│  • Tail dependence                                           │
└───────────────────────┬─────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│      FASE 4: Motor de Simulación Monte Carlo                 │
│  • PortfolioSimulator (HMM + Marginales + Cópulas)           │
│  • 10,000 simulaciones × 126 días                            │
│  • Validación exhaustiva                                     │
└───────────────────────┬─────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│            FASE 5: Escenarios de Estrés                      │
│  • Escenario 1: Estanflación 2022                            │
│  • Escenario 2: Crisis Crédito 2008                           │
│  • Escenario 3: Pérdida Confianza Crediticia USA             │
│  • VaR 99% y CVaR 99% por escenario                          │
└─────────────────────────────────────────────────────────────┘
```

## 🎯 Resumen Ejecutivo

Este proyecto implementa un sistema completo de análisis de riesgo que:

1. **Descarga y prepara datos** de 18 activos financieros (2006-2026)
2. **Detecta regímenes de mercado** (Calma vs Crisis) usando HMM
3. **Analiza el riesgo marginal** de cada activo por régimen
4. **Modela dependencias multivariantes** usando cópulas Gaussianas
5. **Simula trayectorias futuras** mediante Monte Carlo (10,000 simulaciones)
6. **Evalúa escenarios de estrés** históricos y alternativos

## 📁 Estructura de Fases

### **Fase 0: Setup y Datos**
- Descarga desde Yahoo Finance (2006-2026)
- **5 indicadores de mercado**: ^GSPC, ^VIX, TLT, IEF, HYG
- **18 activos en cartera**: AAPL, AMZN, BAC, BRK-B, CVX, ENPH, GLD, GME, GOOGL, JNJ, JPM, MSFT, NVDA, PG, XOM, IEF, ^IRX, HYG
- Política de **no imputación**: cada activo desde su fecha real de inicio
- Cálculo de retornos logarítmicos

### **Fase 1: Detección de Regímenes de Mercado**
- HMM Gaussiano de 2 estados (Calma vs Crisis)
- Features: retornos estandarizados, volatilidad móvil, spread de crédito, VIX
- Validación histórica: identifica crisis de 2008, 2020, 2022
- Outputs: `hmm_model`, `states_df`, `state_calm`, `state_crisis`

### **Fase 2: Análisis Marginal del Riesgo**
- Estadísticas por régimen: media, volatilidad, skewness, kurtosis
- Análisis específico: Oro (GLD) como activo refugio, High Yield (HYG)
- Outputs: `marginal_stats`, `returns_with_regime`

### **Fase 3: Dependencia y Diversificación (Cópulas)**
- Matrices de correlación de Pearson por régimen
- Ajuste de cópulas Gaussianas multivariantes por régimen
- Análisis de tail dependence (dependencia en colas)
- Outputs: `corr_calm`, `corr_crisis`, `copula_models`

### **Fase 4: Motor de Simulación Monte Carlo**
- Clase `PortfolioSimulator` que integra HMM, marginales y cópulas
- Simulación de 10,000 trayectorias de 126 días (6 meses)
- Validación exhaustiva: reproduce volatilidades, drawdowns y riesgo de cola
- Outputs: `simulator`, `simulation_results`, `calculate_risk_metrics()`

### **Fase 5: Escenarios de Estrés**
- **Escenario 1**: Estanflación 2022 (histórico)
- **Escenario 2**: Crisis de Crédito 2008 (histórico)
- **Escenario 3**: Pérdida de Confianza Crediticia de EE.UU. (alternativo)
- Cada escenario: 10,000 simulaciones, cálculo de VaR 99% y CVaR 99%
- Comparación de métricas entre escenarios

## 🔧 Requisitos

```bash
pip install numpy pandas yfinance matplotlib seaborn scipy hmmlearn copulas
```

## 🚀 Uso

Ejecutar el notebook `Tarea_riesgos.ipynb` en orden secuencial:

```
Fase 0 → Fase 1 → Fase 2 → Fase 3 → Fase 4 → Fase 5
```

## 📈 Hallazgos Clave

1. **Volatilidad**: Se amplifica 2-4× en crisis vs calma
2. **Correlaciones**: Aumentan sistemáticamente en crisis (diversificación falla)
3. **Tail dependence**: Co-movimientos extremos aumentan dramáticamente en crisis
4. **Oro (GLD)**: Evalúa si mantiene correlaciones bajas en crisis
5. **Stress Testing**: Los escenarios alternativos muestran pérdidas potenciales más extremas que los históricos

## 📚 Documentación

La documentación completa del proyecto se encuentra en la carpeta [`docs/`](docs/):
- [`docs/Practica_Gestion_Riesgos (1).pdf`](docs/Practica_Gestion_Riesgos%20(1).pdf) - Documentación técnica del proyecto

## 🔬 Metodología

- **HMM**: Detección automática de regímenes Calma/Crisis
- **Cópulas**: Modelización de dependencia multivariante más flexible que correlaciones lineales
- **Monte Carlo**: Simulación de 10,000 trayectorias para cuantificar incertidumbre
- **Universe dinámico**: Sin imputación de datos, cada activo desde su fecha real de inicio

## 📊 Variables Principales por Fase

| Fase | Variables Clave |
|------|----------------|
| **0** | `data_market`, `data_portfolio`, `returns_market`, `returns_portfolio` |
| **1** | `hmm_model`, `states_df`, `state_calm`, `state_crisis` |
| **2** | `marginal_stats`, `returns_with_regime` |
| **3** | `corr_calm`, `corr_crisis`, `copula_models` |
| **4** | `simulator`, `simulation_results`, `calculate_risk_metrics()` |
| **5** | `scenario1_sim_metrics`, `scenario2_sim_metrics`, `scenario3_sim_metrics` |

## 🎓 Contexto del Escenario 3

El **Escenario 3 (Pérdida de Confianza Crediticia de EE.UU.)** está contextualizado en la realidad actual:
- EE.UU. perdió su calificación AAA en mayo de 2025
- Actualmente (2026) se encuentra en Aa1/AA+ debido al aumento del déficit y deuda pública
- El escenario simula una pérdida adicional de confianza crediticia moderada

## 📝 Notas

- Todos los datos se descargan desde Yahoo Finance
- El sistema es completamente reproducible (semilla aleatoria: 42)
- Las simulaciones utilizan cópulas calibradas con fallback robusto a Gaussianas
- Los escenarios de estrés incluyen validación histórica cuando hay datos disponibles

---

**Autor**: Sistema de Gestión de Riesgos  
**Fecha**: 2026  
**Versión**: 1.0

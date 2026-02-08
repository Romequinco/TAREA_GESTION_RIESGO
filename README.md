# Sistema de Stress Testing - Análisis de Riesgo de Cartera Multi-Activo

Sistema completo de **stress testing** para gestión de riesgos financieros utilizando metodologías avanzadas: Hidden Markov Models (HMM) para detección de regímenes, análisis marginal de riesgo, y cópulas para modelización de dependencia multivariante.

## Estructura del Proyecto

### **Fase 0: Setup y Datos**
- Descarga de datos desde Yahoo Finance (2006-2025)
- **Indicadores de mercado** (5): ^GSPC, ^VIX, TLT, IEF, HYG
- **Cartera equiponderada** (18 activos): AAPL, AMZN, BAC, BRK-B, CVX, ENPH, GLD, GME, GOOGL, JNJ, JPM, MSFT, NVDA, PG, XOM, IEF, ^IRX, HYG
- Política de **no imputación**: cada activo se analiza solo desde su fecha real de inicio
- Cálculo de retornos logarítmicos

### **Fase 1: Detección de Regímenes de Mercado**
- HMM Gaussiano de 2 estados (Calma vs Crisis)
- Comienza desde fecha de inicio de HYG (2007-04-11) para incluir este indicador clave
- Features: retornos estandarizados (^GSPC, TLT, IEF, HYG), volatilidad móvil S&P 500, spread de crédito, nivel VIX
- Validación histórica: identifica correctamente crisis de 2008, 2020, 2022
- Outputs: `hmm_model`, `states_df`, `state_calm`, `state_crisis`

### **Fase 2: Análisis Marginal del Riesgo**
- Estadísticas por régimen: media, volatilidad, skewness, kurtosis
- Cada activo analizado solo desde su fecha real de inicio
- Análisis específico: Oro (GLD) como activo refugio, High Yield (HYG)
- Outputs: `marginal_stats`, `returns_with_regime`

### **Fase 3: Dependencia y Diversificación (Cópulas)**
- Matrices de correlación de Pearson por régimen
- Demostración: correlaciones aumentan en crisis → diversificación falla
- Análisis de correlaciones del Oro (GLD)
- Ajuste de cópulas Gaussianas multivariantes por régimen
- Análisis de tail dependence (dependencia en colas)
- Outputs: `corr_calm`, `corr_crisis`, `copula_models`, `tail_dep_df`

## Requisitos

```bash
pip install numpy pandas yfinance matplotlib seaborn scipy hmmlearn copulas
```

## Uso

Ejecutar el notebook `Tarea_riesgos.ipynb` en orden secuencial (Fase 0 → 1 → 2 → 3).

## Variables Principales

**Fase 0**: `data_market`, `data_portfolio`, `returns_market`, `returns_portfolio`, `start_date_by_asset`

**Fase 1**: `hmm_model`, `states_df`, `state_calm`, `state_crisis`, `X_hmm`

**Fase 2**: `marginal_stats`, `returns_with_regime`, `comparison_df`

**Fase 3**: `corr_calm`, `corr_crisis`, `copula_models`, `tail_dep_df`

## Hallazgos Clave

1. **Volatilidad**: Se amplifica 2-4× en crisis vs calma
2. **Correlaciones**: Aumentan sistemáticamente en crisis (diversificación falla)
3. **Tail dependence**: Co-movimientos extremos aumentan dramáticamente en crisis
4. **Oro (GLD)**: Evalúa si mantiene correlaciones bajas en crisis

## Metodología

- **HMM**: Detección automática de regímenes Calma/Crisis
- **Cópulas**: Modelización de dependencia multivariante más flexible que correlaciones lineales
- **Universe dinámico**: Sin imputación de datos, cada activo desde su fecha real de inicio

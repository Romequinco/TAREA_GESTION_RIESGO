# Sistema de Stress Testing - Análisis de Riesgo de Cartera Multi-Activo

## Descripción del Proyecto

Este proyecto implementa un sistema completo de **stress testing** para gestión de riesgos financieros utilizando metodologías avanzadas de análisis cuantitativo. El sistema identifica regímenes de mercado mediante Hidden Markov Models (HMM), modela distribuciones de retornos con colas pesadas, captura dependencias multivariadas mediante cópulas, y genera escenarios sintéticos para evaluación de riesgos bajo diferentes condiciones de mercado.

El análisis está estructurado en **múltiples fases metodológicas** que van desde la preparación y limpieza de datos hasta la generación y validación de escenarios de stress testing.

## Estructura del Proyecto

### **Fase 0: Setup y Datos**

**Objetivo**: Configurar el entorno de trabajo completo, descargar todos los datos necesarios desde Yahoo Finance, realizar la limpieza y preparación de datos, y dejar todo listo para que las fases posteriores puedan ejecutarse sin interrupciones.

**Componentes principales**:
- **Indicadores de estado de mercado** (5 indicadores para HMM):
  - `^GSPC`: S&P 500 (mercado de acciones)
  - `^VIX`: CBOE Volatility Index (volatilidad implícita)
  - `TLT`: iShares 20+ Year Treasury Bond ETF (tipos largo plazo)
  - `IEF`: iShares 7-10 Year Treasury Bond ETF (tipos medio plazo)
  - `HYG`: iShares iBoxx $ High Yield Corporate Bond ETF (crédito)

- **Cartera equiponderada** (18 activos):
  - **Acciones (15)**: AAPL, AMZN, BAC, BRK-B, CVX, ENPH, GLD, GME, GOOGL, JNJ, JPM, MSFT, NVDA, PG, XOM
  - **Bonos gobierno USA (2)**: IEF (bono 10 años proxy), ^IRX (bono 2 años proxy)
  - **High Yield (1)**: HYG

**Procesamiento de datos**:
- Descarga de datos diarios desde 2006-01-01 hasta fecha actual
- Tratamiento de valores faltantes (forward fill + backward fill)
- Verificación de outliers extremos (z-score > 5)
- Alineamiento temporal de todos los DataFrames
- Cálculo de retornos logarítmicos

**Outputs**:
- `data_market`: DataFrame con precios de indicadores de estado
- `data_portfolio`: DataFrame con precios de activos de la cartera
- `returns_market`: DataFrame con retornos logarítmicos de indicadores
- `returns_portfolio`: DataFrame con retornos logarítmicos de la cartera
- `market_indicators`: Lista de tickers para indicadores
- `portfolio_tickers`: Lista de 18 tickers de la cartera

**Visualizaciones**:
1. Serie temporal de indicadores de estado (con períodos de crisis resaltados)
2. Evolución de la cartera equiponderada (escala logarítmica)
3. Mapa de calor de valores faltantes
4. Distribución de retornos (S&P 500, HYG, NVDA)

---

### **Fase 1: Detección de Regímenes de Mercado**

**Objetivo**: Ajustar un Gaussian Hidden Markov Model (HMM) de 2 estados sobre los retornos de mercado para identificar, día a día, en qué régimen se encuentra el mercado: **Estado 0 (Calma)** vs **Estado 1 (Crisis)**.

**Metodología**:
- **Preparación de features**: Transformación de datos para capturar el "pulso" del mercado:
  - Retornos estandarizados (z-scores) de ^GSPC, TLT, IEF, HYG
  - Volatilidad móvil (20 días) del S&P 500
  - Spread de crédito (HYG - IEF)
  - Nivel absoluto de VIX (normalizado)

- **Estimación del HMM**:
  - 2 estados (Calma vs Crisis)
  - Covarianza completa (`covariance_type='full'`)
  - Múltiples inicializaciones aleatorias (10 modelos, selección del mejor)
  - 1000 iteraciones mínimas, tolerancia 1e-4

- **Decodificación de estados**: Algoritmo de Viterbi para asignar cada día histórico al estado más probable

- **Identificación de regímenes**: Análisis estadístico para etiquetar estados:
  - Comparación de volatilidad S&P 500
  - Comparación de niveles de VIX
  - Comparación de retornos medios
  - Análisis de performance de HYG

**Análisis de matriz de transición**:
- Probabilidades de transición entre regímenes
- Duración esperada de cada régimen
- Distribución estacionaria (long-run)
- Interpretación económica de persistencia

**Validación histórica**:
- Verificación de coherencia con crisis conocidas:
  - Crisis Financiera Global (2008-2009)
  - Crisis de Deuda Europea (2011)
  - Taper Tantrum (2013)
  - Sell-off Q4 2018
  - COVID-19 (2020)
  - Inflación y subida de tipos (2022)

**Outputs**:
- `hmm_model`: Modelo HMM entrenado
- `hidden_states`: Array de estados (0 o 1) por cada día
- `states_df`: DataFrame con columnas ['date', 'state', 'regime']
- `state_calm`: Índice del estado de calma
- `state_crisis`: Índice del estado de crisis
- `X_hmm`: Matriz de features usada para entrenar el HMM

**Visualizaciones**:
1. **S&P 500 coloreado por régimen** (obligatoria): Blanco = Calma, Azul = Crisis
2. Timeline de regímenes con fechas clave marcadas
3. Distribución de duraciones de episodios por régimen

---

### **Fases Futuras** (En desarrollo)

- **Fase 2**: Modelización de distribuciones de retornos por estado de mercado
- **Fase 3**: Modelización de dependencia multivariada mediante cópulas
- **Fase 4**: Simulación Monte Carlo de escenarios sintéticos
- **Fase 5**: Validación y análisis de resultados de stress testing

---

## Requisitos Técnicos

### Librerías de Python

```python
numpy              # Cálculos numéricos y matrices
pandas             # Manipulación de series temporales
yfinance           # Descarga de datos financieros
matplotlib         # Visualizaciones básicas
seaborn            # Visualizaciones estadísticas
scipy              # Estadística (skewness, kurtosis, distribuciones)
hmmlearn           # Hidden Markov Models (GaussianHMM)
copulas            # Ajuste de cópulas para modelización de dependencia
warnings           # Suprimir warnings innecesarios
```

### Instalación

```bash
pip install numpy pandas yfinance matplotlib seaborn scipy hmmlearn copulas
```

### Configuración Global

El proyecto utiliza las siguientes configuraciones obligatorias:
- Semilla aleatoria: `42` (reproducibilidad)
- Estilo de gráficos: `seaborn-v0_8-darkgrid`
- Warnings: Suprimidos para limpiar output

---

## Uso

### Ejecución del Notebook

1. Abrir el notebook `Tarea_riesgos.ipynb`
2. Ejecutar las celdas en orden secuencial, fase por fase
3. Cada fase genera resultados y visualizaciones que se utilizan en las fases posteriores
4. Las variables globales se preservan entre fases para el análisis completo

### Orden de Ejecución

1. **Fase 0**: Setup y datos (obligatorio ejecutar primero)
2. **Fase 1**: Detección de regímenes de mercado (requiere Fase 0)
3. **Fases 2-5**: En desarrollo

### Variables Principales Generadas

**Fase 0**:
- `data_market`, `data_portfolio`: DataFrames de precios
- `returns_market`, `returns_portfolio`: DataFrames de retornos logarítmicos
- `market_indicators`, `portfolio_tickers`: Listas de tickers
- `start_date`, `end_date`: Fechas de inicio y fin

**Fase 1**:
- `hmm_model`: Modelo HMM entrenado
- `hidden_states`: Array de estados históricos (0 o 1)
- `states_df`: DataFrame con estados y regímenes etiquetados
- `state_calm`, `state_crisis`: Índices de estados identificados
- `X_hmm`: Matriz de features para HMM

---

## Metodología

### Identificación de Regímenes

- **Hidden Markov Model (HMM)** con 2 estados para identificar automáticamente períodos de mercado **Calma** y **Crisis**
- Identificación basada en análisis estadístico de:
  - Volatilidad del S&P 500
  - Niveles de VIX
  - Retornos medios
  - Performance de crédito (HYG)

### Features para HMM

El modelo utiliza features transformadas en lugar de retornos directos para:
- Normalizar escalas heterogéneas (VIX vs retornos)
- Capturar volatilidad móvil (rolling window)
- Medir spread de crédito (diferencia HYG - IEF)
- Incorporar nivel absoluto de VIX (indicador directo de "miedo")

### Validación de Coherencia

El modelo valida su coherencia identificando correctamente:
- Crisis Financiera Global (2008-2009)
- COVID-19 (2020)
- Inflación y subida de tipos (2022)
- Otros períodos de estrés histórico

---

## Resultados Clave

El sistema permite:
- ✅ Identificar automáticamente períodos históricos de crisis con alta precisión
- ✅ Cuantificar diferencias en riesgo entre estados Calma y Crisis
- ✅ Analizar persistencia y duración de regímenes de mercado
- ✅ Generar visualizaciones interpretables para presentación ejecutiva
- ✅ Validar coherencia con eventos económicos históricos conocidos

---

## Aplicaciones en Gestión de Riesgos

- **Stress Testing**: Evaluación de carteras bajo escenarios extremos identificados por el modelo
- **Detección Temprana de Crisis**: Identificación de transiciones a regímenes de crisis
- **Análisis de Persistencia**: Cuantificación de duración esperada de crisis
- **Gestión de Riesgo de Cola**: Identificación y cuantificación de eventos extremos
- **Reportes a Comités de Riesgo**: Análisis profesional orientado a presentación ejecutiva

---

## Notas Técnicas

- **Período de datos**: 2006-01-01 hasta fecha actual (datos diarios)
- **Retornos**: Se utilizan retornos logarítmicos (continuamente compuestos) por sus propiedades estadísticas (aditividad temporal)
- **Reproducibilidad**: Semilla aleatoria fijada en 42 para garantizar resultados reproducibles
- **Tratamiento de datos**: Forward fill + backward fill para valores faltantes, alineamiento temporal estricto
- **Modelo HMM**: Compatible con uso en tiempo real para identificación de regímenes

---

## Estructura de Archivos

```
TAREA_GESTION_RIESGO/
├── Tarea_riesgos.ipynb          # Notebook principal con todas las fases
├── README.md                    # Este archivo
├── PRUEBAS.ipynb                # Notebook de pruebas (legacy)
└── Practica_Gestion_Riesgos.pdf # Documento de referencia
```

---

## Autor

Sistema desarrollado para análisis cuantitativo de gestión de riesgos financieros.

## Licencia

Este proyecto es parte de un taller de gestión de riesgos.

---

## Contacto y Soporte

Para preguntas o problemas relacionados con el proyecto, consultar la documentación en el notebook o revisar los comentarios inline en el código.

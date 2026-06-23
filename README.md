#  Análisis Comparativo de Modelos Tradicionales y de Aprendizaje Profundo para la Predicción de Precios de Acciones

Código y datos del Trabajo de Fin de Grado de **Enrique Crespillo Ramos**
(Grado en Matemáticas y Ciencia de Datos, Facultad de Ciencias Matemáticas, UCM, curso 2025/2026):
*Análisis Comparativo de Modelos Tradicionales y de Aprendizaje Profundo para la
Predicción de Precios de Acciones.*

El estudio compara cinco modelos (Naïve, ARIMA, Random Forest, XGBoost y LSTM) en la
predicción del **retorno logarítmico** de cuatro acciones del S&P 500 (AAPL, JPM, JNJ,
XOM) y de Bitcoin, en horizontes de 1 y 5 días, con validación *walk-forward*,
intervalos *bootstrap* y test de Diebold-Mariano. Incluye una exploración
complementaria con un Transformer.

## Estructura del repositorio

```
TFG/
├── README.md
├── requirements.txt
├── .gitignore
├── notebooks/
│   ├── anexo_codigo.ipynb           # Notebook principal: descarga y exploración de datos,
│   │                                #   5 modelos, 2 horizontes, clasificación, robustez y BTC intradía
│   ├── lstm_grid_search.ipynb       # Búsqueda de hiperparámetros de la LSTM (AAPL)
│   ├── transformers_diario.ipynb    # Transformer + RevIN (exploración complementaria)
│   └── anexo_correcciones_v2.ipynb  # DM a 5 días con h=5 e IC bootstrap de clasificación
├── Datos/
│   ├── Stocks/   # AAPL, JPM, JNJ, XOM (CSV)
│   └── BTC/      # BTC/USDT en 1d, 4h, 1h y 15m (CSV)
├── Resultados/   # Métricas exportadas (CSV)
└── Figuras/      # Figuras generadas (PNG)
```

## Notebooks

| Notebook | Contenido |
|---|---|
| `anexo_codigo.ipynb` | Notebook principal, ejecutado de principio a fin. Tres bloques: datos y análisis exploratorio (descarga de las acciones con `yfinance`, carga de los CSV de Bitcoin, VIX, construcción de variables y estadísticos descriptivos); los cinco modelos sobre acciones y BTC diario (1d y 5d) con selección de variables, *walk-forward*, *bootstrap*, Diebold-Mariano, clasificación binaria y robustez (Huber, ARIMA sin *look-ahead*); y Bitcoin en 4h, 1h y 15m. |
| `lstm_grid_search.ipynb` | Explora 54 configuraciones de la LSTM sobre AAPL en 1d y 5d. Comprueba que el resultado negativo de la LSTM no se debe a un mal ajuste de hiperparámetros. |
| `transformers_diario.ipynb` | Transformer de auto-atención con RevIN (PyTorch) bajo el mismo protocolo que el anexo. Resultado: segundo hallazgo negativo (pierde frente al Naïve en los diez pares). |
| `anexo_correcciones_v2.ipynb` | Recalcula el Diebold-Mariano a 5 días con `h=5` (ventana HAC para *targets* solapados) y los IC *bootstrap* de la clasificación, sin reejecutar todo el pipeline. |

Todos los notebooks se entregan **ejecutados, con sus salidas**. No hace falta volver a
correrlos para leer los resultados.

## Datos

- **Acciones**: AAPL, JPM, JNJ, XOM (S&P 500), precios diarios ajustados 2015–2026, vía
  `yfinance`. `anexo_codigo.ipynb` los descarga y guarda en `Datos/Stocks/`, donde ya
  están incluidos.
- **Bitcoin**: BTC/USDT en cuatro temporalidades (1d, 4h, 1h, 15m), 2018–2026, OHLCV de
  Binance, en `Datos/BTC/`.

## Entorno y ejecución

El proyecto usa Python 3.10 o superior. Las dependencias están en `requirements.txt`:

```bash
pip install -r requirements.txt
```

Cada notebook define `BASE_DIR` con la ruta a la raíz del proyecto y, a partir de ella,
las carpetas `Datos/`, `Resultados/` y `Figuras/`. En otra máquina solo hay que sustituir
esa ruta por la local. El `transformers_diario.ipynb` está en PyTorch y preparado para
Google Colab, con una línea `BASE_DIR` alternativa para Drive.

Se ejecuta primero `anexo_codigo.ipynb` (descarga los datos y genera los CSV de
métricas) y, después, `lstm_grid_search.ipynb`, `transformers_diario.ipynb` y
`anexo_correcciones_v2.ipynb`, que leen esos CSV de `Resultados/`.

## Reproducibilidad

Las semillas se fijan a 42 en NumPy, scikit-learn, XGBoost (`random_state`) y TensorFlow
(`tf.random.set_seed`). La partición es estrictamente cronológica (entrenamiento hasta
2022-12-31, validación en 2023, test desde 2024-01-01) y los escaladores se ajustan solo
con datos de entrenamiento. La ejecución es determinista salvo la variabilidad numérica
menor que el entrenamiento de la LSTM introduce según el hardware. La robustez frente a
la semilla se comprueba con tres valores (7, 42 y 123).

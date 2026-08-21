# Estrategia de reversión a la media del PER en TradingView

Esta estrategia busca aprovechar desviaciones del PER de una empresa respecto a su comportamiento reciente. Utiliza precios diarios y únicamente resultados empresariales que ya habían sido publicados en cada fecha del backtest.

La especificación funcional completa se encuentra en [Plan.md](Plan.md).

## Requisitos

- Una cuenta de TradingView con acceso al Pine Editor.
- Un gráfico de una empresa con resultados trimestrales disponibles.
- Tipo de gráfico estándar de velas o barras.
- Temporalidad de 1 día.

No está diseñada para ETF, índices, criptomonedas ni activos sin resultados trimestrales compatibles.

## Cómo cargar la estrategia

1. Abre TradingView y carga el gráfico de una empresa, por ejemplo NASDAQ:MSFT.
2. Selecciona un gráfico estándar con temporalidad **1D**.
3. Abre la pestaña **Pine Editor**.
4. Crea una estrategia nueva y sustituye su contenido por el de pe_mean_reversion_strategy.pine.
5. Guarda el script.
6. Pulsa **Añadir al gráfico**.
7. Abre la configuración de la estrategia para elegir el período del backtest y revisar el resto de parámetros.
8. Consulta las operaciones, la curva de patrimonio y las métricas en el informe de estrategia de TradingView.

## Parámetros principales

- **Ventana del PER:** número de días naturales usados para estimar su media y dispersión.
- **Mínimo de aperturas válidas:** observaciones exigidas antes de permitir operaciones.
- **Distancia de las bandas:** separación de los niveles de compra y venta respecto a la media.
- **Inicio y fin:** fechas inclusivas del backtest. La fecha final debe ser una sesión bursátil.
- **Tasa libre de riesgo:** referencia utilizada para calcular el Sharpe diario personalizado.
- **Visualización:** permite ocultar o mostrar bandas, publicaciones, señales, fondo de estado y tabla resumen.

El capital inicial, el tamaño de las posiciones y otras propiedades del emulador pueden revisarse desde la pestaña **Propiedades** de la estrategia.

## Principios de la estrategia

### Información punto-en-tiempo

El BPA TTM se forma con los cuatro últimos resultados trimestrales publicados. Un resultado nuevo no afecta a la apertura de su propia fecha de publicación: empieza a utilizarse en la siguiente sesión.

El PER se considera inválido si todavía no existen cuatro informes compatibles, falta algún BPA o la suma de los cuatro BPA es cero o negativa. Mientras sea inválido no se abren posiciones y una posición existente se cierra.

### Reversión a la media

Cada apertura se compara con la distribución de los PER observados durante los últimos 365 días naturales:

- Se compra cuando el PER de apertura está en la banda inferior o por debajo.
- Se vende cuando el PER de apertura está en la banda superior o por encima.
- La observación actual no participa en las bandas contra las que se compara.
- Antes de operar se exige una ventana temporal completa y un mínimo de 200 aperturas válidas.

La estrategia es exclusivamente compradora: no abre cortos, no piramida y no utiliza stop-loss ni órdenes límite.

## Ejecución simulada

La señal utiliza información disponible en la apertura, pero la operación se simula al cierre de esa misma sesión. Esto representa una orden Market-on-Close preparada durante el día.

Se aplica un coste del 0,035% tanto en la compra como en la venta. El nominal reserva una pequeña parte del patrimonio para cubrir el coste de entrada sin utilizar apalancamiento.

En la última sesión del período no se abren posiciones nuevas y cualquier posición existente se liquida al cierre.

## Cómo interpretar los resultados

El panel inferior muestra el PER de apertura, su media histórica y las bandas de compra y venta. Los marcadores E identifican fechas de publicación de resultados, y las etiquetas de compra y venta aparecen sobre el gráfico principal.

La tabla resumen incluye:

- Estado actual del modelo.
- BPA TTM disponible para la próxima sesión.
- PER, media y bandas.
- Rentabilidad total.
- Máximo drawdown.
- Duración máxima del drawdown.
- Sharpe anualizado.
- Número de operaciones.

Los Pine Logs se limitan a los eventos importantes: disponibilidad del modelo, publicaciones de resultados, cambios a BPA inválido y órdenes de entrada o salida.

## Supuestos del backtest

- Los datos diarios y fundamentales proporcionados por TradingView son correctos.
- La fecha asociada a un resultado representa su fecha de publicación.
- Un resultado se utiliza siempre desde la sesión siguiente, aunque se publicara antes de la apertura.
- Las señales conocidas durante la sesión pueden prepararse para una ejecución Market-on-Close.
- El precio de cierre de la vela es el precio de ejecución empleado por el emulador.
- Los costes son constantes y no dependen de liquidez, tamaño o volatilidad.
- El Sharpe personalizado utiliza retornos diarios y 252 sesiones anuales.

## Limitaciones

- TradingView puede corregir retrospectivamente datos fundamentales; Pine no permite auditar todas sus versiones históricas.
- La disponibilidad y calidad del BPA varía entre empresas y mercados.
- El gráfico diario no permite distinguir con precisión publicaciones anteriores o posteriores a la apertura.
- Una ejecución real enviada al terminar la sesión puede no conseguir el mismo precio de cierre que el emulador.
- No se modelan impuestos, impacto de mercado, deslizamiento variable, dividendos ni restricciones específicas de cada bróker.
- El resultado depende de los parámetros, del período elegido y del universo analizado.
- Un backtest favorable no garantiza resultados futuros.

## Comprobaciones antes de usar los resultados

1. Confirma que los marcadores de resultados coinciden con los eventos mostrados por TradingView.
2. Verifica que un BPA nuevo empieza a afectar al PER en la sesión posterior.
3. Revisa que las operaciones se ejecutan al cierre de la vela que contiene la señal.
4. Comprueba que la fecha final corresponde a una sesión bursátil.
5. Confirma en el informe que no existen liquidaciones por margen.
6. Compara la rentabilidad y el drawdown del panel con el informe nativo.

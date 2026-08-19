# TradingView Backtesting: reversión a la media del PER

Estrategia long-only para TradingView que prueba una reversión a la media del PER de una empresa usando precios diarios y BPA trimestral publicado. El modelo evita utilizar el BPA de un informe antes de su fecha de publicación.

La especificación completa y sus decisiones de diseño están en [Plan.md](Plan.md).

## Archivos

- `pe_mean_reversion_strategy.pine`: estrategia Pine Script v6 lista para copiar en TradingView.
- `Plan.md`: definición funcional, supuestos y criterios de prueba.

## Cómo utilizarla

1. Abre en TradingView el gráfico de una empresa con resultados trimestrales disponibles, por ejemplo `NASDAQ:MSFT`.
2. Selecciona un gráfico estándar de **1 día**. La estrategia genera un error intencionado en otros marcos temporales.
3. Abre el Pine Editor, copia el contenido de `pe_mean_reversion_strategy.pine` y pulsa **Añadir al gráfico**.
4. Revisa en **Configuración → Inputs** las fechas, la ventana, el multiplicador de las bandas y la tasa libre de riesgo.
5. Consulta las operaciones y la curva de patrimonio en el informe de estrategia de TradingView.

La estrategia se declara con un capital inicial de 100.000 unidades monetarias y usa el 100% del patrimonio en cada posición. Estos valores pueden cambiarse en las propiedades de la estrategia.

## Secuencia punto-en-tiempo

En cada sesión diaria se respeta este orden:

1. Se conserva el BPA TTM que era conocido antes de la apertura.
2. Se calcula `PER_apertura = open / BPA_TTM`.
3. El PER de apertura se compara con las bandas finalizadas en la sesión anterior.
4. Si existe señal, se compra o vende al cierre de la misma sesión mediante una orden de mercado Market-on-Close simulada con `process_orders_on_close = true`.
5. El PER de la apertura se añade a la ventana histórica.
6. Si TradingView vincula un informe a esa fecha, su BPA se incorpora al final del cálculo y solo puede utilizarse desde la sesión siguiente.

Esta secuencia impide que el BPA publicado durante una fecha intervenga en el PER de la apertura de esa misma fecha.

## Fundamentales y bandas

La fecha de cada evento se detecta con `request.earnings(..., earnings.actual)` y el BPA empleado en el cálculo es `earnings.standardized`, que TradingView define como BPA diluido GAAP estandarizado.

El BPA TTM es la suma de los cuatro últimos informes consecutivos. Si uno no tiene BPA estandarizado, o la suma es cero o negativa, el PER se considera inválido. En ese estado no se abren posiciones y cualquier posición existente se cierra al final de la primera sesión afectada.

La distribución utiliza el PER punto-en-tiempo de las aperturas contenidas en los últimos 365 días naturales:

- Compra: `media − 0,89 × desviación estándar`.
- Venta: `media + 0,89 × desviación estándar`.
- Desviación estándar poblacional.
- Calentamiento mínimo: una ventana natural completa y 200 aperturas válidas.

La observación del día actual no participa en las bandas contra las que ella misma se compara.

## Costes y ejecución

Cada compra o venta soporta un coste del **0,035%**:

- 2,5 puntos básicos de medio spread.
- 1 punto básico de comisión.
- 0,07% para una operación completa de entrada y salida.

El emulador de TradingView rellena las órdenes en el cierre de la vela de señal. Esto representa una orden Market-on-Close preparada durante la sesión; no garantiza que una alerta real emitida después del cierre pueda obtener ese mismo precio.

No se utilizan posiciones cortas, apalancamiento, piramidación, órdenes límite ni stop-loss.

## Resultados

Además del informe nativo de TradingView, el script muestra un resumen con:

- Rentabilidad total, incluido el beneficio o pérdida abierto.
- Máximo drawdown intradiario calculado por TradingView.
- Máxima duración close-to-close por debajo del máximo de equity, en días naturales.
- Sharpe anualizado a partir de retornos diarios y 252 sesiones anuales.
- BPA TTM, PER, media, bandas y número de operaciones.

La tasa libre de riesgo del Sharpe personalizado es configurable y vale 2% por defecto. El Sharpe nativo de TradingView también usa 2%, pero se calcula con retornos mensuales y puede diferir.

## Alcance y limitaciones

- La primera versión está diseñada para empresas con cuatro eventos trimestrales compatibles. ETF e índices como SPY o SPX normalmente mostrarán `BPA TTM no válido` y no operarán.
- Las fechas y valores dependen del proveedor de datos de TradingView.
- Pine permite alinear el BPA con el día del informe, pero no expone versiones históricas auditables de todas las correcciones posteriores del proveedor.
- La estrategia ignora deliberadamente si un informe fue publicado antes o después de la apertura y espera siempre a la sesión siguiente para utilizarlo.
- El backtest no garantiza resultados futuros.

## Comprobaciones recomendadas

Antes de interpretar resultados:

1. Activa los eventos de resultados de TradingView y confirma que los marcadores `E` coinciden con ellos.
2. Comprueba que el nuevo BPA TTM aparece en el panel al cierre del informe y que el PER no lo utiliza hasta la siguiente barra.
3. Revisa varias entradas y salidas: la señal debe depender del PER de apertura y el precio ejecutado debe ser el cierre de la misma vela.
4. Compara el máximo drawdown del panel con el informe nativo.
5. Repite la prueba con costes distintos desde las propiedades si quieres modelar otro bróker.

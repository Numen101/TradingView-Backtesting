# Backtest de reversión a la media del PER en TradingView

## Resumen

Crear una estrategia Pine Script v6 para gráficos diarios estándar de empresas con fundamentales. Será long-only, asignará prácticamente el 100% del patrimonio entre posición y coste de entrada, y reconstruirá el BPA TTM usando los cuatro últimos resultados publicados.

La señal se calculará con el PER de apertura y las bandas conocidas antes de comenzar la sesión. Cuando se active, la operación se ejecutará mediante una orden de mercado al cierre de esa misma sesión.

## Datos y modelo punto-en-tiempo

- Solicitar `earnings.actual` para detectar las fechas de publicación y `earnings.standardized` para obtener el BPA diluido GAAP.
- Usar `barmerge.gaps_on` y `barmerge.lookahead_off`.
- Incorporar un resultado únicamente después del cierre de su fecha de publicación. No usar la fecha de fin del trimestre.
- Mantener los cuatro resultados trimestrales consecutivos más recientes:
  - `BPA TTM = suma de los cuatro BPA estandarizados`.
  - El PER será inválido si falta algún trimestre o el BPA TTM es cero o negativo.
- TradingView confirma que `request.earnings()` asocia el valor al informe publicado, mientras que `request.financial()` adelanta el dato al comienzo del período fiscal siguiente. [Datos financieros en Pine](https://www.tradingview.com/support/solutions/43000564727-what-financial-data-is-available-in-pine/), [definición de Earnings](https://www.tradingview.com/support/solutions/43000629790-earnings/).
- Calcular cada día:
  - `PER_apertura = open / BPA_TTM_conocido_antes_de_la_sesión`.
  - Media y desviación estándar poblacional del PER punto-en-tiempo de los últimos 365 días naturales.
  - Banda de compra: `media − 0,89σ`.
  - Banda de venta: `media + 0,89σ`.
- Las bandas utilizadas en una apertura serán las calculadas al cierre anterior. El PER de la apertura actual se añadirá a la ventana después de evaluar la señal, evitando que la observación se incluya en su propio umbral.
- No operar hasta disponer de una ventana completa y al menos 200 aperturas válidas.

## Señales y ejecución

- Entrada:
  - Estar fuera del mercado.
  - PER válido.
  - `PER_apertura <= banda_compra_anterior`.
  - Comprar al cierre de esa sesión con el nominal máximo que, sumado al coste de entrada, no exceda el 100% del patrimonio.
- Salida:
  - Tener una posición abierta.
  - `PER_apertura >= banda_venta_anterior`.
  - Vender toda la posición al cierre de esa sesión.
- Si el PER queda inválido, cerrar la posición al cierre de la primera sesión que abra con ese estado y suspender las compras.
- Configurar `process_orders_on_close = true` para que TradingView rellene las órdenes de mercado en el cierre de la vela que genera la operación. [Ejecución de órdenes en TradingView](https://www.tradingview.com/pine-script-docs/faq/strategies/#why-are-my-orders-executed-on-the-bar-following-my-triggers).
- Los resultados publicados en una fecha solo modificarán el BPA utilizado desde la siguiente sesión bursátil, incluso si el informe se publicó antes de la apertura.
- La fecha final debe coincidir con una sesión bursátil. No abrir nuevas posiciones ese día y liquidar cualquier posición existente en su cierre.
- Sin piramidación, posiciones cortas, apalancamiento, órdenes límite ni stop-loss.
- Aplicar un coste del 0,035% por compra o venta:
  - 2,5 bps de medio spread.
  - 1 bp de comisión.
  - Coste completo de ida y vuelta: 0,07%.

## Archivos, configuración y resultados

Crear:

- `pe_mean_reversion_strategy.pine`: estrategia completa Pine v6.
- `README.md`: instrucciones en español, metodología y limitaciones.

Inputs principales:

- Ventana: 365 días naturales.
- Multiplicador simétrico: 0,89σ.
- Tasa libre de riesgo: 2%.
- Fecha inicial y final del backtest.
- Controles de visualización.

Salidas:

- Panel inferior con PER de apertura, media y bandas.
- Marcadores de resultados y estado de validez del BPA.
- Operaciones de compra y venta sobre el gráfico principal.
- Curva de patrimonio e informe nativo de TradingView.
- Tabla resumen con:
  - Resultado total, incluido P/L abierto.
  - Máximo drawdown intradiario de TradingView, mostrado con signo negativo.
  - Máxima duración close-to-close por debajo del máximo de equity, en días naturales, incluyendo el drawdown actual si no se ha recuperado.
  - Sharpe anualizado con retornos diarios, 252 sesiones y tasa libre de riesgo configurable.
  - Número de operaciones, PER, BPA TTM y bandas actuales.

El Sharpe personalizado puede diferir del nativo porque TradingView calcula su métrica con retornos mensuales. [Sharpe Ratio de TradingView](https://www.tradingview.com/support/solutions/43000681694-sharpe-ratio/).

## Pruebas

- Verificar en Microsoft sobre gráfico 1D que el BPA de un informe no interviene en el PER de la propia fecha de publicación.
- Confirmar que la primera apertura que usa el nuevo BPA es la siguiente sesión bursátil.
- Comprobar que la señal compara el PER de apertura actual con las bandas congeladas del cierre anterior.
- Verificar que el precio de cada operación coincide con el cierre de la misma vela diaria que contiene la señal.
- Validar manualmente BPA TTM, media, desviación y bandas en varias fechas.
- Comprobar la expulsión de observaciones con más de 365 días naturales.
- Confirmar el coste de 0,035% en cada lado.
- Confirmar que el dimensionamiento de la entrada no genera operaciones `Margin Call`.
- Comparar resultado total y máximo drawdown con el informe nativo.
- Probar BPA negativo, trimestre ausente, historial insuficiente y activos como SPY o SPX: deberán permanecer sin operar y mostrar el motivo.

## Supuestos y limitaciones

- “Misma sesión” significa señal basada en la apertura y ejecución en el cierre, no ejecución retroactiva en la apertura.
- El relleno exacto al cierre es una convención del emulador. TradingView advierte que una orden o alerta generada cuando la sesión ya termina podría no obtener ese precio en operativa real.
- Conceptualmente, la señal está disponible desde la apertura y podría utilizarse para preparar una orden Market-on-Close real, aunque la estrategia Pine se calcule al terminar la vela histórica.
- La primera versión solo admite empresas con cuatro eventos trimestrales de BPA compatibles; índices y ETF quedan fuera.
- Pine no proporciona versiones históricas auditables de cada revisión del proveedor. Se evita el adelanto por período fiscal, pero no puede garantizarse que TradingView nunca haya corregido retrospectivamente un BPA.

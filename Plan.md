# Backtest de compra por PER y distancia a SMA 68 en TradingView

## Resumen

Crear una estrategia Pine Script v6 para gráficos diarios estándar de empresas con fundamentales. Será long-only, ajustará mediante compras la posición a un apalancamiento objetivo de 1,6×, reconstruirá el BPA TTM con los cuatro últimos resultados publicados y solo venderá en la última vela completa del gráfico.

La señal combinará el PER de apertura respecto al PER mínimo conocido de los tres años naturales anteriores con una distancia extrema de la apertura por debajo de la SMA 68 conocida. Cuando se active, la compra se ejecutará mediante una orden de mercado al cierre de esa misma sesión.

## Datos y modelo punto-en-tiempo

- Solicitar `earnings.actual` para detectar fechas de publicación y `earnings.standardized` para obtener el BPA diluido GAAP.
- Usar `barmerge.gaps_on` y `barmerge.lookahead_off`.
- Incorporar un resultado únicamente después del cierre de su fecha de publicación.
- Mantener los cuatro resultados trimestrales consecutivos más recientes.
- Calcular `BPA TTM` como la suma de esos cuatro resultados.
- Considerar el PER inválido si falta algún trimestre o el BPA TTM es cero o negativo.
- Calcular `PER apertura = apertura / BPA TTM conocido antes de la sesión`.
- Mantener una ventana móvil de tres años naturales con los PER válidos anteriores a la sesión actual.
- Calcular el PER mínimo de esa ventana antes de incorporar la observación actual.
- Calcular `d = ln(PER apertura / PER mínimo)`.
- No habilitar compras hasta tener una ventana temporal completa y al menos 200 aperturas válidas por defecto.

## Modelo de distancia a la SMA

- Calcular una SMA de 68 cierres.
- Cuando el cierre esté por debajo de la SMA, calcular `D = (SMA − cierre) / SMA`.
- Agrupar las distancias en ventanas consecutivas, no solapadas y de 30 días naturales desde la primera SMA disponible.
- Al completar cada ventana, añadir su distancia máxima a una lista histórica persistente.
- No añadir ningún valor para una ventana sin cierres bajo la SMA.
- Excluir siempre la ventana actual incompleta de la referencia.
- Mantener intacta la lista histórica de máximos; no borrar valores al aplicar el filtro.
- Para cada evaluación, obtener el máximo de la lista y crear una selección temporal con los valores mayores o iguales a `máximo × 0,20`.
- Calcular la referencia como la mediana de esa selección temporal.
- Calcular la distancia de la apertura con la SMA disponible antes de la sesión: `D apertura = (SMA previa − apertura) / SMA previa`.

## Señales y ejecución

- Tener un PER válido y el modelo preparado.
- Exigir que `d < límite`; la comparación es estricta.
- Exigir además que exista una referencia SMA, que la apertura esté bajo la SMA 68 previa y que `D apertura > mediana SMA`; la comparación también es estricta.
- Evaluar la condición en cada sesión, sin exigir un cruce del límite.
- Comprar al cierre el nominal adicional necesario para que `valor posición / equity = 1,6` después de la comisión.
- No hacer nada cuando el apalancamiento ya sea igual o superior a 1,6×, porque no se permiten ventas de ajuste.
- Volver a comprar cuando ambos criterios sigan activos y el apalancamiento haya descendido por debajo de 1,6×.
- No abrir una posición nueva en la última sesión efectiva del período.
- No vender por distancia, BPA inválido ni fecha final configurada.
- Vender toda la posición al cierre de la última vela completa disponible en el gráfico.
- Mantener posiciones exclusivamente largas, sin órdenes límite ni stop-loss.
- Configurar margen largo del 25%, que permite hasta 4× en el emulador, conservando un objetivo de órdenes de 1,6×.
- Aplicar un coste del 0,035% a cada compra y a la venta final.

## Evaluación de límites

- Proporcionar diez límites de distancia editables con valores iniciales desde 0,00 hasta 0,18 en pasos de 0,02.
- Permitir seleccionar uno de ellos para generar la orden nativa del Strategy Tester.
- Simular en paralelo diez carteras apalancadas independientes, con todos sus reajustes mediante compras y una venta en la última vela completa.
- Registrar para cada límite el número de compras y la primera fecha de compra.
- Valorar cada cartera al cierre de la última vela completa del gráfico.
- Mostrar la rentabilidad de cada cartera después de las comisiones de entrada y salida.
- Mostrar **Sin compra** cuando un límite nunca se active.

## Visualización y resultados

- Panel inferior con distancia logarítmica, límite activo, nivel cero, distancia SMA actual y mediana SMA.
- PER actual, PER mínimo, SMA previa, máximo de las ventanas y corte del 20% disponibles en la ventana de datos.
- Marcadores de publicaciones de resultados y de la compra correspondiente al límite activo.
- Tabla resumen para la estrategia nativa.
- Tabla comparativa para los diez límites con límite, número de compras, primera fecha y rentabilidad.
- Pine Logs limitados a disponibilidad del modelo, publicaciones, cambios a BPA inválido y compra.

## Pruebas

- Verificar que el BPA de un informe no interviene en el PER de su propia fecha de publicación.
- Confirmar que la primera apertura que usa el nuevo BPA es la siguiente sesión bursátil.
- Comprobar que la ventana utiliza exactamente los tres años naturales anteriores.
- Confirmar que la observación actual no participa en el mínimo contra el que se compara.
- Validar manualmente varios cálculos de `ln(PER actual / PER mínimo)`.
- Probar el límite cero con nuevos mínimos, incluidas distancias negativas.
- Verificar que no se opera antes de completar la ventana ni sin el mínimo de observaciones.
- Confirmar que todas las sesiones con `d < límite` intentan ajustar el apalancamiento y que `d >= límite` nunca genera compras.
- Confirmar que las ventanas SMA son consecutivas, no se solapan y solo guardan su máximo al completarse.
- Confirmar que una ventana sin distancias válidas no añade un máximo.
- Verificar que la lista acumulada permanece intacta y que el filtro del 20% solo se aplica a una selección temporal para calcular la mediana.
- Confirmar que una distancia SMA igual a la mediana no compra y que debe ser estrictamente mayor.
- Validar que cada compra deja el cociente entre posición y equity aproximadamente en 1,6 después de la comisión.
- Confirmar que no hay ventas de ajuste cuando el apalancamiento supera 1,6×.
- Confirmar que los diez límites se evalúan de forma independiente.
- Comparar la fila activa con la orden del Strategy Tester.
- Confirmar el coste del 0,035% en ambos lados y una única venta en la última vela completa.
- Probar BPA negativo, trimestre ausente, historial insuficiente y activos sin resultados compatibles.

## Supuestos y limitaciones

- La señal se basa en la apertura y se ejecuta al cierre de la misma sesión como convención Market-on-Close.
- Las ventanas SMA duran exactamente 30×24 horas y no equivalen a meses naturales.
- El historial SMA emplea cierres; la señal utiliza la apertura y la SMA conocida al cierre anterior para evitar anticipación.
- Los datos fundamentales pueden ser corregidos retrospectivamente por TradingView.
- La fecha final configurada limita las entradas, pero la posición se mantiene hasta la última vela completa del gráfico para cerrar la operación en el Strategy Tester.
- Cada límite puede acumular múltiples compras, pero solo genera una venta voluntaria al final.
- El emulador puede ejecutar ventas forzosas por `Margin Call`; la comparación personalizada no las reproduce.
- Un resultado histórico favorable no garantiza resultados futuros.

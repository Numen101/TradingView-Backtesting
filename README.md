# Estrategia de compra por distancia al PER mínimo

Esta estrategia compra una empresa cuando su PER punto-en-tiempo está suficientemente cerca del PER mínimo observado durante los tres años naturales anteriores. Utiliza precios diarios y únicamente resultados empresariales que ya habían sido publicados en cada fecha del backtest.

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
4. Crea una estrategia nueva y sustituye su contenido por el de `pe_mean_reversion_strategy.pine`.
5. Guarda el script y pulsa **Añadir al gráfico**.
6. Elige cuál de los diez límites debe usar el Strategy Tester y configura el período del backtest.
7. Consulta el informe nativo para el límite activo y la tabla comparativa para los diez límites.

## Cálculo punto-en-tiempo

El BPA TTM se forma con los cuatro últimos resultados trimestrales publicados. Un resultado nuevo no afecta a la apertura de su propia fecha de publicación: empieza a utilizarse en la siguiente sesión.

El PER de apertura se calcula como:

`PER actual = apertura / BPA TTM conocido antes de la sesión`

El PER se considera inválido si todavía no existen cuatro informes compatibles, falta algún BPA o la suma de los cuatro BPA es cero o negativa. Un PER inválido no puede generar una compra, pero tampoco provoca una venta después de entrar.

Antes de evaluar la señal de cada sesión, la estrategia conserva los PER válidos de los tres años naturales anteriores y obtiene su mínimo. La observación actual todavía no pertenece a esa ventana, por lo que nunca se compara consigo misma.

La distancia empleada es:

`d = ln(PER actual / PER mínimo de los tres años anteriores)`

Una distancia igual a cero indica que ambos PER coinciden. Una distancia negativa indica un nuevo mínimo respecto al historial disponible y una distancia positiva mide cuánto se ha separado el PER actual del mínimo en escala logarítmica.

## Señal y ejecución

La estrategia compra si se cumplen simultáneamente estas condiciones:

- La fecha pertenece al período del backtest y no es su última sesión.
- El BPA TTM y el PER son válidos.
- Hay tres años naturales completos de historial.
- La ventana contiene al menos el número configurado de aperturas válidas, 200 por defecto.
- `d < límite`.

La condición se evalúa en todas las sesiones. Mientras `d < límite`, la estrategia calcula el nominal adicional necesario para que el valor de la posición represente 1,6 veces el equity después de descontar la comisión de la nueva compra. Si el apalancamiento ya es igual o superior a 1,6×, no compra ni vende para reducirlo. Si posteriormente cae por debajo de 1,6× y la distancia continúa bajo el límite, vuelve a comprar para reajustarlo.

La señal utiliza datos conocidos en la apertura y cada compra se simula al cierre de esa sesión mediante `process_orders_on_close = true`.

Después de comprar, la estrategia mantiene la posición aunque cambie el BPA, el PER deje de ser válido o termine el período configurado. La única venta se ejecuta al cierre de la última vela completa disponible en el gráfico, para que TradingView registre una operación cerrada en el Strategy Tester.

Se aplica un coste del 0,035% a cada compra y a la venta final. El tamaño de cada orden incorpora su propia comisión al resolver el ajuste a 1,6×.

La estrategia configura un margen de mantenimiento del 25%, equivalente a una capacidad máxima teórica de 4×, pero sus órdenes tienen como objetivo 1,6×. Este margen adicional reduce el riesgo de una liquidación inmediata; una pérdida suficientemente grande todavía puede provocar un `Margin Call` automático del emulador de TradingView.

## Comparación de diez límites

Los diez límites son editables. Sus valores iniciales son `0,00`, `0,02`, `0,04`, `0,06`, `0,08`, `0,10`, `0,12`, `0,14`, `0,16` y `0,18`.

El input **Límite usado por el Strategy Tester** decide cuál genera las órdenes nativas. La tabla inferior derecha evalúa simultáneamente los diez límites como carteras apalancadas independientes y muestra:

- El límite evaluado.
- El número de compras efectuadas.
- La fecha de su primera compra.
- La rentabilidad hasta la última vela completa del gráfico, incluyendo todas las comisiones de compra y la comisión de venta.

La fila resaltada corresponde al límite activo del Strategy Tester. Si un límite no llega a generar señal, la tabla muestra **Sin compra**.

## Paneles y métricas

El panel inferior representa la distancia logarítmica, el límite activo y el nivel cero. El PER actual y su mínimo histórico también están disponibles en la ventana de datos.

La tabla superior derecha resume el estado del modelo, BPA TTM, PER, mínimo histórico, distancia, límite activo, apalancamiento objetivo, rentabilidad, máximo drawdown, duración máxima del drawdown, Sharpe anualizado y número de compras para la estrategia nativa.

El informe nativo y la tabla comparativa tienen responsabilidades distintas: el informe muestra un único límite seleccionado; la tabla calcula los diez escenarios a la vez. Ambos valoran la posición en la última vela completa disponible en el gráfico.

## Supuestos y limitaciones

- Los datos diarios y fundamentales proporcionados por TradingView son correctos.
- La fecha asociada a un resultado representa su fecha de publicación.
- Un resultado se utiliza siempre desde la sesión siguiente, aunque se publicara antes de la apertura.
- Las señales conocidas durante la sesión pueden prepararse para una ejecución Market-on-Close.
- El precio de cierre de la vela es el precio de ejecución empleado por el emulador.
- Los costes son constantes y no dependen de liquidez, tamaño o volatilidad.
- TradingView puede corregir retrospectivamente datos fundamentales; Pine no permite auditar todas sus versiones históricas.
- La disponibilidad y calidad del BPA varía entre empresas y mercados.
- No se modelan impuestos, impacto de mercado, deslizamiento variable, dividendos ni restricciones específicas de cada bróker.
- La comparación de diez límites usa la comisión definida en el código; si se sobrescribe desde las propiedades del Strategy Tester, su fila activa puede dejar de coincidir exactamente con la simulación personalizada.
- La simulación personalizada no reproduce los `Margin Call` automáticos de TradingView ni posibles redondeos de cantidades negociables, por lo que puede diferir del Strategy Tester tras movimientos extremos.
- Cada escenario puede realizar múltiples compras, pero solo genera una venta voluntaria en la última vela completa.
- Un backtest favorable no garantiza resultados futuros.

## Comprobaciones recomendadas

1. Confirma que los marcadores de resultados coinciden con los eventos mostrados por TradingView.
2. Verifica que un BPA nuevo empieza a afectar al PER en la sesión posterior.
3. Comprueba que el PER mínimo solo contiene observaciones de los tres años anteriores y excluye la apertura actual.
4. Valida manualmente varias distancias con `ln(PER actual / PER mínimo)`.
5. Comprueba que no hay compras antes de completar tres años y el mínimo de observaciones.
6. Verifica que cada límite solo compra cuando la distancia es estrictamente menor y el apalancamiento está por debajo de 1,6×.
7. Confirma que la única orden de venta se genera en la última vela completa del gráfico.
8. Comprueba después de cada compra que `valor de la posición / equity` queda aproximadamente en 1,6.
9. Compara la fila del límite activo con el número de compras y la rentabilidad mostrados por el Strategy Tester, teniendo en cuenta las limitaciones descritas arriba.

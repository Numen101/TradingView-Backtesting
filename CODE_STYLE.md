# Criterios de estilo

## Condiciones con varios estados

Usar `if / else if / else` en lugar de encadenar operadores ternarios. Así, la prioridad y los estados posibles quedan explícitos.

```pine
string status = if not complete
    "Faltan " + str.tostring(4 - reportCount) + " informes"
else if not valuesAvailable
    "Falta BPA estandarizado"
else if not valid
    "BPA TTM <= 0"
else
    "BPA válido"
```

Reservar el operador ternario para decisiones simples de dos alternativas.

## Variables descriptivas para resultados lógicos

Asignar las condiciones o cálculos relevantes a variables con nombres descriptivos y usar esas variables en el resultado de la función.

```pine
bool insideBacktestPeriod = currentDate >= startDate and currentDate <= endDate
bool onFinalBacktestSession = currentDate == endDate
bool finalBacktestSessionMissing = currentDate > endDate and
     (na(previousDate) or previousDate < endDate)

[insideBacktestPeriod, onFinalBacktestSession, finalBacktestSessionMissing]
```

Evitar devolver expresiones complejas directamente cuando un nombre permite entender qué representa cada elemento del resultado.

## Nomenclatura descriptiva

Usar nombres que permitan entender la responsabilidad y el significado de tipos, campos, variables, funciones y métodos sin tener que revisar su implementación.

- Nombrar los tipos y clases con sustantivos que describan el estado o concepto que representan.
- Nombrar las funciones y métodos según la acción o el resultado que producen.
- Formular los booleanos como condiciones o estados reconocibles, por ejemplo `hasEnoughObservations` o `isModelReady`.
- Incluir el dominio, el contexto temporal o la unidad de medida cuando sean relevantes, por ejemplo `perValues`, `equityAtClose` o `maximumDrawdownDurationMilliseconds`.
- Mantener una nomenclatura base coherente entre los valores devueltos por una función y las variables que los reciben. El llamador puede añadir contexto, como `BeforeOpen` o `NextSession`.
- Evitar nombres genéricos o abreviaturas poco evidentes, como `data`, `values`, `tmp` o `m2`, cuando exista una alternativa más precisa.

Evitar también nombres innecesariamente largos o redundantes cuando el tipo o el ámbito reducido ya proporcionen suficiente contexto.

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

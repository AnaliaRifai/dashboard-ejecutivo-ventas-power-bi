# Documentación de medidas DAX | Dashboard Ejecutivo de Ventas

Este archivo reúne las **17 medidas DAX** utilizadas en el análisis comercial de 2024. Las métricas principales de ventas, unidades y transacciones consideran únicamente los pedidos con estado `Entregado`. Los demás estados se conservan en el modelo para posibles análisis adicionales.

> **Criterio de interpretación:** las cifras de ventas se refieren a facturación por pedidos entregados, no a utilidad. Las medidas temporales están orientadas al análisis mensual del dataset de 2024.

## 1. Métricas base

### Ventas_Totales

Calcula la facturación de los pedidos entregados, considerando cantidad, precio unitario y descuento.

```dax
Ventas_Totales =
CALCULATE(
    SUMX(
        TblFactVentas,
        TblFactVentas[PrecioUnitario] *
        TblFactVentas[Cantidad] *
        (1 - TblFactVentas[Descuento])
    ),
    TblFactVentas[EstadoPedido] = "Entregado"
)
```

### Cantidad_Vendida

Suma las unidades incluidas en pedidos entregados.

```dax
Cantidad_Vendida =
CALCULATE(
    SUM(TblFactVentas[Cantidad]),
    TblFactVentas[EstadoPedido] = "Entregado"
)
```

### Total_Transacciones

Cuenta los identificadores de venta únicos correspondientes a pedidos entregados.

```dax
Total_Transacciones =
CALCULATE(
    DISTINCTCOUNT(TblFactVentas[VentaID]),
    TblFactVentas[EstadoPedido] = "Entregado"
)
```

### Ticket_Promedio

Calcula el importe promedio por transacción entregada.

```dax
Ticket_Promedio =
DIVIDE(
    [Ventas_Totales],
    [Total_Transacciones]
)
```

## 2. Análisis temporal

### Ventas Periodo Anterior

Calcula las ventas del mes previo cuando el contexto contiene un único mes. Deja en blanco la fila de total anual y los contextos con varios meses. Enero queda sin comparación por falta de datos de diciembre de 2023.

```dax
Ventas Periodo Anterior =
IF(
    NOT HASONEVALUE('Tabla Calendario'[NúmeroMes]),
    BLANK(),
    CALCULATE(
        [Ventas_Totales],
        DATEADD('Tabla Calendario'[Date], -1, MONTH)
    )
)
```

### Variacion de Ventas

Calcula la diferencia absoluta de facturación respecto del mes anterior. Si la medida anterior es cero o no tiene valor, no presenta una variación interpretable.

```dax
Variacion de Ventas =
IF(
    [Ventas Periodo Anterior] = 0,
    BLANK(),
    [Ventas_Totales] - [Ventas Periodo Anterior]
)
```

### Variacion %

Expresa el cambio mensual como porcentaje. Se muestra en blanco cuando no hay un único mes seleccionado o no existe base de comparación.

```dax
Variacion % =
IF(
    NOT HASONEVALUE('Tabla Calendario'[NombreMesLargo]),
    BLANK(),
    DIVIDE(
        [Variacion de Ventas],
        [Ventas Periodo Anterior],
        BLANK()
    )
)
```

### Variacion % Tarjeta

Medida de **presentación** para la tarjeta `VARIACIÓN %`. Conserva la medida numérica `[Variacion %]` para cálculos y otras visualizaciones, y devuelve un texto informativo en los casos en que no corresponde mostrar un porcentaje. Al utilizar `FORMAT`, esta medida devuelve texto y se usa únicamente en la tarjeta.

```dax
Variacion % Tarjeta =
IF(
    NOT HASONEVALUE('Tabla Calendario'[NombreMesLargo]),
    "Seleccioná un mes",
    IF(
        ISBLANK([Variacion %]),
        "Sin período anterior",
        FORMAT([Variacion %], "0.0%")
    )
)
```

**Comprobaciones en el dashboard:**

- **Vista general (sin un único mes seleccionado):** muestra `Seleccioná un mes`.
- **Febrero de 2024:** muestra `42,8 %`, consistente con el insight automático (`42,80 %`).
- **Enero de 2024:** muestra `Sin período anterior`, ya que no se dispone de diciembre de 2023 para la comparación.

Esta medida corrige el `--` que aparecía en la tarjeta general sin cambiar el cálculo de variación mensual.
### Ventas Acumuladas

Calcula las ventas acumuladas desde el comienzo del año hasta la fecha del contexto.

```dax
Ventas Acumuladas =
IF(
    ISBLANK([Ventas_Totales]),
    BLANK(),
    TOTALYTD(
        [Ventas_Totales],
        'Tabla Calendario'[Date]
    )
)
```

## 3. Métricas de negocio

### Participacion Ventas %

Calcula la contribución de cada cliente respecto de las ventas del contexto seleccionado, sin limitar el denominador al cliente de la fila. Esta versión se utiliza **para el ranking de clientes**; para productos u otras dimensiones se requiere adaptar el campo cuyo filtro se elimina.

```dax
Participacion Ventas % =
DIVIDE(
    [Ventas_Totales],
    CALCULATE(
        [Ventas_Totales],
        REMOVEFILTERS(TblDimCliente[Cliente])
    )
)
```

### Ranking Productos

Ordena los productos según su facturación en el contexto de filtros seleccionado.

```dax
Ranking Productos =
IF(
    ISINSCOPE(TblDimProducto[Producto]),
    RANKX(
        ALLSELECTED(TblDimProducto[Producto]),
        [Ventas_Totales],
        ,
        DESC
    ),
    BLANK()
)
```

### Ranking Clientes

Ordena los clientes según su facturación en el contexto de filtros seleccionado.

```dax
Ranking Clientes =
IF(
    ISINSCOPE(TblDimCliente[Cliente]),
    RANKX(
        ALLSELECTED(TblDimCliente[Cliente]),
        [Ventas_Totales],
        ,
        DESC
    ),
    BLANK()
)
```

## 4. Metas y KPI

### Meta Ventas

Suma las metas correspondientes a los años, meses y canales seleccionados. `TREATAS` lleva esos filtros a `TblMetasVentas`, que no tiene relaciones físicas con las otras tablas.

```dax
Meta Ventas =
VAR AniosSeleccionados =
    VALUES('Tabla Calendario'[Año])
VAR MesesSeleccionados =
    VALUES('Tabla Calendario'[NúmeroMes])
VAR CanalesSeleccionados =
    VALUES(TblDimCanal[CanalID])
RETURN
CALCULATE(
    SUM(TblMetasVentas[MetaVentas]),
    TREATAS(AniosSeleccionados, TblMetasVentas[Año]),
    TREATAS(MesesSeleccionados, TblMetasVentas[Mes_Num]),
    TREATAS(CanalesSeleccionados, TblMetasVentas[CanalID])
)
```

### Cumplimiento Meta

Mide las ventas entregadas como proporción de la meta comercial correspondiente.

```dax
Cumplimiento Meta =
DIVIDE(
    [Ventas_Totales],
    [Meta Ventas],
    0
)
```

### Diferencia Contra Meta

Calcula en unidades monetarias cuánto superan o cuánto faltan las ventas respecto del objetivo.

```dax
Diferencia Contra Meta =
[Ventas_Totales] - [Meta Ventas]
```

### Estado KPI

Clasifica el nivel de cumplimiento: `Cumplida` (100 % o más), `Cerca` (entre 90 % y menos de 100 %) o `No Cumplida` (menos de 90 %).

```dax
Estado KPI =
SWITCH(
    TRUE(),
    [Cumplimiento Meta] >= 1, "Cumplida",
    [Cumplimiento Meta] >= 0.9, "Cerca",
    "No Cumplida"
)
```

## 5. Insight automático

### Insight Automático

Genera una oración con las ventas, la variación mensual cuando existe comparación válida y el producto con mayor facturación dentro del contexto seleccionado. El texto resume resultados, pero no identifica sus causas.

```dax
Insight Automático =
VAR Ventas = [Ventas_Totales]
VAR Variacion = [Variacion %]
VAR ProductoTop =
    TOPN(
        1,
        ALLSELECTED(TblDimProducto[Producto]),
        [Ventas_Totales],
        DESC
    )
VAR NombreProductoTop =
    CONCATENATEX(
        ProductoTop,
        TblDimProducto[Producto],
        ", "
    )
VAR TextoVariacion =
    SWITCH(
        TRUE(),
        ISBLANK(Variacion),
            "no se dispone de información suficiente para comparar con el período anterior",
        Variacion > 0,
            "registrando un crecimiento del " & FORMAT(Variacion, "0.00%"),
        Variacion < 0,
            "mostrando una caída del " & FORMAT(ABS(Variacion), "0.00%"),
        "sin variación respecto al período anterior"
    )
RETURN
"Las ventas alcanzaron "
& FORMAT(Ventas, "$ #,##0")
& " durante el período seleccionado, "
& TextoVariacion
& ". El producto con mejor desempeño fue "
& NombreProductoTop
& "."
```

## Validación de referencia: 2024

Con todos los segmentadores sin filtros adicionales, la versión revisada del proyecto obtuvo:

| Indicador | Resultado |
|---|---:|
| Ventas de pedidos entregados | $1.244.108,50 |
| Unidades entregadas | 2.406 |
| Meta anual | $1.303.553 |
| Cumplimiento | 95,44 % |
| Diferencia contra la meta | -$59.444,50 |
| Estado KPI | Cerca |

**Notas de alcance:** la participación porcentual documentada es la utilizada por cliente; las variaciones se probaron para los meses de 2024. Esta documentación recoge las fórmulas finales informadas y validadas durante la revisión del proyecto, no los borradores anteriores incluidos en la bitácora.

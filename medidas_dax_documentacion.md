# Documentación de medidas DAX | Análisis de ventas y cumplimiento de metas

Este archivo reúne las **22 medidas DAX documentadas** del análisis comercial de 2024 (17 originales y cinco nuevas de rentabilidad validadas en la versión local). Las métricas principales de ventas, unidades y transacciones consideran únicamente los pedidos con estado `Entregado`. Los demás estados se conservan en el modelo para posibles análisis adicionales.

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

## 6. Análisis de rentabilidad por canal (medidas nuevas)

Estas cinco medidas se calcularon y comprobaron en la **versión local de Power BI** durante la revisión del proyecto. El archivo `.pbix` publicado y las capturas del repositorio se actualizarán cuando finalicen los cambios del dashboard.

**Criterios de negocio:** solo se consideran pedidos con estado `Entregado`. El costo por operación se calcula con `TblFactVentas[CostoUnitario]`, multiplicado por `Cantidad`. El ingreso corresponde al precio unitario con el descuento aplicado. La comisión se calcula sobre ese ingreso después del descuento y toma el porcentaje correspondiente a cada canal mediante la relación del modelo (`RELATED`). El margen después de comisiones **no equivale a la utilidad neta**, porque no descuenta otros gastos, impuestos ni costos no representados en el dataset.

### Costo Productos Vendidos

Suma el costo de los artículos de pedidos entregados, según la cantidad y el costo unitario de cada transacción.

```dax
Costo Productos Vendidos =
CALCULATE(
    SUMX(
        TblFactVentas,
        TblFactVentas[Cantidad] *
        TblFactVentas[CostoUnitario]
    ),
    TblFactVentas[EstadoPedido] = "Entregado"
)
```

### Margen Bruto

Calcula el margen monetario después del costo de los productos y antes de las comisiones.

```dax
Margen Bruto =
[Ventas_Totales] - [Costo Productos Vendidos]
```

### Comisiones Ventas

Calcula las comisiones correspondientes a ventas entregadas, aplicando el porcentaje asociado a cada canal al importe después del descuento.

```dax
Comisiones Ventas =
CALCULATE(
    SUMX(
        TblFactVentas,
        TblFactVentas[PrecioUnitario] *
        TblFactVentas[Cantidad] *
        (1 - TblFactVentas[Descuento]) *
        RELATED(TblDimCanal[ComisionPct])
    ),
    TblFactVentas[EstadoPedido] = "Entregado"
)
```

**Porcentajes observados en la dimensión de canales:** Online 3,5 %, Tienda 2 % y Distribuidor 5 %.

### Margen Despues de Comisiones

Calcula cuánto queda de las ventas después de descontar costos de productos y comisiones comerciales, sin contemplar otros gastos.

```dax
Margen Despues de Comisiones =
[Margen Bruto] - [Comisiones Ventas]
```

### Margen Despues de Comisiones %

Expresa el margen después de comisiones como proporción de la facturación de pedidos entregados en el contexto de filtros.

```dax
Margen Despues de Comisiones % =
DIVIDE(
    [Margen Despues de Comisiones],
    [Ventas_Totales],
    BLANK()
)
```

### Resultados comprobados: año 2024, sin filtros

| Canal | Ventas entregadas | Margen bruto | Comisiones | Margen después de comisiones | Margen % |
|---|---:|---:|---:|---:|---:|
| Online | $475.877,55 | $150.484,42 | $16.655,71 | $133.828,71 | 28,12 % |
| Distribuidor | $386.256,61 | $125.182,67 | $19.312,83 | $105.869,84 | 27,41 % |
| Tienda | $381.974,34 | $109.161,22 | $7.639,49 | $101.521,74 | 26,58 % |
| **Total** | **$1.244.108,50** | **$384.828,31** | **$43.608,03** | **$341.220,28** | **27,43 %** |

**Interpretación:** Online presenta la mayor facturación y el mayor margen después de comisiones, tanto absoluto como porcentual, entre los tres canales. Se trata de un **margen comercial parcial**, no de un análisis completo de rentabilidad neta. Las diferencias de porcentaje deben interpretarse con cautela y no atribuirse a una causa sin un análisis adicional.

---

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

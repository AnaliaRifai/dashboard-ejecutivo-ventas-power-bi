# Análisis comparativo de períodos | Ventas 2024

## Objetivo y alcance

Profundizar el hallazgo de **seis meses consecutivos con cumplimiento de metas (mayo–octubre)** comparando la composición de ventas por canal, producto y cliente frente al **resto del año (enero–abril y noviembre–diciembre)**.

- **Origen:** matrices exploratorias realizadas en Power BI con datos simulados de una empresa ficticia del sector tecnológico.
- **Métrica:** `[Ventas_Totales]`, que considera exclusivamente pedidos con estado `Entregado`, con descuentos aplicados.
- **Períodos:** dos grupos de seis meses cada uno, clasificados mediante la columna calculada `Periodo Comparacion` de `Tabla Calendario`.
- **Alcance:** análisis descriptivo de facturación y participación; los contrastes no prueban causalidad ni explican por sí solos el cumplimiento de metas.

## Clasificación de períodos (DAX)

La siguiente **columna calculada** permite separar las fechas del calendario en los dos grupos:

```dax
Periodo Comparacion =
IF(
    'Tabla Calendario'[NúmeroMes] >= 5 &&
    'Tabla Calendario'[NúmeroMes] <= 10,
    "Mayo - Octubre",
    "Resto del año"
)
```

Las matrices exploratorias utilizaron:
- **Filas:** alternativamente `TblDimCanal[Canal]`, `TblDimProducto[Producto]` y `TblDimCliente[Cliente]`.
- **Columnas:** `Tabla Calendario[Periodo Comparacion]`.
- **Valores:** `[Ventas_Totales]`.

## 1. Volumen de ventas

| Indicador | Mayo–octubre | Resto del año |
|---|---:|---:|
| Ventas de pedidos entregados | $746.602,60 | $497.505,89 |

**Cálculo del cambio relativo:**

```text
((746.602,60 - 497.505,89) / 497.505,89) × 100 = 50,07 %
```

**Lectura:** mayo–octubre registró un **50,07 % más de facturación** que el resto de los meses de 2024 agrupados. La comparación usa un período de seis meses como referencia; no es una tasa de crecimiento mes a mes.

## 2. Composición por canal

| Canal | Mayo–octubre | Resto del año | Total 2024 |
|---|---:|---:|---:|
| Online | $319.187,32 | $156.690,23 | $475.877,55 |
| Distribuidor | $210.886,28 | $175.370,33 | $386.256,61 |
| Tienda | $216.529,01 | $165.445,34 | $381.974,34 |
| **Total** | **$746.602,60** | **$497.505,89** | **$1.244.108,50** |

**Participación de Online sobre la facturación de cada período:**

```text
Mayo–octubre: 319.187,32 / 746.602,60 × 100 = 42,75 %
Resto del año: 156.690,23 / 497.505,89 × 100 = 31,50 %
Diferencia sin redondear: 42,75197 % - 31,49515 % = 11,26 puntos porcentuales
```

**Lectura:** la participación de Online fue **11,26 puntos porcentuales mayor** en mayo–octubre. Se trata de un cambio de participación, no de un crecimiento del 11,26 % en sus ventas.

## 3. Composición por producto

| Producto | Mayo–octubre | Resto del año | Total 2024 |
|---|---:|---:|---:|
| Laptop Air 13 | $231.335,99 | $59.257,59 | $290.593,58 |
| Laptop Pro 14 | $193.759,73 | $44.538,45 | $238.298,18 |
| **Ambas laptops** | **$425.095,72** | **$103.796,04** | **$528.891,76** |

**Participación conjunta de las dos laptops:**

```text
Mayo–octubre: (231.335,99 + 193.759,73) / 746.602,60 × 100 = 56,94 %
Resto del año: (59.257,59 + 44.538,45) / 497.505,89 × 100 = 20,86 %
Diferencia sin redondear: 56,93735 % - 20,86328 % = 36,07 puntos porcentuales
```

**Lectura:** las dos laptops concentraron **56,94 %** de las ventas de mayo–octubre, frente a **20,86 %** en el resto del año. Fue la diferencia de composición más marcada entre los productos revisados.

Otros productos presentaron mayor facturación fuera de mayo–octubre, por ejemplo **Tablet 11** ($50.181,63 frente a $105.597,25) y **Silla Ergonómica** ($36.729,70 frente a $71.555,41).

## 4. Clientes: ejemplos de variación en la composición

| Cliente | Mayo–octubre | Resto del año |
|---|---:|---:|
| Beta Market | $81.113,50 | $46.029,48 |
| Omega Tech | $75.350,48 | $9.940,40 |
| Logística Prime | $61.400,74 | $4.122,36 |
| Grupo Nova | $8.698,86 | $81.511,35 |

Estos clientes ilustran perfiles distintos de compra por período. Son **diferencias observadas en facturación**, no evidencia suficiente para determinar por qué compraron más o menos.

## Conclusión y límites

Los seis meses consecutivos de cumplimiento de metas coincidieron con **mayor facturación**, **mayor participación del canal Online** y una **fuerte concentración de ingresos en Laptop Air 13 y Laptop Pro 14**. Los clientes también presentaron distribuciones de compra diferentes.

Este análisis ayuda a formular hipótesis comerciales, pero **no demuestra que los canales, productos o clientes mencionados causaran el cumplimiento de las metas**. Para investigar causas haría falta incorporar información de demanda, promociones, precios, campañas, stock y otros factores, además de las metas correspondientes.

**Presentación en el portfolio:** las tres matrices se utilizaron para explorar los datos y respaldar este documento. No se incluyen como una página adicional del dashboard final, para mantener una experiencia de lectura clara.

# 📈 Análisis de ventas y cumplimiento de metas | Power BI — 2024

Proyecto de análisis comercial desarrollado en **Power BI** a partir de datos simulados de una empresa ficticia del sector tecnológico. El objetivo es evaluar el desempeño de ventas de 2024, analizar el cumplimiento de metas e identificar oportunidades de mejora mediante indicadores y visualizaciones interactivas.

## 🎯 Hallazgo principal

> **El criterio de cálculo cambió la conclusión del negocio:** el cumplimiento anual de metas de 2024 pasó de **117,77 %** (cálculo inicial) a **95,44 %** al considerar únicamente las ventas de pedidos con estado **`Entregado`**. Por lo tanto, **la meta anual no se alcanzó**, aunque entre mayo y octubre hubo seis meses consecutivos de cumplimiento.

Este resultado demuestra por qué es fundamental validar qué operaciones incluye cada indicador, además de comprobar que la fórmula DAX funcione correctamente.

## ⚙️ Herramientas utilizadas

- **Excel:** fuente de datos del proyecto.
- **Power Query:** limpieza, transformación y validación.
- **Power BI:** modelo de datos, visualizaciones e interactividad.
- **DAX:** indicadores de ventas, inteligencia temporal, rankings y cumplimiento de metas.

## ❔ Objetivo y preguntas de negocio

El objetivo fue convertir los datos de ventas en información útil para el seguimiento comercial y responder:

1. ¿Se alcanzaron las metas de ventas?
2. ¿Cómo evolucionaron las ventas durante 2024?
3. ¿Qué productos generaron mayor facturación?
4. ¿Qué clientes tuvieron mayor participación?
5. ¿Qué canal aportó más ventas?

## 🗂️ Estructura del repositorio

```text
dashboard-ejecutivo-ventas-power-bi/
├── dataset/
│   └── datos_ventas_tecnologia_2024.xlsx
├── screenshots/
│   ├── dashboard-general.png
│   ├── dashboard-rentabilidad.png
│   ├── dashboard-filtro-mes.png
│   ├── dashboard-filtro-canal.png
│   ├── modelo-datos.png
│   └── power-query-limpieza.png
├── Dashboard-Ejecutivo-Ventas.pbix
├── medidas_dax_documentacion.md
├── analisis_comparativo_periodos_2024.md
└── README.md
```

La carpeta `dataset` contiene los datos fuente del proyecto y el archivo `.pbix` incluye el modelo de datos y el informe interactivo de **dos páginas**: **Resumen ejecutivo** y **Análisis de rentabilidad**. Las capturas muestran ambas páginas, dos ejemplos de filtrado, el modelo de datos y el proceso de limpieza en Power Query.

## 📥 Cómo abrir y actualizar el proyecto

1. Descargar [el archivo de Power BI](Dashboard-Ejecutivo-Ventas.pbix) y [el Excel de origen](dataset/datos_ventas_tecnologia_2024.xlsx) desde este repositorio.
2. Abrir el archivo `.pbix` con **Power BI Desktop**. El informe contiene datos cargados y puede explorarse sin actualizar el origen.
3. Para **actualizar los datos**, ir a **Archivo → Opciones y configuración → Configuración de origen de datos**.
4. En **Orígenes de datos en el archivo actual**, seleccionar la conexión de Excel y hacer clic en **Cambiar origen...**. Indicar la ubicación donde se descargó `datos_ventas_tecnologia_2024.xlsx` y confirmar.
5. Ejecutar **Actualizar** en Power BI Desktop para volver a cargar los datos desde ese archivo.

**Nota de reproducibilidad:** el `.pbix` se desarrolló con un Excel local llamado `proyecto_powerbi_avanzado_dirty.xlsx`. En GitHub, ese mismo archivo fuente está publicado como `datos_ventas_tecnologia_2024.xlsx`. La conexión almacenada en Power BI hace referencia a una ruta local de la computadora donde se desarrolló el informe; por eso, **cada persona que desee actualizar los datos debe configurar el origen con su propia ruta**. No es necesario renombrar el Excel descargado.


## ⏳ Exploración y preparación de los datos (EDA y ETL)

Se partió de cinco tablas: **ventas, productos, clientes, canales y metas comerciales**.

En **Power Query** se realizaron las principales transformaciones:

- **Ventas:** exclusión de dos registros sin fecha de venta; normalización de identificadores, textos, precios y costos; reemplazo de descuentos nulos por cero. Se conservaron las cantidades negativas asociadas a devoluciones. La tabla final contiene **578 registros**.
- **Productos:** estandarización de nombres y categorías, corrección de tipos numéricos y eliminación de un identificador duplicado. La dimensión quedó con **12 productos únicos**.
- **Clientes:** normalización de países, segmentos y correos electrónicos. Se conservaron los **20 clientes**, incluido un correo faltante que no afectaba el análisis.
- **Canales:** limpieza de textos y corrección de comisiones para **Online, Tienda y Distribuidor**.
- **Metas:** verificación de **36 registros**, correspondientes a los 12 meses de 2024 para los tres canales.

Se revisó la calidad de las columnas relevantes y los campos necesarios para las relaciones y los indicadores.
### Limpieza y transformación en Power Query

![Transformación y calidad de datos en Power Query](screenshots/power-query-limpieza.png)

### Criterio comercial utilizado

Las medidas principales de facturación, unidades vendidas y transacciones contabilizan **únicamente los pedidos con estado `Entregado`**. Los registros pendientes, cancelados y devueltos permanecen en el dataset, pero no se incluyen en estos KPI.

Esta definición resultó fundamental: al corregir el indicador, el cumplimiento anual de metas pasó del **117,77 %** al **95,44 %**. El cambio muestra por qué es necesario validar el significado de cada métrica, además de comprobar que su fórmula funcione.

## ⭐ Modelo de datos

Se construyó un **modelo en estrella** con `TblFactVentas` como tabla central, relacionada mediante vínculos activos de uno a muchos y filtrado unidireccional con las dimensiones:

- `TblDimCliente`
- `TblDimProducto`
- `TblDimCanal`
- `Tabla Calendario`

La tabla calendario se creó con DAX, contiene las **366 fechas de 2024** y fue configurada como tabla de fechas.

`TblMetasVentas` se mantuvo sin relaciones físicas: la medida de metas utiliza **`TREATAS`** para aplicar los filtros de año, mes y canal. Las métricas se organizaron en una tabla independiente denominada `Medidas`.

![Modelo de datos](screenshots/modelo-datos.png)

## 📐 Medidas DAX

Se documentaron **22 medidas DAX**: las 17 del análisis original y cinco medidas adicionales de rentabilidad, incluidas en el archivo `.pbix` actualizado.

Entre las funciones utilizadas se encuentran `CALCULATE`, `SUMX`, `DIVIDE`, `DISTINCTCOUNT`, `DATEADD`, `TOTALYTD`, `RANKX`, `REMOVEFILTERS`, `TREATAS`, `TOPN` y `SWITCH`.

Las fórmulas finales y sus explicaciones están en **[Documentación de medidas DAX](medidas_dax_documentacion.md)**.

## 🖥️ Dashboard e interactividad

La página **Resumen ejecutivo** permite explorar los resultados mediante cinco segmentadores: **Mes, País, Canal, Categoría y Segmento**. Incluye tarjetas de ventas, meta, cumplimiento, variación y ticket promedio; gráficos de evolución mensual, ventas frente a metas y desempeño por cliente, producto y canal; un ranking de clientes y un texto de insight generado con DAX. La segunda página, **Análisis de rentabilidad**, compara márgenes y comisiones por canal. Ambas páginas están conectadas mediante botones de navegación.

La tarjeta **VARIACIÓN %** utiliza una medida de presentación adicional (`Variacion % Tarjeta`): en la vista general indica **«Seleccioná un mes»**, muestra el porcentaje cuando existe una comparación mensual válida y señala **«Sin período anterior»** cuando no hay base de comparación. La medida numérica original se conserva para los cálculos.

Se personalizaron las interacciones para preservar el contexto de comparación:

- Al seleccionar un **mes**, los indicadores y rankings se actualizan, mientras que los dos gráficos temporales mantienen la vista de todo 2024.
- Al seleccionar un **canal**, los indicadores y el resto del informe se actualizan, pero el gráfico de ventas por canal conserva la comparación entre los tres canales.


### Dashboard general

![Dashboard general](screenshots/dashboard-general.png)

### Filtro por mes: septiembre de 2024

![Dashboard filtrado por septiembre](screenshots/dashboard-filtro-mes.png)

### Filtro por canal: Online

![Dashboard filtrado por canal Online](screenshots/dashboard-filtro-canal.png)
  

## 💰 Rentabilidad por canal

Se incorporaron al informe medidas de **costo de productos vendidos, margen bruto, comisiones, margen después de comisiones y margen porcentual**. Bajo el criterio de pedidos entregados, el margen después de costos de productos y comisiones fue de **$341.220,28 (27,43 % de las ventas)**.

En la comparación por canal, **Online** presentó el mayor margen monetario (**$133.828,71**) y porcentual (**28,12 %**), seguido de Distribuidor (**27,41 %**) y Tienda (**26,58 %**). El cálculo descuenta costos de productos y comisiones, pero **no representa utilidad neta**. Los cálculos y resultados están documentados en [medidas DAX](medidas_dax_documentacion.md).

### Página: Análisis de rentabilidad

![Dashboard de análisis de rentabilidad](screenshots/dashboard-rentabilidad.png)

## 💎 Hallazgos comerciales de 2024

### 1. La meta anual no se alcanzó, aunque hubo seis meses consecutivos de cumplimiento

| Indicador | Resultado |
|---|---:|
| Ventas de pedidos entregados | $1.244.108,50 |
| Meta anual | $1.303.553 |
| Cumplimiento | **95,44 %** |
| Diferencia frente a la meta | **-$59.444,50** |

El análisis mensual permitió identificar **tres períodos**:

- **Enero–abril:** ningún mes alcanzó la meta.
- **Mayo–octubre:** seis meses consecutivos con ventas superiores a las metas mensuales. **Junio** registró el mayor importe de ventas (**$144.524,31**) y el mayor cumplimiento (**130,31 %**).
- **Noviembre–diciembre:** los resultados volvieron a quedar por debajo de sus objetivos.

El buen desempeño de mayo a octubre no fue suficiente para compensar los desvíos negativos del resto del año. Este patrón no se observa al mirar únicamente el KPI anual.

**Profundización del hallazgo:** al comparar mayo–octubre con los otros seis meses de 2024, la facturación fue **50,07 % mayor**. Las ventas de **Laptop Air 13 y Laptop Pro 14** representaron **56,94 %** del período de cumplimiento, frente a **20,86 %** en el resto del año. A su vez, la participación del canal **Online** pasó de **31,50 %** a **42,75 %** (**+11,26 puntos porcentuales**). También se observaron diferencias en la distribución de compras por cliente.

Estas cifras indican un **cambio en la composición de ventas**, pero **no prueban que esos factores causaran el cumplimiento de metas**. Las matrices exploratorias, fórmulas de porcentaje, datos por cliente y criterios de comparación están documentados en [Análisis comparativo de períodos 2024](analisis_comparativo_periodos_2024.md).

### 2. Dos laptops concentraron una parte importante de la facturación

**Laptop Air 13** fue el producto líder, con **$290.593,58**, seguido de **Laptop Pro 14**, con **$238.298,18**. Juntas representaron aproximadamente **42,5 %** de las ventas anuales.

**Recomendación:** analizar su desempeño por período y canal y complementar la facturación con información de márgenes antes de definir acciones comerciales.

### 3. Beta Market fue el cliente principal

**Beta Market** generó **$127.142,98**, equivalentes al **10,22 %** de las ventas anuales. Le siguieron **Grupo Nova** y **Omega Tech**.

**Recomendación:** hacer seguimiento de los principales clientes y explorar oportunidades en el resto de la cartera, sin interpretar la concentración de ventas como un riesgo demostrado por sí solo.

### 4. Online fue el canal con mayor facturación

El canal **Online** aportó aproximadamente **$475.878** (**38,3 %** del total), por delante de Distribuidor y Tienda.

**Recomendación:** estudiar el potencial de crecimiento del canal junto con sus costos y comisiones. Mayor facturación no implica necesariamente mayor rentabilidad.

## ✅ Conclusión

El proyecto permitió recorrer un proceso de análisis de punta a punta: preparar datos con inconsistencias, construir un modelo de datos, definir indicadores de negocio, desarrollar un dashboard interactivo y convertir los resultados en hallazgos y recomendaciones.

El aprendizaje más importante fue la necesidad de **validar los criterios comerciales detrás de las medidas**: una fórmula puede funcionar correctamente y, aun así, conducir a conclusiones equivocadas si incluye operaciones que no corresponden al indicador analizado.

## Análisis de Ventas Trimestrales en Power BI

Este proyecto presenta un análisis de las ventas trimestrales de una base de datos para los meses de octubre,noviembre y diciembre mediante el sofware Power BI. Se exploran tendencias de ventas, segmentación por productos y clientes, se muestran graficos e indicadores claves  que se destacan en el informe.

## Fuentes de Datos y Recolección

Los datos se obtuvieron de un archivo excel de un curso de Power Bi de cousera."conjuntos de datos para practicar,sin resolver"

*Los datos se exportaron de un archivo XLSX y se integraron en Power BI mediante Power Query.*

## Objetivos

- **Analizar** la evolución de las ventas por mes.
- **Identificar** productos y clientes que aportan mayor y menor ingreso.
- **Evaluar** los indicadores claves de las ventas,productos y clientes.

**Motivacion** identificar informacion relevante para la empresa,lo cual se pueda analizar tendencias de ventas,preferencias de los clientes,segmentacion de productos.

**Metodología y Proceso de Trabajo**
- Cuando se conecta la base de datos a Power Bi,en la previsualización se observa que hay tablas con diferentes nombres,pero con la misma información de otras tablas,por algún error en la creación de su estructura.Se eliminaron las tablas duplicadas y nos quedamos solo con las necesarias para trabajar.
- transformamos los datos en Power Query.En la tabla clientes,eliminamos columnas que no se relacionan con el cliente,como atributos del producto,información que debería ir en la tabla de ventas.Combinamos el nombre del cliente,que esta en diferentes columnas su nombre y apellido,las únimos para crear el nombre completo por cliente,con el objetivo de reducir información y generar consultas eficientes.
- Cambiamos los tipos de datos como el id_cliente en formato texto,pues de tipo númerico no nos interesa ya que no es para realizar cálculos.La columna feha tipo texto,la cambiamos a tipo fecha y creamos una columna nueva que nos extraiga,solo el mes respectivo de compra para posteriores análisis por mes.
- La clave principal de las tablas en este caso cliente id,como identificador único,vemos que hay dúplicados,lo cual no debería existir y generamos la columna de valores únicos.
- En este caso no hay valores faltantes en las filas,por lo que no es necesario ningúna limpieza de este tipo.
- Juntamos las tablas de ventas separadas de octubre,noviembre y diciembre en una,con el fin de facilitar los análisis y mejorar el rendimiento del modelo.
  
**Modelos relacionales**
- Después de eliminar las tablas duplicada nos quedamos con cuatro tablas para trabajar:
- 
   -La tabla de clientes con su columna de  identificador único llamada customers ID,para conectar con la tabla de ventas de los tres meses.En la tabla sales oct,nov,dic;creamos una columna llamada ID_Venta de valores Únicos,con su identificación única y combianciones únicas de  su categoria y estado donde se adquirió el producto,para poder relacionar diferentes  atributos de la tabla sales oct,nov y dic y la tabla Total Q4 Regional Sales,con la tabla clientes"Customers".
  
   -Construimos una tabla puente,con una columna Única y de valores únicos llamada category_state,creada con la combinación de las columnas  de categoría del producto y su estado geográfico donde se realizó la compra,con el fin de generar valores Únicos y conectarlas.

    ![modelo relacionar,Power Bi,ventas](https://github.com/luismontes031/my-portfolio/blob/main/PrimerProyecto/modelo%20relacional,ventas.png?raw=true)

  **Visualizaciones Clave**
  
  Se creó un gráfico de barras agrupadas que visualiza las diferentes categorías de productos junto con el estado donde se realizó la compra. Dado que una misma categoría de producto puede adquirirse en distintos estados, este gráfico permite visualizar filas únicas según su categoría y estado.

Se generó un gráfico de barras agrupadas que muestra las tres mayores compras de productos realizadas por los clientes. En este caso, se observa que el primer lugar lo ocupa la compra de ropa en el estado de Alabama, el segundo lugar corresponde a productos de librería también en Alabama, y en tercer lugar se encuentran los productos electrónicos en el estado de Carolina del Norte, destacando así el TOP 3 de compras.

Se puede visualizar una tabla que clasifica el total de ventas por cliente y su nombre completo.

Se presentan diferentes tarjetas con indicadores clave, tales como: el producto más vendido, el producto con mayor crecimiento en ventas durante los últimos tres meses, el producto con menor crecimiento en ventas en el mismo período, y el producto junto con el nombre del cliente que más gastó entre todos.

Se observa un gráfico de columnas agrupadas que clasifica, por mes, el total de las ventas realizadas.

Junto al gráfico de columnas agrupadas, se encuentran otras tarjetas que indican el porcentaje de crecimiento o decrecimiento por producto y su categoría de un mes a otro, así como el total acumulado de los meses.

## Código y Detalles Técnicos**
###**Medidas DAX:**

**`cliente_MayorGasto`** *"calcula qué cliente gastó más dinero en sus compras"*
```DAX
Cliente_MayorGasto = 
VAR TablaClientes =
    ADDCOLUMNS(
        'Customers',
        "TotalCompras", [TotalVentasPorCliente]
    )
VAR ClienteTop =
    TOPN(1, Customers, [TotalVentasPorCliente], DESC)
RETURN
    CONCATENATEX(ClienteTop, 'Customers'[fullname], ", ")

**`cliente_MenorGasto`** *"calcula que cliente gasto menos dinero en sus compras"*
```DAX
Cliente_MenorGasto = 
VAR TablaClientes =
    ADDCOLUMNS(
        'Customers',
        "TotalCompras", [TotalVentasPorCliente]
    )
VAR ClienteBottom =
    TOPN(1, Customers, [TotalVentasPorCliente], ASC)
RETURN
    CONCATENATEX(ClienteBottom, 'Customers'[fullname], ", ")

**`crecimiento_Dic`** *"crecimiento de ventas en porcentaje,el mes diciembre"*
```DAX
Crecimiento_Dic = 
IF(
    ISBLANK([total ventas_noviembre]), 
    BLANK(), 
    ([total ventas_december] - [total ventas_noviembre]) / [total ventas_noviembre]
)
**`crecimiento_Nov`** *"crecimiento de ventas en porcentaje,el mes Noviembre"*
```DAX
Crecimiento_Nov = 
IF(
    ISBLANK([total ventas_october]), 
    BLANK(), 
    ([total ventas_noviembre] - [total ventas_october]) / [total ventas_october]
)

**`crecimiento_total_Oct_Dic`** *"crecimiento de ventas en porcentaje,durante los tres meses"*
```DAX

Crecimiento_Total_Oct_Dic = 
    VAR Crec_Oct_Nov = [Crecimiento_Nov]
    VAR Crec_Nov_Dic = [Crecimiento_Dic]
    RETURN 
        Crec_Oct_Nov + Crec_Nov_Dic

**`Producto_Mas_Vendido`** *"producto más vendido según su Categoria y Estado donde se compró"*
```DAX
Producto_Mas_Vendido = 
VAR Producto_Max = 
    TOPN(1, 'Total Q4 Regional Sales', 
        'Total Q4 Regional Sales'[Units Sold (October)] + 
        'Total Q4 Regional Sales'[Units Sold (November)] + 
        'Total Q4 Regional Sales'[Units Sold (December)], DESC
    )
RETURN
    CONCATENATEX(Producto_Max, 'Total Q4 Regional Sales'[Product Category], ", ")

**`Producto_MasComprado_TopCliente`** *"producto mas comprado por cliente"*
```DAX
Producto_MasComprado_TopCliente = 
VAR TopClienteTabla =
    CALCULATETABLE(
        TOPN(1, 'Customers', [TotalVentasPorCliente], DESC),
        ALL('Customers')
    )
VAR TopClienteID =
    MAXX(TopClienteTabla, 'Customers'[Customer ID])
VAR TablaProductos =
    SUMMARIZE(
        FILTER(
            'sales oct,nov,dic',
            'sales oct,nov,dic'[Customer ID] = TopClienteID
        ),
        'sales oct,nov,dic'[Product Category],
        "TotalProducto",
            CALCULATE(
                SUM('Total Q4 Regional Sales'[Sales Total]),
                TREATAS(
                    VALUES('sales oct,nov,dic'[Category_state]),
                    'Total Q4 Regional Sales'[Category_state]
                )
            )
    )
VAR ProductoTop =
    TOPN(1, TablaProductos, [TotalProducto], DESC)
RETURN
    CONCATENATEX(ProductoTop, 'sales oct,nov,dic'[Product Category], ", ")

**`Producto_Mayor_crecimiento`** *"producto con mayor crecimiento de compra durante los tres meses"*
DAX```
Producto_Mayor_Crecimiento = 
    SELECTCOLUMNS(
        TOPN(1, 
            VALUES('Total Q4 Regional Sales'[Category_State]), 
            [Crecimiento_Total_Oct_Dic], 
            DESC
        ), 
        "Producto", 'Total Q4 Regional Sales'[Category_State]
    )

**`Producto_Mayor_decrecimiento`** *"producto con mayor decrecimiento de compra durante los tres meses"*
DAX```
Producto_Mayor_Decrecimiento = 
    SELECTCOLUMNS(
        TOPN(1, 
            VALUES('Total Q4 Regional Sales'[Category_State]), 
            [Crecimiento_Total_Oct_Dic], 
            ASC
        ), 
        "Producto", 'Total Q4 Regional Sales'[Category_State]
    )


**`Total_Ventas_totales`** *"total de ventas por los tres meses"*
DAX```
Ventas_Totales_Oct_Dic = 
    [total ventas_october]+ [total ventas_noviembre] + [total ventas_december]

**`Totalventasporcliente`** *"compras totales por cada cliente"*
DAX```
TotalVentasPorCliente = 
CALCULATE(
    SUM('Total Q4 Regional Sales'[Sales Total]),
    TREATAS(
        VALUES('sales oct,nov,dic'[Category_state]),
        'Total Q4 Regional Sales'[Category_state]
    )
)

## Conclusiones generales:
-En nuestro informe final,podemos visualizar una gráfica de barras agrupadas llamada ventas totales por Categoria y estado,lo cual cada vez que seleccionamos una barra según la categoria y estado,nos muestra en las tarjetas,la variación del porcentaje de crecimiento y decrecimiento de las ventas totales por cada categoria y estado,tanto de noviembre a diciembre como los tres meses juntos.

-El producto con mayor crecimiento de ventas durante el trimestre fue para Pets -LA "producto para mascotas en el Estado de Louisiana, EE. UU"
-El producto con menor crecimiento de ventas durante el trimestre fue para Books - SC "producto libros en el Estado de South Carolina (Carolina del Sur, EE. UU.)
-El producto más comprado por cliente fué para producto ropa (cloting),el cliente que más gastó en este producto fue Shepherd Charli.
-EL producto más vendido de todos fué para productos electronicos(Electrónics),sin embargo el que más produjo ganancias fué para cloting.
-Durante los tres meses hubo un crecimiento de ventas totales de 1,39%,entre Septiembre y Noviembre crecierón 0,95%,entre Noviembre y Diciembre crecieron 0,44%.

## Conclusiones detalladas:

-Los productos para mascotas fuerón los más demandados durante los tres meses,cada vez las personas invierten más para sus mascotas,esto está asociado a que las familias cada vez adoptan y prefieren convivir y tener de familia a un animal que tener hijos.
-Los productos menos demandados fuerón los libros,podría relacionarse con la preferencia de los clientes de adquirir y leer libros electrónicos o descargarlos gratis en internet,en caso que se traté de libros físicos o que los clientes prefierán pasar su tiempo menos en lectura y más en otros tipos de entretenimietos como las redes.
-El producto cloting,fué el que generó mas ganancias de todos, los bajos costos de producción y la preferencias de clientes por la vestimenta,inciden en mayor rentabilidad del negocio.
-Aunque el producto Pets,fue el que más creció en porcentaje en número de ventas en ese tiempo,el producto electronics fué el más vendido,pero no obtuvo las mayores ganancias y puede estar asociado a los costos de producción y materias primas que se utilizan para su producción que pueden variar de precios según el contexto de relaciones y precios comerciales.
-Creció más las ventas de Septiembre a Noviembre que de Noviembre a Diciembre,se espera que la gente prefiere guardar su dinero para Diciembre,el crecimiento de ventas de Diciembre fue mayor en valor absoluto,claramente por la época.

## Propuesta como analista: El mercado de mascotas cada vez crece más,tiene oportunidades futuras de negocio.El mercado de libros en el Estado de Carolina del Sur,fué donde más decrecierón sus ventas,sin embargo no es general porque en el Estado de Alabama,fué el segundo producto que más genero ventas,es importante segmentar por Estado las familias que prefieren las lecturas o existan mayores centros educativos que aumenten la demanda de libros,para oportunidades de negocios.El producto más vendido durante el trimestre fueron para productos electrónicos,sin embargo este no genero mayores ganancias;su relación esta influenciada por los costos de producción,transporte y logística,variación de precios de materias de la cadena productiva y las fluctuaciones en relacion con las exportaciones e importaciones.

## Tecnologías Utilizadas

- Power BI Desktop, Power Query, DAX.
- Git y GitHub para control de versiones y documentación.
- Visual Studio Code para la edición de este README y otros archivos de documentación.

## Contacto

Proyecto desarrollado por [Luis Sánchez](https://github.com/luismontes031). 
Para consultas, contáctame a [lsanchez9135@hotmail.com](mailto:lsanchez9135@hotmail.com).

  
  
  


<div align="center">
  <h1>Ingeniería de datos con SSIS <b><i><a href="https://www.ibm.com/docs/es/ida/9.1.2?topic=schemas-snowflake" target="_blank">[Modelo Copo de Nieve]</a></i></b></h1>
</div>

<div align="center"> 
  <img src="readme_img/ssis-ingenieria_de_datos.png" width="">
</div>

# Introducción al documento

El contenido de este documento son **apuntes teoricos y prácticos** y un proyecto **ETL** de una tienda llamada **"SUPERMAYORISTA"** y busca ser una guía para futuros trabajos personales. El mismo está hecho por mi persona [José Eduardo Galván Salvador](https://www.linkedin.com/in/eduardo-galvan1208/) para el grupo de estudio [Data Growth Community](https://www.linkedin.com/company/datagrowthcommunity/).

# Herramientas 

- **SQL Server 2017**
- **SSDT 2017 - SQL Server Data Tools**
- **Visual Studio 2017**

# Objetivos del documento

- Generar información precisa y oportuna para los usuarios finales de formal y que sea representativa para su uso en el proceso de toma decisiones.
- Analizar los datos mediante el uso de herramientas tales como PowerBi que permitirá el desarrollo de un Dashboard donde estos datos serán presentados para los usuarios finales.
- Trabajar con Power Query.
- Hacer preprocesamiento de datos para crear dashboards. 
- Extraer información de la base de datos de la empresa.

## Tabla de contenido
- [ETL "Supermayorista"](#ETL-Supermayorista)
  - [FLUJO_DIM_DOCUMENTO](#FLUJO_DIM_DOCUMENTO)
  - [FLUJO_DIM_VENDEDOR](#FLUJO_DIM_VENDEDOR)
  - [FLUJO_DIM_VENTA](#FLUJO_DIM_VENTA)
  - [FLUJO_DIM_BANCO](#FLUJO_DIM_BANCO)
  - [FLUJO_DIM_PERIODO](#FLUJO_DIM_PERIODO)
  - [FLUJO_DIM_CATEGORIA](#FLUJO_DIM_CATEGORIA)
  - [FLUJO_DIM_SUBCATEGORIA](#FLUJO_DIM_SUBCATEGORIA)
  - [FLUJO_DIM_PRODUCTO](#FLUJO_DIM_PRODUCTO)
  - [FLUJO_DIM_DEPARTAMENTO](#FLUJO_DIM_DEPARTAMENTO)
  - [FLUJO_DIM_DISTRITO](#FLUJO_DIM_DISTRITO)
  - [FLUJO_DIM_CLIENTE](#FLUJO_DIM_CLIENTE)
  - [CORRECCION_ORTOGRAFICA](#CORRECCION_ORTOGRAFICA)
  - [FLUJO_FACT_SUPERMAYORISTA](#FLUJO_FACT_SUPERMAYORISTA)
  - [BUCLE_TXT_VENTA](#BUCLE_TXT_VENTA)
    - [FLUJO_TRANSACCIONES_TXT_VENTAS_&_PERIODO](#FLUJO_TRANSACCIONES_TXT_VENTAS_&_PERIODO)
    - [FLUJO_TRANSACCIONES_TXT_FACT](#FLUJO_TRANSACCIONES_TXT_FACT)
- [Planificación del proyecto](#Planificación-del-proyecto)
  - [Alcance del negocio](#Alcance-del-negocio)
- [Definición de requerimientos para la empresa SUPERMAYORISTA](#Definición-de-requerimientos-para-la-empresa-SUPERMAYORISTA)
  - [Requisitos del negocio](#Requisitos-del-negocio)
  - [Base de Datos transaccional en SQL Server](#Base-de-Datos-transaccional-en-SQL-Server)
  - [Diccionario de datos del origen a nivel general de la base de datos transaccional en SQL Server 2017](#Diccionario-de-datos-del-origen-a-nivel-general-de-la-base-de-datos-transaccional-en-SQL-Server-2017)
    - [Tablas](#Tablas)
    - [Diccionario de datos de la base de datos transaccional en SQL Server 2017](#Diccionario-de-datos-de-la-base-de-datos-transaccional-en-SQL-Server-2017)
      - [VENTA](#VENTA)
      - [DETALLE_VENTA](#DETALLE_VENTA)
      - [PRODUCTO](#PRODUCTO)
      - [CLIENTE](#CLIENTE)
      - [VENDEDOR](#VENDEDOR)
      - [TIPO_DOCUMENTO](#TIPO_DOCUMENTO)
- [Modelo Dimensional](#Modelo-Dimensional)
  - [Elección de dimensiones](#Elección-de-dimensiones)
- [Dimensiones encontradas](#Dimensiones-encontradas)
- [Diseño lógico](#Diseño-lógico)
- [Diseño físico](#Diseño-físico)
  - [Tablas del modelo dimensional](#Tablas-del-modelo-dimensional)

# ETL **_Supermayorista_**

>*IMPORTANTE: Para poder tener acceso a este proyecto, revisar el archivo DM_SUPERMAYORISTA.sln dentro de la carpeta "DM_SPUERMAYORISTA_SSIS"*

<div align="center"> 
  <img src="readme_img/SSIS_0.png" width="800px" height="500px">
</div>

## FLUJO_DIM_DOCUMENTO
  
  ### SOURCE_DOCUMENTO
  
  ```sql
  SELECT cod_tipo_documento,nombre_tipo_documento, SUBSTRING(nombre_tipo_documento,CHARINDEX(' ',nombre_tipo_documento)+1,1) AS INDICADOR
  FROM TIPO_DOCUMENTO
  ```
  
  ### MERGE_LEFT_JOIN
  
  ```sql
  --Combinación externa completa
  --**Se relacionan por cod_tipo_documento = COD_TIPO_DOCUMENTO**
  --Se traen las siguientes columnas::
    cod_tipo_documento (Columna del origen) 
    nombre_tipo_documento (Columna del origen)
    COD_TIPO_DOCUMENTO (Columna del destino)
    INDICADOR (Columna del origen)
  ```
   
  ### SPLIT_COD_TIPO_DOCUMENTO_ISNULL
  
  ```sql
  --Se crea la condición dentro de la caja de División Condicional
  INSERT_UPDATE = ISNULL(COD_TIPO_DOCUMENTO)
  ```
  
<div align="center"> 
  <img src="readme_img/SSIS_documento.png" width="800px" height="500px">
</div>
## FLUJO_DIM_VENDEDOR

  ### SOURCE_VENDEDOR
  
  ```sql
  SELECT V.ID,V.COD_VENDEDOR, CONCAT(V.apellido_vendedor,', ',V.NOMBRE_VENDEDOR) AS NOMBRE_COMPLETO, S.ID AS SUPERVISOR
  FROM VENDEDOR_PRUEBA V
  FULL OUTER JOIN VENDEDOR_PRUEBA S ON V.SUPERVISOR=S.COD_VENDEDOR
  WHERE V.ID IS NOT NULL AND V.NOMBRE_VENDEDOR IS NOT NULL
  ```
  ### MERGE_LEFT_JOIN
  
  ```sql
  --Combinación externa completa
  --**Se relacionan por COD_VENDEDOR = COD_VENDEDOR**
  --Se traen las siguientes columnas::
    COD_VENDEDOR (Columna del origen) 
    NOMBRE_COMPLETO (Columna del origen)
    SUPERVISOR (Columna del origen) 
    COD_VENDEDOR (Columna del destino) >> COD_VENDEDOR_DIM
    ID (Columna del origen)
  ```
   
  ### SPLIT_COD_VENDEDOR_DIM_ISNULL
  
  ```sql
  --Se crea la condición dentro de la caja de División Condicional
  INSERT_UPDATE = ISNULL(COD_VENDEDOR_DIM)
  ```
  
  ### SCRIPT_SUPERVISOR_REPLACE_ID
  
  > *Este script reemplaza el SUPERVISOR por su ID de vendedor*
  
  ```c#
  public override void Entrada0_ProcessInputRow(Entrada0Buffer Row)
    {
        if (Row.SUPERVISOR_IsNull)
        {
            Row.SUPERVISOR = Row.ID;
        }
    }
  ```
  
<div align="center"> 
  <img src="readme_img/SSIS_vendedor.png" width="800px" height="500px">
</div>

## FLUJO_DIM_VENTA

  ### SOURCE_VENTA
  
  ```sql
  SELECT cod_documento AS COD_VENTA, estado
  FROM VENTA
  ```
  ### MERGE_LEFT_JOIN
  
  ```sql
  --Combinación externa completa
  --**Se relacionan por COD_VENTA = COD_VENTA**
  --Se traen las siguientes columnas::
    COD_VENTA (Columna del origen) 
    estado (Columna del origen)
    COD_VENTA (Columna del destino) >> COD_VENTA_DIM
  ```
   
  ### SPLIT_COD_VENTA_DIM_ISNULL
  
  ```sql
  --Se crea la condición dentro de la caja de División Condicional
  INSERT_UPDATE = ISNULL([COD_VENTA _DIM])
  ```
  
<div align="center"> 
  <img src="readme_img/SSIS_venta.png" width="800px" height="500px">
</div>

## FLUJO_DIM_BANCO

  ### SOURCE_BANCO
  
  ```sql
  SELECT *
  FROM BANCO
  ```
  ### MERGE_LEFT_JOIN
  
  ```sql
  --Combinación externa completa
  --**Se relacionan por cod_banco = COD_BANCO**
  --Se traen las siguientes columnas::
    cod_banco (Columna del origen) 
    nombre_banco (Columna del origen)
    comision (Columna del origen)
    COD_BANCO (Columna del destino)
  ```
   
  ### SPLIT_COD_BANCO_ISNULL
  
  ```sql
  --Se crea la condición dentro de la caja de División Condicional
  INSERT_UPDATE = ISNULL(COD_BANCO)
  ```
  
<div align="center"> 
  <img src="readme_img/SSIS_banco.png" width="800px" height="500px">
</div>

 ## FLUJO_DIM_PERIODO

  ### SOURCE_PERIODO
  
  ```sql
  SELECT DISTINCT CONCAT(CAST(YEAR(fecha_venta)as varchar(4)),
  CAST(RIGHT('0'+ RTRIM(MONTH(fecha_venta)),2)as varchar(2)),CAST(RIGHT('0'+ RTRIM(DAY(fecha_venta)),2)as varchar(2))) as codigo,YEAR(fecha_venta) AS ANIO,
  MONTH(fecha_venta) AS MES,DAY(fecha_venta) AS DIA
  FROM VENTA
  ```
  
  ### DERIVED_COLUMN_FECHA_OFICIAL
  
  ```sql
  --Se crea la columna derivada dentro de la caja de Columna Derivada
  FECHA_OFICIAL = (DT_DBDATE)(SUBSTRING(codigo,1,4) + "/" + SUBSTRING(codigo,5,2) + "/" + SUBSTRING(codigo,7,2))
  ```
  
  ### MERGE_LEFT_JOIN
  
  ```sql
  --Combinación externa completa
  --**Se relacionan por codigo = COD_PERIODO**
  --Se traen las siguientes columnas::
    codigo (Columna del origen) 
    FECHA_OFICIAL
    ANIO (Columna del origen)
    MES (Columna del origen)
    DIA (Columna del origen)
    COD_PERIODO (Columna del destino)
  ```
   
  ### SPLIT_COD_PERIODO_ISNULL
  
  ```sql
  --Se crea la condición dentro de la caja de División Condicional
  INSERT_UPDATE = ISNULL(COD_PERIODO)
  ```
  
<div align="center"> 
  <img src="readme_img/SSIS_periodo.png" width="800px" height="500px">
</div>

 ## FLUJO_DIM_CATEGORIA

  ### SOURCE_CATEGORIA
  
  ```sql
  SELECT DISTINCT categoria_producto
  FROM PRODUCTO
  ```
  ### MERGE_LEFT_JOIN
  
  ```sql
  --Combinación externa completa
  --**Se relacionan por categoria_producto = DESCRIPCION**
  --Se traen las siguientes columnas::
    categoria_producto (Columna del origen) 
    DESCRIPCION (Columna del destino)
  ```
   
  ### SPLIT_DESCRIPCION_ISNULL
  
  ```sql
  --Se crea la condición dentro de la caja de División Condicional
  INSERT_UPDATE = ISNULL(DESCRIPCION)
  ```
  
<div align="center"> 
  <img src="readme_img/SSIS_categoria.png" width="800px" height="500px">
</div>

## FLUJO_DIM_SUBCATEGORIA

  ### SOURCE_SUBCATEGORIA
  
  ```sql
  SELECT DISTINCT P.subcategoria_producto,C.CATEGORIA_KEY AS FK_CATEGORIA_S
  FROM PRODUCTO P
  JOIN DM_SUPERMAYORISTA.DBO.DIM_CATEGORIA C ON C.DESCRIPCION = P.categoria_producto
  ```
  ### MERGE_LEFT_JOIN
  
  ```sql
  --Combinación externa completa
  --**Se relacionan por subcategoria_producto = DESCRIPCION**
  --Se traen las siguientes columnas::
    subcategoria_producto (Columna del origen)
    FK_CATEGORIA_S (Columna del origen)
    DESCRIPCION (Columna del destino)
  ```
   
  ### SPLIT_DESCRIPCION_ISNULL
  
  ```sql
  --Se crea la condición dentro de la caja de División Condicional
  INSERT_UPDATE = ISNULL(DESCRIPCION)
  ```
<div align="center"> 
  <img src="readme_img/SSIS_subcategoria.png" width="800px" height="500px">
</div>

## FLUJO_DIM_PRODUCTO

  ### SOURCE_PRODUCTO
  
  ```sql
  SELECT DISTINCT P.cod_producto, P.nombre_producto, S.SUBCATEGORIA_KEY AS FK_SUBCATEGORIA_P
  FROM PRODUCTO P
  JOIN DM_SUPERMAYORISTA.DBO.DIM_SUBCATEGORIA S ON S.DESCRIPCION = P.subcategoria_producto
  ORDER BY P.cod_producto
  ```
  ### MERGE_LEFT_JOIN
  
  ```sql
  --Combinación externa completa
  --**Se relacionan por cod_producto = FK_SUBCATEGORIA**
  --Se traen las siguientes columnas::
    cod_producto (Columna del origen)
    nombre_producto (Columna del origen)
    FK_SUBCATEGORIA_P (Columna del origen)
    FK_SUBCATEGORIA (Columna del destino)
  ```
   
  ### SPLIT_FK_SUBCATEGORIA_ISNULL
  
  ```sql
  --Se crea la condición dentro de la caja de División Condicional
  INSERT_UPDATE = ISNULL(FK_SUBCATEGORIA)
  ```
  
<div align="center"> 
  <img src="readme_img/SSIS_producto.png" width="800px" height="500px">
</div>

## FLUJO_DIM_DEPARTAMENTO

  ### SOURCE_DEPARTAMENTO
  
  ```sql
  SELECT DISTINCT departamento_cliente
  FROM CLIENTE
  ```
  ### MERGE_LEFT_JOIN
  
  ```sql
  --Combinación externa completa
  --**Se relacionan por departamento_cliente = NOMBRE_DEPARTAMENTO**
  --Se traen las siguientes columnas::
    departamento_cliente (Columna del origen)
    NOMBRE_DEPARTAMENTO (Columna del destino)
  ```
   
  ### SPLIT_NOMBRE_DEPARTAMENTO_ISNULL
  
  ```sql
  --Se crea la condición dentro de la caja de División Condicional
  INSERT_UPDATE = ISNULL(NOMBRE_DEPARTAMENTO)
  ```
<div align="center"> 
  <img src="readme_img/SSIS_departamento.png" width="800px" height="500px">
</div>

## FLUJO_DIM_DISTRITO

  ### SOURCE_DISTRITO
  
  ```sql
  SELECT DISTINCT L.distrito_cliente,L.ubigeo_cliente,L.punto_geografico,E.DEPARTAMENTO_KEY AS FK_DEPARTAMENTO_D
  FROM CLIENTE L
  JOIN DM_SUPERMAYORISTA.DBO.DIM_DEPARTAMENTO E ON E.NOMBRE_DEPARTAMENTO = L.departamento_cliente
  ORDER BY L.distrito_cliente,E.DEPARTAMENTO_KEY
  ```
  ### MERGE_LEFT_JOIN
  
  ```sql
  --Combinación externa completa
  --**Se relacionan por distrito_cliente = NOMBRE_DISTRITO**
  --Se traen las siguientes columnas::
    distrito_cliente (Columna del origen)
    ubigeo_cliente (Columna del origen)
    punto_geografico (Columna del origen)
    FK_DEPARTAMENTO_D (Columna del origen)
    NOMBRE_DISTRITO (Columna del destino)
  ```
   
  ### SPLIT_NOMBRE_DISTRITO_ISNULL
  
  ```sql
  --Se crea la condición dentro de la caja de División Condicional
  INSERT_UPDATE = ISNULL(NOMBRE_DISTRITO)
  ```
  
<div align="center"> 
  <img src="readme_img/SSIS_distrito.png" width="800px" height="500px">
</div>

## FLUJO_DIM_CLIENTE

  ### SOURCE_CLIENTE
  
  ```sql
  SELECT DISTINCT CL.dni_cliente,UB.DISTRITO_KEY AS FK_DISTRITO_C,DT.DEPARTAMENTO_KEY AS FK_DEPARTAMENTO_C
  FROM CLIENTE CL
  JOIN DM_SUPERMAYORISTA.DBO.DIM_DISTRITO UB ON UB.UBIGEO=CL.ubigeo_cliente
  JOIN DM_SUPERMAYORISTA.DBO.DIM_DEPARTAMENTO DT ON DT.NOMBRE_DEPARTAMENTO=CL.departamento_cliente
  ```
  ### MERGE_LEFT_JOIN
  
  ```sql
  --Combinación externa completa
  --**Se relacionan por dni_cliente = DNI_CLIENTE**
  --Se traen las siguientes columnas::
    dni_cliente (Columna del origen)
    FK_DISTRITO_C (Columna del origen)
    DNI_CLIENTE (Columna del destino)
  ```
   
  ### SPLIT_DNI_CLIENTE_ISNULL
  
  ```sql
  --Se crea la condición dentro de la caja de División Condicional
  INSERT_UPDATE = ISNULL(DNI_CLIENTE)
  ```
  
<div align="center"> 
  <img src="readme_img/SSIS_cliente.png" width="800px" height="500px">
</div>

## CORRECCION_ORTOGRAFICA

> *OJO: Este proceso se hace antes de la carga dentro de FACT_SUPERMAYORISTA para que no haya pérdida de datos.*

  ```sql
  UPDATE VENTA_PRUEBA
  SET VENDEDOR = REPLACE(vendedor,'Ã‘','Ñ')
  ```
  
<div align="center"> 
  <img src="readme_img/SSIS_correccion.png" width="800px" height="500px">
</div>

## FLUJO_FACT_SUPERMAYORISTA

  ### SOURCE_CLIENTE
  
  ```sql
  SELECT DISTINCT X1.VENTA_KEY AS FK_VENTA_F,X2.DNI_CLIENTE AS FK_CLIENTE_F,X3.VENDEDOR_KEY AS FK_VENDEDOR_F,7 AS FK_BANCO_F,
				X4.PRODUCTO_KEY AS FK_PRODUCTO_F,X5.PERIODO_KEY AS FK_PERIODO_F,X6.DOCUMENTO_KEY AS FK_DOCUMENTO_F,
				Y.cantidad,Y.precio_unitario
  FROM VENTA_PRUEBA X
  JOIN DETALLE_VENTA Y ON Y.cod_documento=X.cod_documento
  JOIN DM_SUPERMAYORISTA.DBO.DIM_VENTA X1 ON X1.COD_VENTA=X.cod_documento
  JOIN DM_SUPERMAYORISTA.DBO.DIM_CLIENTE X2 ON X2.DNI_CLIENTE=X.dni_cliente
  JOIN DM_SUPERMAYORISTA.DBO.DIM_VENDEDOR X3 ON X3.NOMBRE_VENDEDOR=X.vendedor
  JOIN DM_SUPERMAYORISTA.DBO.DIM_PRODUCTO X4 ON X4.COD_PRODUCTO=Y.cod_producto
  JOIN DM_SUPERMAYORISTA.DBO.DIM_PERIODO X5 ON X5.COD_PERIODO=CONCAT(CAST(YEAR(X.fecha_venta)as varchar(4)),CAST(RIGHT('0'+ RTRIM(MONTH(X.fecha_venta)),2)as varchar(2)),CAST(RIGHT('0'+ RTRIM(DAY(X.fecha_venta)),2)as   varchar(2)))
  JOIN DM_SUPERMAYORISTA.DBO.DIM_DOCUMENTO X6 ON X6.TIPO_DOCUMENTO=X.nombre_tipo_documento
  ORDER BY X3.VENDEDOR_KEY
  ```
  ### MERGE_LEFT_JOIN
  
  ```sql
  --Combinación externa completa
  
  /*Se relacionan por:
    - FK_VENTA_F = FK_VENTA
    - FK_CLIENTE_F = FK_CLIENTE
    - FK_VENDEDOR_F = FK_VENDEDOR
    - FK_PRODUCTO_F = FK_PRODUCTO
    - FK_PERIODO_F = FK_PERIODO
    - FK_DOCUMENTO_F = FK_TIPO_DOCUMENTO
  */
  --Se traen las siguientes columnas::
    FK_VENTA_F (Columna del origen)
    FK_CLIENTE_F (Columna del origen)
    FK_VENDEDOR_F (Columna del origen)
    FK_PRODUCTO_F (Columna del origen)
    FK_PERIODO_F (Columna del origen)
    FK_DOCUMENTO_F (Columna del origen)
    cantidad (Columna del origen)
    precio_unitario (Columna del origen)
    FK_VENTA (Columna del destino)
    FK_CLIENTE (Columna del destino)
    FK_VENDEDOR (Columna del destino)
    FK_PRODUCTO (Columna del destino)
    FK_PERIODO (Columna del destino)
    FK_TIPO_DOCUMENTO (Columna del destino)
    FK_BANCO (Columna del destino)
    FK_BANCO_F (Columna del origen)
  ```
   
  ### SPLIT_(FK_VENTA_FK_CLIENTE_FK_VENDEDOR_FK_BANCO_FK_PRODUCTO_FK_PERIODO_FK_TIPO_DOCUMENTO)_ISNULL
  
  ```sql
  --Se crea la condición dentro de la caja de División Condicional
  INSERT_UPDATE = ISNULL(FK_VENTA) && ISNULL(FK_CLIENTE) && ISNULL(FK_VENDEDOR) && ISNULL(FK_BANCO) && ISNULL(FK_PRODUCTO) && ISNULL(FK_PERIODO) && ISNULL(FK_TIPO_DOCUMENTO)
  ```
  
<div align="center"> 
  <img src="readme_img/SSIS_fact_1.png" width="800px" height="500px">
</div>

## BUCLE_TXT_VENTA

<div align="center"> 
  <img src="readme_img/SSIS_fact_2.png" width="800px" height="500px">
</div>

  ### FLUJO_TRANSACCIONES_TXT_VENTAS_&_PERIODO
  
   #### SOURCE_TRANSACCIONES_TXT
   
   > *OJO: Para poder usar los archivos txt, dbe ubicar la carpeta __Folder_Archivos_TXT__ en un directorio de su preferencia para poder tener accesos a ellos.*
   
   <div align="center"> 
     <img src="readme_img/SSIS_TXT_0.png" width="800px" height="500px">
     <img src="readme_img/SSIS_TXT_1.png" width="800px" height="500px">
   </div>
   
   >*OJO: El MULTICAST_DIM_VENTA_&_DIM_PERIODO se usó para partir los procesos que se le va a dar a la data, la 1° parte viene a estar dada por la parte de la izquierdo y la 2° parte por el lado derecho*
  
   #### DATA_CONVERSION_METADATA (PRECIO_UNIT_ANTIDAD_NUMERO_PRODUCTO_NUMERO_VENDEDOR) [1° Parte]
   
   > Se hace la conversión de datos dentro de la caja Conversión de Datos
   
   |   Columna de entrada    |      Alias de salida     |     Tipo de datos    | Precisión |
   |:-----------------------:|:------------------------:|:--------------------:|:---------:|
   | PRECIO_UNIT             | Copia de PRECIO_UNIT     | flotante[DT_R4]      |           |
   | CANTIDAD                | Copia de CANTIDAD        | numérico[DT_NUMERIC] | 18        |
   | NUMERO_PRODUCTO         | Copia de NUERMO_PRODUCTO | numérico[DT_NUMERIC] | 18        |
   | NUMERO_VENDEDOR         | Copia de NUMERO_VENDEDOR | numérico[DT_NUMERIC] | 18        |

   #### ADD_COLUMN_CODIGO_VENTA
   
   ```sql
   # Se crea la columna derivada
   COPIA_NUMERO_VENTA = TIPO_DOCUMENTO + SUBSTRING(NUMERO_VENTA,FINDSTRING(NUMERO_VENTA,"0",1) + 1,10)
   ```
   
   #### SINGLE_ROW_CODIGO_VENTA
   
   ```sql
   # Se eliminan las columnas innecesarias, quedando solo la columna:
   COPIA_NUMERO_VENTA
   ```
   
   #### DISTINCT_CODIGO_VENTA
   
   ```sql
   # Se eliminan duplicados para la columna:
   COPIA_NUMERO_VENTA
   ```
   
   #### ADD_COLUMN_ESTADO
   
   ```sql
   # Se crea la columna derivada
   ESTADO = "VENDIDO"
   ```
   
   #### DATA_ONVERSION_METADATA (CODIGO_VENTA_ESTADO)
   
   > Se hace la conversión de datos dentro de la caja Conversión de Datos
   
   |   Columna de entrada    |         Alias de salida         |     Tipo de datos    | Longitud  |
   |:-----------------------:|:-------------------------------:|:--------------------:|:---------:|
   | COPIA_NUMERO_VENTA      | Copia de COPIA_NUMERO_VENTA     | cadena[DT_STR]       | 13        |
   | ESTADO                  | Copia de ESTADO                 | cadena[DT_STR]       | 7         |

   #### MERGE_LEFT_JOIN_TO_DIM_VENTA
   
   ```sql
  --Combinación externa completa
  --**Se relacionan por Copia de COPIA_NUMERO_VENTA = COD_VENTA**
  --Se traen las siguientes columnas::
    Copia de COPIA_NUMERO_VENTA (Columna del origen)
    Copia de ESTADO (Columna del origen)
    COD_VENTA (Columna del destino)
   ```
  
  #### SPLIT_COD_VENTA_ISNULL
  
  ```sql
  --Se crea la condición dentro de la caja de División Condicional
  INSERT_UPDATE = ISNULL(COD_VENTA)
  ```
  
  #### SINGLE_ROW_FECHA_VENTA [2° Parte]
  
  ```sql
   # Se eliminan las columnas innecesarias, quedando solo la columna:
   FECHA_VENTA
   ```
   
  #### DISTINCT_FECHA_VENTA
  
  ```sql
   # Se eliminan duplicados para la columna:
   FECHA_VENTA
   ```
   
  #### ADD_COLUMNS (CODIGO_FECHA_OFICIAL_ANIO_OFICIAL_MES_OFICIAL_DIA_OFICIAL)
  
  ```sql
   # Se crean las columnas derivadas
   CODIGO_FECHA_OFICIAL = (DT_DBDATE)(SUBSTRING(FECHA_VENTA,1,4) + "/" + SUBSTRING(FECHA_VENTA,5,2) + "/" + SUBSTRING(FECHA_VENTA,7,2))
   ANIO_OFICIAL = SUBSTRING(FECHA_VENTA,1,4)
   MES_OFICIAL = SUBSTRING(FECHA_VENTA,5,2)
   DIA_OFICIAL = SUBSTRING(FECHA_VENTA,7,2)
  ```
  
  #### MERGE_LEFT_JOIN_TO_DIM_PERIODO
  
  ```sql
  --Combinación externa completa
  --**Se relacionan por Copia de FECHA_VENTA = COD_PERIODO**
  --Se traen las siguientes columnas::
    FECHA_VENTA (Columna del origen)
    CODIGO_FECHA_OFICIAL
    ANIO_OFICIAL
    MES_OFICIAL
    DIA_OFICIAL
    COD_PERIODO (Columna del destino)
   ```
  #### SPLIT_COD_PERIODO_ISNULL
  
  ```sql
  --Se crea la condición dentro de la caja de División Condicional
  INSERT_UPDATE = ISNULL(COD_PERIODO)
  ```
  
  > *Esta imagen es el resultado de aplicar las 1° y 2° parte de los procedimientos para el tratamiento de los archivos txt y su respectiva inserción dentro del del FACT_SUPERMAYORISTA*
<div align="center"> 
  <img src="readme_img/SSIS_fact_3.png" width="800px" height="500px">
</div>

### FLUJO_TRANSACCIONES_TXT_FACT
  
 #### SOURCE_TRANSACCIONES_TXT_FACT
    
   <div align="center"> 
     <img src="readme_img/SSIS_TXT_0.png" width="800px" height="500px">
     <img src="readme_img/SSIS_TXT_1.png" width="800px" height="500px">
   </div>

 #### DATA_CONVERSION_METADATA (PRECIO_UNIT_CANTIDAD_NUMERO_PRODUCTO_NUMERO_VENDEDOR)
 
 > Se hace la conversión de datos dentro de la caja Conversión de Datos
   
   |   Columna de entrada    |      Alias de salida     |     Tipo de datos    | Precisión |
   |:-----------------------:|:------------------------:|:--------------------:|:---------:|
   | PRECIO_UNIT             | Copia de PRECIO_UNIT     | flotante[DT_R4]      |           |
   | CANTIDAD                | Copia de CANTIDAD        | numérico[DT_NUMERIC] | 18        |
   | NUMERO_PRODUCTO         | Copia de NUERMO_PRODUCTO | numérico[DT_NUMERIC] | 18        |
   | NUMERO_VENDEDOR         | Copia de NUMERO_VENDEDOR | numérico[DT_NUMERIC] | 18        |
 
 #### ADD_COLUMN_CODIGO_VENTA
   
 ```sql
 # Se crea la columna derivada
 COPIA_NUMERO_VENTA = TIPO_DOCUMENTO + SUBSTRING(NUMERO_VENTA,FINDSTRING(NUMERO_VENTA,"0",1) + 1,10)
 ```
 #### DELETE_UNNECESSARY_COLUMNS
 
 ```sql
 # Se eliminan las columnas innecesarias, quedando solo la columna:
 BANCO
 TIPO_DOCUMENTO
 FECHA_VENTA
 Copia de PRECIO_UNIT
 Copia de CANTIDAD
 Copia de NUMERO_PRODUCTO
 Copia de NUMERO_VENDEDOR
 COPIA_NUMERO_VENTA
 DNI
 ```
 
 #### MERGE_JOIN_DIM_VENTA
 
 ```sql
 --Combinación completa
 --**Se relacionan por Copia de CODIGO_NUMERO_VENTA = VENTA_KEY**
 --Se traen las siguientes columnas::
 VENTA_KEY
 BANCO (Columna del origen)
 TIPO_DOCUMENTO (Columna del origen)
 FECHA_VENTA (Columna del origen)
 Copia de PRECIO_UNIT (Columna del origen)
 Copia de CANTIDAD (Columna del origen)
 Copia de NUMERO_PRODUCTO (Columna del origen)
 Copia de NUMERO_VENDEDOR (Columna del origen)
 DNI (Columna del origen)
 ```
 
 #### MERGE_JOIN_DIM_BANCO
 
 ```sql
 --Combinación completa
 --**Se relacionan por Copia de BANCO = COD_BANCO**
 --Se traen las siguientes columnas::
 VENTA_KEY (Columna del origen)
 BANCO_KEY 
 TIPO_DOCUMENTO (Columna del origen)
 FECHA_VENTA (Columna del origen)
 Copia de PRECIO_UNIT (Columna del origen)
 Copia de CANTIDAD (Columna del origen)
 Copia de NUMERO_PRODUCTO (Columna del origen)
 Copia de NUMERO_VENDEDOR (Columna del origen)
 DNI (Columna del origen)
 ```
 
 #### MEGE_JOIN_DIM_DOCUMENTO
 
 ```sql
 --Combinación completa
 --**Se relacionan por Copia de TIPO_DOCUMENTO = INDICADOR**
 --Se traen las siguientes columnas::
 VENTA_KEY (Columna del origen)
 BANCO_KEY (Columna del origen)
 FECHA_VENTA (Columna del origen)
 Copia de PRECIO_UNIT (Columna del origen)
 Copia de CANTIDAD (Columna del origen)
 Copia de NUMERO_PRODUCTO (Columna del origen)
 Copia de NUMERO_VENDEDOR (Columna del origen)
 DNI (Columna del origen)
 DOCUMENTO_KEY
 ```
 
 #### ADD_COLUMNS (VENDEDOR_OFICIAL_PRODUCTO_OFICIAL)
 
 ```sql
 # Se crea las columnas derivadas
 VENDEDOR_OFICIAL = (DT_I4)[Copia de NUMERO_VENDEDOR]
 PRODUCTO_OFICIAL = (DT_I4)[Copia de NUMERO_PRODUCTO]
 ```
 
 #### DATA_CONVERSION_METADATA_DNI
 
  > Se hace la conversión de datos dentro de la caja Conversión de Datos
   
   |   Columna de entrada    |      Alias de salida     |     Tipo de datos    | Longitud  |
   |:-----------------------:|:------------------------:|:--------------------:|:---------:|
   | DNI                     | Copia deDNI              | cadena[DT_STR]       | 14        |
 
 #### MERGE_JOIN_DIM_PERIODO
 
 ```sql
 --Combinación completa
 --**Se relacionan por Copia de FECHA_VENTA = COD_PERIODO**
 --Se traen las siguientes columnas::
 VENTA_KEY (Columna del origen)
 BANCO_KEY (Columna del origen)
 Copia de PRECIO_UNIT (Columna del origen)
 Copia de CANTIDAD (Columna del origen)
 DOCUMENTO_KEY (Columna del origen)
 VENDEDOR_OFICIAL (Columna del origen)
 PRODUCTO_OFICIAL (Columna del origen)
 Copia de DNI (Columna del origen)
 PERIODO_KEY
 ```
 
 #### MERGE_LEFT_JOIN (Cruce con SOURCE_FACT_SUPERMAYORISTA)
 
 ```sql
 --Combinación externa completa
  
  /*Se relacionan por:
    - VENTA_KEY = FK_VENTA
    - BANCO_KEY = FK_BANCO
    - DOCUMENTO_KEY = FK_TIPO_DOCUMENTO
    - VENDEDOR_OFICIAL = FK_VENDEDOR
    - PRODUCTO_OFICIAL = FK_PRODUCTO
    - Copia de DNI = FK_CLIENTE
    - PERIODO_KEY = FK_PERIODO
  */
 --Se traen las siguientes columnas::
 VENTA_KEY (Columna del origen)
 Copia de DNI (Columna del origen)
 VENDEDOR_OFICIAL (Columna del origen)
 BANCO_KEY (Columna del origen)
 PRODUCTO_OFICIAL (Columna del origen)
 PERIODO_KEY (Columna del origen)
 DOCUMENTO_KEY (Columna del origen)
 Copia de CANTIDAD (Columna del origen)
 Copia de PRECIO_UNIT (Columna del origen)
 FK_VENTA
 FK_CLIENTE
 FK_VENDEDOR
 FK_BANCO
 FK_PRODUCTO
 FK_PERIODO
 FK_TIPO_DOCUMENTO 
 ```
 
<div align="center"> 
  <img src="readme_img/SSIS_fact_4.png" width="800px" height="500px">
</div>

## Planificación del proyecto

### Alcance del negocio

- El presente proyecto busca ayudar con la gestión del área de ventas de la empresa SuperMayorista mediante informes de análisis de la información completos, veraces y en tiempo real que permita apoyar en la toma de decisiones.
- **_SE ESTÁ TRABAJANDO CON UN MODELO COPO DE NIEVE_**, es recomendable siempre tirar para un **_MODELO ESTRELLA_**, pero ello va a depender de la empresa en la que esté trabajando sea pequeña, mediana o grande.

## Definición de requerimientos para la empresa SUPERMAYORISTA

### Requisitos del negocio

<table>
    <thead>
        <tr>
            <th><strong>Identificador</strong></th>
            <th><i>R-01</i></th>
            <th><strong>Nombre</strong></th>
            <th><i>TOP N V VS T</i></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>Tipo</strong></td>
            <td align="center"><i>Funcional</i></td>
            <td align="center"><strong>Fecha</strong></td>
            <td align="center"><i>08/12/2022</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Prioridad</strong></td>
            <td align="center"><i>Alta</i></td>
            <td align="center"><strong>Necesidad</strong></td>
            <td align="center"><i>Si</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Descripción</strong></td>
            <td colspan=3 align="center"><i>"El total de ventas por periodo"</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Datos dimensionales</strong></td>
            <td align="center"><i>Producto</i></td>
            <td align="center"><strong>Datos Hechos</strong></td>
            <td align="center"><i>Más vendidos</i></td>
        </tr>
    </tbody>
</table>


<table>
    <thead>
        <tr>
            <th><strong>Identificador</strong></th>
            <th><i>R-02</i></th>
            <th><strong>Nombre</strong></th>
            <th><i>TOP N PROD</i></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>Tipo</strong></td>
            <td align="center"><i>Funcional</i></td>
            <td align="center"><strong>Fecha</strong></td>
            <td align="center"><i>08/12/2022</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Prioridad</strong></td>
            <td align="center"><i>Alta</i></td>
            <td align="center"><strong>Necesidad</strong></td>
            <td align="center"><i>Si</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Descripción</strong></td>
            <td colspan=3 align="center"><i>"Cuáles fueron los productos más vendidos"</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Datos dimensionales</strong></td>
            <td align="center"><i>Periodo</i></td>
            <td align="center"><strong>Datos Hechos</strong></td>
            <td align="center"><i>Cantidad de ventas</i></td>
        </tr>
    </tbody>
</table>

<table>
    <thead>
        <tr>
            <th><strong>Identificador</strong></th>
            <th><i>R-03</i></th>
            <th><strong>Nombre</strong></th>
            <th><i>TOP N VE</i></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>Tipo</strong></td>
            <td align="center"><i>Funcional</i></td>
            <td align="center"><strong>Fecha</strong></td>
            <td align="center"><i>08/12/2022</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Prioridad</strong></td>
            <td align="center"><i>Alta</i></td>
            <td align="center"><strong>Necesidad</strong></td>
            <td align="center"><i>Si</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Descripción</strong></td>
            <td colspan=3 align="center"><i>"Quiénes fueron los vendedores con las mayores ventas realizadas"</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Datos dimensionales</strong></td>
            <td align="center"><i>Vendedor</i></td>
            <td align="center"><strong>Datos Hechos</strong></td>
            <td align="center"><i>Mayor cantidad de ventas</i></td>
        </tr>
    </tbody>
</table>

<table>
    <thead>
        <tr>
            <th><strong>Identificador</strong></th>
            <th><i>R-04</i></th>
            <th><strong>Nombre</strong></th>
            <th><i>TOP N S VS F</i></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>Tipo</strong></td>
            <td align="center"><i>Funcional</i></td>
            <td align="center"><strong>Fecha</strong></td>
            <td align="center"><i>08/12/2022</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Prioridad</strong></td>
            <td align="center"><i>Alta</i></td>
            <td align="center"><strong>Necesidad</strong></td>
            <td align="center"><i>Si</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Descripción</strong></td>
            <td colspan=3 align="center"><i>"Quiénes fueron los supervisores que tuvieron baja efectividad (cantidad de ventas en cuerto periodo)"</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Datos dimensionales</strong></td>
            <td align="center"><i>Vendedor</i></td>
            <td align="center"><strong>Datos Hechos</strong></td>
            <td align="center"><i>Cantidad de ventas</i></td>
        </tr>
    </tbody>
</table>

<table>
    <thead>
        <tr>
            <th><strong>Identificador</strong></th>
            <th><i>R-05</i></th>
            <th><strong>Nombre</strong></th>
            <th><i>TOP N V AND C</i></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>Tipo</strong></td>
            <td align="center"><i>Funcional</i></td>
            <td align="center"><strong>Fecha</strong></td>
            <td align="center"><i>08/12/2022</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Prioridad</strong></td>
            <td align="center"><i>Alta</i></td>
            <td align="center"><strong>Necesidad</strong></td>
            <td align="center"><i>Si</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Descripción</strong></td>
            <td colspan=3 align="center"><i>"Cuál es la zona con maores ventas y mayores cantidades de productos vendidos"</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Datos dimensionales</strong></td>
            <td align="center"><i>Zona</i></td>
            <td align="center"><strong>Datos Hechos</strong></td>
            <td align="center"><i>Mayor cantidad de ventas, mayor cantidad de productos vendidos</i></td>
        </tr>
    </tbody>
</table>

<table>
    <thead>
        <tr>
            <th><strong>Identificador</strong></th>
            <th><i>R-06</i></th>
            <th><strong>Nombre</strong></th>
            <th><i>TOP N B VS AC</i></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>Tipo</strong></td>
            <td align="center"><i>Funcional</i></td>
            <td align="center"><strong>Fecha</strong></td>
            <td align="center"><i>08/12/2022</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Prioridad</strong></td>
            <td align="center"><i>Alta</i></td>
            <td align="center"><strong>Necesidad</strong></td>
            <td align="center"><i>Si</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Descripción</strong></td>
            <td colspan=3 align="center"><i>"Cuál es el banco más factible para solicitar un ajuste de comisión"</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Datos dimensionales</strong></td>
            <td align="center"><i>Banco</i></td>
            <td align="center"><strong>Datos Hechos</strong></td>
            <td align="center"><i>Mayor comisión obtenida por las ventas</i></td>
        </tr>
    </tbody>
</table>

<table>
    <thead>
        <tr>
            <th><strong>Identificador</strong></th>
            <th><i>R-07</i></th>
            <th><strong>Nombre</strong></th>
            <th><i>TOP N VE VS V AND C AND M</i></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>Tipo</strong></td>
            <td align="center"><i>Funcional</i></td>
            <td align="center"><strong>Fecha</strong></td>
            <td align="center"><i>08/12/2022</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Prioridad</strong></td>
            <td align="center"><i>Alta</i></td>
            <td align="center"><strong>Necesidad</strong></td>
            <td align="center"><i>Si</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Descripción</strong></td>
            <td colspan=3 align="center"><i>"El top de los vendedores que más comisionan por ventas"</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Datos dimensionales</strong></td>
            <td align="center"><i>Vendedor</i></td>
            <td align="center"><strong>Datos Hechos</strong></td>
            <td align="center"><i>Mayor comisión obtenida en base a la venta</i></td>
        </tr>
    </tbody>
</table>

### Base de Datos transaccional en SQL Server

<div align="center"> 
  <img src="readme_img/fig_1.png" width="">
</div>

### Diccionario de datos del origen a nivel general de la base de datos transaccional en SQL Server 2017

#### Tablas

<table>
    <thead>
        <tr>
            <th>Tabla</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>VENTA</strong></td>
            <td><i>Se registran los movimiento de las ventas realizadas.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>DETALLE_VENTA</strong></td>
            <td><i>Se registran los detalle de las ventas realizadas.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>PRODUCTO</strong></td>
            <td><i>Se registran los productos de la empresa.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>CLIENTE</strong></td>
            <td><i>Se registran los clientes de la empresa.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>VENDEDOR</strong></td>
            <td><i>Se registran los vendedores de la empresa.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>TIPO_DOCUMENTO</strong></td>
            <td><i>Se registran los tipos de documentos que maneja la empresa.</i></td>
      </tr>
    </tbody>
</table>

#### Diccionario de datos de la base de datos transaccional en SQL Server 2017

##### VENTA

<table>
    <thead>
        <tr>
            <th>Nombre Columna</th>
            <th>Tipo de Dato</th>
            <th>Null Option</th>
            <th>Descripción</th>
            <th>PK</th>
            <th>FK</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>cod_documento</strong></td>
            <td><i>Char(10)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Código del documento</i></td>
            <td><i>PK</i></td>
            <td><i>-</i></td>
        </tr>
        <tr>
            <td align="center"><strong>dni_cliente</strong></td>
            <td><i>Char(8)</i></td>
            <td><i>Not Null</i></td>
            <td><i>DNI del cliente</i></td>
            <td><i>-</i></td>
            <td><i>FK</i></td>
        </tr>
        <tr>
            <td align="center"><strong>vendedor</strong></td>
            <td><i>Varchar(50)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Nombre del vendedor</i></td>
            <td><i>-</i></td>
            <td><i>-</i></td>
        </tr>
        <tr>
            <td align="center"><strong>nombre_tipo_documento</strong></td>
            <td><i>Varchar(20)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Nombre del tipo de documento</i></td>
            <td><i>-</i></td>
            <td><i>-</i></td>
        </tr>
        <tr>
            <td align="center"><strong>estado</strong></td>
            <td><i>Varchar(10)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Estado de la venta</i></td>
            <td><i>-</i></td>
            <td><i>-</i></td>
        </tr>
        <tr>
            <td align="center"><strong>fecha_venta</strong></td>
            <td><i>datetime</i></td>
            <td><i>Not Null</i></td>
            <td><i>fecha de la venta</i></td>
            <td><i>-</i></td>
            <td><i>-</i></td>
      </tr>
    </tbody>
</table>

##### DETALLE_VENTA

<table>
    <thead>
        <tr>
            <th>Nombre Columna</th>
            <th>Tipo de Dato</th>
            <th>Null Option</th>
            <th>Descripción</th>
            <th>PK</th>
            <th>FK</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>cod_documento</strong></td>
            <td><i>int</i></td>
            <td><i>Not Null</i></td>
            <td><i>Número de orden de compra</i></td>
            <td><i>-</i></td>
            <td><i>FK</i></td>
        </tr>
        <tr>
            <td align="center"><strong>cod_producto</strong></td>
            <td><i>Char(9)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Código del producto</i></td>
            <td><i>-</i></td>
            <td><i>FK</i></td>
        </tr>
        <tr>
            <td align="center"><strong>cantidad</strong></td>
            <td><i>int</i></td>
            <td><i>Not Null</i></td>
            <td><i>Cantidad de productos vendidos</i></td>
            <td><i>-</i></td>
            <td><i>FK</i></td>
        </tr>
    </tbody>
</table>

##### PRODUCTO

<table>
    <thead>
        <tr>
            <th>Nombre Columna</th>
            <th>Tipo de Dato</th>
            <th>Null Option</th>
            <th>Descripción</th>
            <th>PK</th>
            <th>FK</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>cod_producto</strong></td>
            <td><i>Char(9)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Código del producto</i></td>
            <td><i>PK</i></td>
            <td><i>-</i></td>
        </tr>
        <tr>
            <td align="center"><strong>nombre_producto</strong></td>
            <td><i>Varchar(50)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Nombre del producto</i></td>
            <td><i>-</i></td>
            <td><i>-</i></td>
        </tr>
        <tr>
            <td align="center"><strong>categoria_producto</strong></td>
            <td><i>Varchar(50)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Nombre de la categoría del producto</i></td>
            <td><i>-</i></td>
            <td><i>-</i></td>
        </tr>
        <tr>
            <td align="center"><strong>subcategoria_producto</strong></td>
            <td><i>Varchar(50)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Nombre de la subcategoría del producto</i></td>
            <td><i>-</i></td>
            <td><i>-</i></td>
        </tr>
        <tr>
            <td align="center"><strong>precio_unitario</strong></td>
            <td><i>decimal(8,2)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Precio Unitario del producto</i></td>
            <td><i>-</i></td>
            <td><i>-</i></td>
        </tr>
    </tbody>
</table>

##### CLIENTE

<table>
    <thead>
        <tr>
            <th>Nombre Columna</th>
            <th>Tipo de Dato</th>
            <th>Null Option</th>
            <th>Descripción</th>
            <th>PK</th>
            <th>FK</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>dni_cliente</strong></td>
            <td><i>Char(11)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Código del cliente</i></td>
            <td><i>PK</i></td>
            <td><i>-</i></td>
        </tr>
        <tr>
            <td align="center"><strong>departamento_cliente</strong></td>
            <td><i>Varchar(50)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Nombre del departamento</i></td>
            <td><i>-</i></td>
            <td><i>-</i></td>
        </tr>
        <tr>
            <td align="center"><strong>distrito_cliente</strong></td>
            <td><i>Varchar(50)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Nombre del distrito</i></td>
            <td><i>-</i></td>
            <td><i>-</i></td>
        </tr>
        <tr>
            <td align="center"><strong>punto_geografico</strong></td>
            <td><i>Varchar(50)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Punto geográfico</i></td>
            <td><i>-</i></td>
            <td><i>-</i></td>
        </tr>
        <tr>
            <td align="center"><strong>ubigeo_cliente</strong></td>
            <td><i>Varchar(50)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Ubigeo del cliente</i></td>
            <td><i>-</i></td>
            <td><i>-</i></td>
        </tr>
    </tbody>
</table>

##### VENDEDOR

<table>
    <thead>
        <tr>
            <th>Nombre Columna</th>
            <th>Tipo de Dato</th>
            <th>Null Option</th>
            <th>Descripción</th>
            <th>PK</th>
            <th>FK</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>cod_vendedor</strong></td>
            <td><i>Char(7)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Código del empleado</i></td>
            <td><i>PK</i></td>
            <td><i>-</i></td>
        </tr>
        <tr>
            <td align="center"><strong>apellido_vendedor</strong></td>
            <td><i>Varchar(30)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Apellido del vendedor</i></td>
            <td><i>-</i></td>
            <td><i>-</i></td>
        </tr>
        <tr>
            <td align="center"><strong>nombre_vendedor</strong></td>
            <td><i>Varchar(20)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Nombre del vendedor</i></td>
            <td><i>-</i></td>
            <td><i>-</i></td>
        </tr>
        <tr>
            <td align="center"><strong>supervisor</strong></td>
            <td><i>Char(7)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Apellido del empleado</i></td>
            <td><i>-</i></td>
            <td><i>-</i></td>
        </tr>
    </tbody>
</table>

##### TIPO_DOCUMENTO

<table>
    <thead>
        <tr>
            <th>Nombre Columna</th>
            <th>Tipo de Dato</th>
            <th>Null Option</th>
            <th>Descripción</th>
            <th>PK</th>
            <th>FK</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>cod_tipo_documento</strong></td>
            <td><i>Char(6)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Código del tipo de documento</i></td>
            <td><i>PK</i></td>
            <td><i>-</i></td>
        </tr>
        <tr>
            <td align="center"><strong>nombre_tipo_documento</strong></td>
            <td><i>Varchar(20)</i></td>
            <td><i>Not Null</i></td>
            <td><i>Nombre del tipo de documento</i></td>
            <td><i>-</i></td>
            <td><i>-</i></td>
        </tr>
    </tbody>
</table>

### Modelo Dimensional

Tras analizar las entrevistas y requerimientos se continuará determinando las medidas y funciones orientadas a analizar diferentes niveles de información.

#### Elección de dimensiones

Para poder determinar las dimensiones a usar en el Datamart, primero debemos elegir las variables a analizar y que los usuarios suelen utilizar para realizar sus informes. Destacan:

- [x] Fecha
- [x] Estado de la venta
- [x] Nombre del Producto
- [x] Categoría del producto
- [x] Sub categoría del producto
- [x] Nombre del vendedor
- [x] Apellido del vendedor
- [x] Supervisor del vendedor
- [x] DNI del cliente
- [x] Nombre del departamento
- [x] Nombre del distrito
- [x] Nombre del banco
- [x] Comisión del banco
- [x] Tipo de documento

<table>
    <thead>
        <tr>
            <th colspan=2 align="center">Dimensiones</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan=3 align="center"><strong>Vendedor</strong></td>
            <td><i>Nombre-Vendedor</i></td>
        </tr>
        <tr>
            <td><i>Apellido-Vendedor</i></td>
        </tr>
        <tr>
            <td><i>Supevisor-Vendedor</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Venta</strong></td>
            <td><i>Estado de la venta-Venta</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Cliente</strong></td>
            <td><i>DNI-Cliente</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Producto</strong></td>
            <td><i>Nombre-Producto</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Subcategoría</strong></td>
            <td><i>Subcategoría-Producto</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Categoría</strong></td>
            <td><i>Categoría-Producto</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Distrito</strong></td>
            <td><i>Distrito-Distrito</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Departamento</strong></td>
            <td><i>Departamento-Departamento</i></td>
        </tr>
        <tr>
            <td rowspan=2 align="center"><strong>Banco</strong></td>
            <td><i>Nombre-Banco</i></td>
        </tr>
        <tr>
            <td><i>Comisión-Banco</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Periodo</strong></td>
            <td><i>Fecha-Periodo</i></td>
        </tr>
        <tr>
            <td align="center"><strong>Documento</strong></td>
            <td><i>Tipo-Documento</i></td>
        </tr>
    </tbody>
</table>

### Dimensiones encontradas

- [x] DIM_VENDEDOR
- [x] DIM_VENTA
- [x] DIM_CLIENTE
- [x] DIM_PRODUCTO
- [x] DIM_SUBCATEGORIA
- [x] DIM_CATEGORIA
- [x] DIM_DISTRITO
- [x] DIM_DEPARTAMENTO
- [x] DIM_BANCO
- [x] DIM_PERIODO
- [x] DIM_DOCUMENTO

### Diseño lógico

<div align="center"> 
  <img src="readme_img/fig_2.png" width="">
</div>

### Diseño físico

<div align="center"> 
  <img src="readme_img/fig_3.png" width="">
</div>

#### Tablas del modelo dimensional

<table>
    <thead>
        <tr>
            <th>Tabla</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>FACT_SUPERMAYORISTA</strong></td>
            <td><i>Se registran los movimientos de cada una de las ventas.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>DIM_VENDEDOR</strong></td>
            <td><i>Se registran los vendedores.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>DIM_VENTA</strong></td>
            <td><i>Se registran las ventas realizadas y no realizadas.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>DIM_CLIENTE</strong></td>
            <td><i>Se registran los clientes que interactúan con la empresa.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>DIM_PRODUCTO</strong></td>
            <td><i>Se registran los productos con los que cuenta la empresa.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>DIM_SUBCATEGORIA</strong></td>
            <td><i>Se registran las subcategorías a las que pertenecen los productos.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>DIM_CATEGORIA</strong></td>
            <td><i>Se registran las categorías a las que pertenecen las subcategorías.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>DIM_DISTRITO</strong></td>
            <td><i>Se registran los diferentes distritos de nuestro país.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>DIM_DEPARTAMENTO</strong></td>
            <td><i>Se registran los diferentes departamentos de nuestro país y a los cuales los distritos pertenecen.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>DIM_BANCO</strong></td>
            <td><i>Se registran los bancos con los quer trabaja la empresa.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>DIM_PERIODO</strong></td>
            <td><i>Se registra el periodo en fecha, año, mes y día.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>DIM_DOCUMENTO</strong></td>
            <td><i>Se registran los diferentes tipos de documentos con los que trabaja la empresa.</i></td>
        </tr>
    </tbody>
</table>

##### Tabla Hechos: FACT_SUPERMAYORISTA

FACT_SUPERMAYORISTA está conformada por:
<table>
    <thead>
        <tr>
            <th>Campo</th>
            <th>Tipo de Dato</th>
            <th>Longitud</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>FACT_VENTA</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave foránea relacionada a la dimensión Venta.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>FK_CLIENTE</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave foránea relacionada a la dimensión Cliente.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>FK_VENDEDOR</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave foránea relacionada a la dimensión Vendedor.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>FK_BANCO</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave foránea relacionada a la dimensión Banco.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>FK_PRODUCTO</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave foránea relacionada a la dimensión Producto.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>FK_PERIODO</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave foránea relacionada a la dimensión Periodo.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>FK_TIPO_DOCUMENTO</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave foránea relacionada a la dimensión Documento.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>CANTIDAD_PRODUCTO</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Cantidad del producto.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>PRECIO_UNIT</strong></td>
            <td><i>float</i></td>
            <td><i>-</i></td>
            <td><i>Precio del producto.</i></td>
        </tr>
    </tbody>
</table>

##### Dimensión vendedor: DIM_VENDEDOR

La dimensión DIM_VENDEDOR está conformada por:
<table>
    <thead>
        <tr>
            <th>Campo</th>
            <th>Tipo de Dato</th>
            <th>Longitud</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>VENDEDOR_KEY</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave para la dimnesión vendedor.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>CODIGO_VENDEDOR</strong></td>
            <td><i>Varchar</i></td>
            <td><i>9</i></td>
            <td><i>Código para la dimensión vendedor.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>NOMBRE_VENDEDOR</strong></td>
            <td><i>Varchar</i></td>
            <td><i>65</i></td>
            <td><i>Nombre del vendedor.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>SUPERVISOR</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Número del supervisor.</i></td>
        </tr>
    </tbody>
</table>


##### Dimensión venta: DIM_VENTA

La dimensión DIM_VENTA está conformada por:
<table>
    <thead>
        <tr>
            <th>Campo</th>
            <th>Tipo de Dato</th>
            <th>Longitud</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>VENTA_KEY</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave para la dimnesión venta.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>COD_VENTA</strong></td>
            <td><i>Varchar</i></td>
            <td><i>13</i></td>
            <td><i>Código para la dimensión venta.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>ESTADO</strong></td>
            <td><i>Varchar</i></td>
            <td><i>13</i></td>
            <td><i>Nombre del estado de la venta.</i></td>
        </tr>
    </tbody>
</table>


##### Dimensión cliente: DIM_CLIENTE

La dimensión DIM_CLIENTE está conformada por:
<table>
    <thead>
        <tr>
            <th>Campo</th>
            <th>Tipo de Dato</th>
            <th>Longitud</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>DNI_CLIENTE</strong></td>
            <td><i>Varchar</i></td>
            <td><i>14</i></td>
            <td><i>Llave para la dimnesión cliente.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>FK_DISTRITO</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave foránea de la dimensión cliente.</i></td>
        </tr>
    </tbody>
</table>


##### Dimensión producto: DIM_PRODUCTO

La dimensión DIM_PRODUCTO está conformada por:
<table>
    <thead>
        <tr>
            <th>Campo</th>
            <th>Tipo de Dato</th>
            <th>Longitud</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>PRODUCTO_KEY</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave para la dimnesión producto.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>COD_PRODUCTO</strong></td>
            <td><i>Varchar</i></td>
            <td><i>12</i></td>
            <td><i>Código para la dimensión producto.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>NOMBRE_PRODUCTO</strong></td>
            <td><i>Varchar</i></td>
            <td><i>195</i></td>
            <td><i>Nombre del producto.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>FK_SUPERCATEGORIA</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave foránea para la dimensión producto.</i></td>
        </tr>
    </tbody>
</table>


##### Dimensión subcategoría: DIM_SUBCATEGORIA

La dimensión DIM_SUBCATEGORIA está conformada por:
<table>
    <thead>
        <tr>
            <th>Campo</th>
            <th>Tipo de Dato</th>
            <th>Longitud</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>SUBCATEGORIA_KEY</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave para la dimnesión subcategoría.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>DESCRIPCION</strong></td>
            <td><i>Varchar</i></td>
            <td><i>65</i></td>
            <td><i>Descripción para la dimensión subcategoría.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>FK_CATEGORIA</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave foránea para la dimensión subcategoría</i></td>
        </tr>
    </tbody>
</table>


##### Dimensión categoría: DIM_CATEGORIA

La dimensión DIM_CATEGORIA está conformada por:
<table>
    <thead>
        <tr>
            <th>Campo</th>
            <th>Tipo de Dato</th>
            <th>Longitud</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>CATEGORIA_KEY</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave para la dimnesión categoría.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>DESCRIPCION</strong></td>
            <td><i>Varchar</i></td>
            <td><i>65</i></td>
            <td><i>Nombre de la categoría.</i></td>
        </tr>
    </tbody>
</table>


##### Dimensión distrito: DIM_DISTRITO

La dimensión DIM_DISTRITO está conformada por:
<table>
    <thead>
        <tr>
            <th>Campo</th>
            <th>Tipo de Dato</th>
            <th>Longitud</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>DISTRITO_KEY</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave para la dimnesión producto.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>NOMBRE_DISTRITO</strong></td>
            <td><i>Varchar</i></td>
            <td><i>65</i></td>
            <td><i>Nombre del distrito.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>UBIGEO</strong></td>
            <td><i>Varchar</i></td>
            <td><i>13</i></td>
            <td><i>Código del producto.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>NOMBRE_DISTRITO</strong></td>
            <td><i>Varchar</i></td>
            <td><i>65</i></td>
            <td><i>Número del distrito.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>FK_DEPARTAMERNTO</strong></td>
            <td><i>Varchar</i></td>
            <td><i>65</i></td>
            <td><i>Llave foránea para la dimensión distrito.</i></td>
        </tr>
    </tbody>
</table>


##### Dimensión departamento: DIM_DEPARTAMENTO

La dimensión DIM_DEPARTAMENTO está conformada por:
<table>
    <thead>
        <tr>
            <th>Campo</th>
            <th>Tipo de Dato</th>
            <th>Longitud</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>DEPARTAMENTO_KEY</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave para la dimnesión departamento.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>NOMBRE_DEPARTAMENTO</strong></td>
            <td><i>Varchar</i></td>
            <td><i>65</i></td>
            <td><i>Nombre del departamento.</i></td>
        </tr>
    </tbody>
</table>

##### Dimensión banco: DIM_BANCO

La dimensión DIM_BANCO está conformada por:
<table>
    <thead>
        <tr>
            <th>Campo</th>
            <th>Tipo de Dato</th>
            <th>Longitud</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>BANCO_KEY</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave para la dimnesión banco.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>COD_BANCO</strong></td>
            <td><i>Varchar</i></td>
            <td><i>7</i></td>
            <td><i>Código del banco.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>NOMBRE_BANCO</strong></td>
            <td><i>Varchar</i></td>
            <td><i>65</i></td>
            <td><i>Nombre del banco.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>COMISION</strong></td>
            <td><i>float</i></td>
            <td><i>-</i></td>
            <td><i>Comisión según sea el banco.</i></td>
        </tr>
    </tbody>
</table>

##### Dimensión periodo: DIM_PERIODO

La dimensión DIM_PERIODO está conformada por:
<table>
    <thead>
        <tr>
            <th>Campo</th>
            <th>Tipo de Dato</th>
            <th>Longitud</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>PERIODO_KEY</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave para la dimnesión producto.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>FECHA</strong></td>
            <td><i>date</i></td>
            <td><i>-</i></td>
            <td><i>Fecha de la venta.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>ANIO</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Año de la venta.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>MES</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Mes de la venta.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>DIA</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Día de la venta.</i></td>
        </tr>
    </tbody>
</table>

##### Dimensión documento: DIM_DOCUMENTO

La dimensión DIM_DOCUMENTO está conformada por:
<table>
    <thead>
        <tr>
            <th>Campo</th>
            <th>Tipo de Dato</th>
            <th>Longitud</th>
            <th>Descripción</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="center"><strong>DOCUMENTO_KEY</strong></td>
            <td><i>int</i></td>
            <td><i>-</i></td>
            <td><i>Llave para la dimnesión documento.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>COD_TIPO_DOCUMENTO</strong></td>
            <td><i>Varchar</i></td>
            <td><i>8</i></td>
            <td><i>Código del tipo de documento.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>NOMBRE_TIPO_DOCUMENTO</strong></td>
            <td><i>Varchar</i></td>
            <td><i>26</i></td>
            <td><i>Nombre del tipo de documento.</i></td>
        </tr>
        <tr>
            <td align="center"><strong>INDICADOR</strong></td>
            <td><i>Varchar</i></td>
            <td><i>1</i></td>
            <td><i>Indicador del tipo de documento.</i></td>
        </tr>
    </tbody>
</table>

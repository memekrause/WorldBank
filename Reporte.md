# Programas de Ayuda - Banco Mundial

## **Motivación** 
Cada año, el Banco Mundial tiene dos programas de ayuda para los cuales tiene que elegir dos países beneficiarios. En 2024, estos programas se enfocaron en la desigualdad de género. En particular, hay dos temas de relevancia en los cuales se quiere invertir: la participación de la mujer en el mercado laboral y la reducción de los embarazos adolescentes. 
A lo largo del ultimo siglo, las condiciones de vida mundiales han mejorado globalmente. La expectativa de vida fue aumentando considerablemente, tanto para hombres como para mujeres. En esta misma tendencia, la población femenina fue ingresando al mercado laboral y los embarazos adolescentes fueron disminuyendo. Sin embargo, la evolución no fue equitativa para todos los países del mundo y hay grandes poblaciones de mujeres que sufren de limitado acceso a la educación sexual y de acceso al mercado laboral.

## **Plan de Metricas**

![image](https://github.com/user-attachments/assets/9871dd8d-84cb-48ab-8d1f-c5b18a18ca40)

[LINK TEXT](https://docs.google.com/spreadsheets/d/1KKse8wZxhNrMu-JPZgr1lYuoET-QHiHb/edit?gid=750930123#gid=750930123)

## **Base de datos: primer acercamiento** 
Esta trabajo analiza la base de datos del Banco Mundial, bigquery-public-data.world_bank_health_population, que contiene muchos indicadores de todos los paises a lo largo de los años 1990-2019.

Haciendo un primer acercamiento a la base de datos, me di cuenta que habian muchos datos nulos. Es decir, muchos indicadores que no fueron respondidos por muchos paises o en muchos años. Entonces el primer paso que hice fue analizar en promedio cuales fueron los indicadores que tenían más respuestas a lo largo de los años (contando los países que habían respondido). Agrupe los datos por indicator_name y year para contar cuántas respuestas no nulas hay para cada combinación de indicador y año; y luego, conte cuántos valores no nulos existen para cada indicator_name en cada año. Despues, calcule el promedio de respuestas anuales (yearly_response) para cada indicador. 

![image](https://github.com/user-attachments/assets/5615214b-c755-4aa9-8ebd-d80ddd1b2dc5)

Teniendo una lista amplia de "buenos" indicadores, elegí los que mas me interesaban, limitandome a indicadores que tuvieran un promedio anual de respuestas de 229 o más.
Tome los indicadores que podrían serme utiles para hacer un estudio de genero, y arme mi Silver Layer.Tuve que traspolar mis registros de indicator_name como una nueva columna por indicador, usando MAX(CASE que elige el valor mas alto por cada registro en las combinaciones unicas de pais-año-indicador.
Por otro lado, decidí hacer un inner join para incorporar la variable region de otra la tabla de la base worldbank-434116.BronzeLayer.country_summary. 

![image](https://github.com/user-attachments/assets/ffcb47cf-e3ab-4522-995b-8b0780c39461)


## **Power Query**

La tabla de resultados obtenida con la query anterior fue la que guarde como mi Silver Layer y conecte a Power BI. 

En mi capa Gold, la tabla tuvo modificaciones que realice en PowerQuery. Cambie los nombres de los indicadores (es decir, de las columnas) al español.
Elimine columnas: mortality ratio, ya que todos sus registros eran 1; basic_sanitation porque la información no era relevante para el estudio y fertility_rate, life_expectancy_total, life_expactancy_male porque no fueron utilizada en este caso. 

Genere un ID de pais, y una tabla de dimension geografia que este conectado por ese ID y tuviera los campos de descripcion de pais y region.

![image](https://github.com/user-attachments/assets/479a976f-b375-4e32-b321-b2c820debb94)
[LINK TEXT](https://dbdiagram.io/d/WorldBank-66f9d9303430cb846c00033f)


## **PowerBI**

[LINK TEXT](https://app.powerbi.com/groups/me/reports/2ca79f20-1803-4c48-a5e0-982da3db7042/d501943250208d7e3955?experience=power-bi)

Una vez, en el ambiente de PowerBI, mis visualizaciones se enforcaron en encontrar los dos paises que estaban más "necesitados" de ayuda financiera para combatir la alta fertilidad adolescente y la baja participación femenina en el mercado laboral. 

Primero, se analizo el promedio por región; eligiendo dos regiones diferentes en las cuales hacer "zoom", por cada problematica. En particular, Africa Sub Sahariana y Medio Oriente. Decido trabajar con los datos de los ultimos años 2010-2020 de la encuenta para tener los datos mas actualizados y también porque son los años donde hay más registros (respuestas por indicador).

## **Elección de beneficiario: PRIMER PROGRAMA**
Para fertilidad adolescente, se analiza el caso de Africa Subsahariana. Se decide recortar los numeros de países en los cuales nos enfocaremos, siendo 150+ el numero corte de nacimientos por cada 1000 mujeres, para poder analizar con más claridad. Se decide que sea 150 porque se ve una clara distancia entre los países con mas de 150 y los que tienen menos que 150. 

Para los países Subsaharianos con la mayor fertilidad adolescente entre 2010-2020, también se quiere evaluar el nivel de gastos en salud que hace el gobierno, ya que se ha estudiado que la atención primaria es la mayor fuente de ayuda para combatir la fertilidad adolescente.

Dentro de los 6 países de la región con mayor alta fertilidad adolescente, se ha elegido a Angola como beneficiario del Programa "Un 2040 sin embarazos adolescentes: Inversión en la atención primaria a mujeres de 15-19 años y en la expansión de Educación sexual integral". Dado que el Programa de ayuda se apoya en el sistema de salud, se ha priorizado a Angola por su mayor gasto público en salud. En Angola al 2019, hay 145 nacimientos por cada 1.000 mujeres de entre 15-19 años. El programa de inversión proyecta disminuirlo a 120 para 2040.

## **Elección de beneficiario: SEGUNDO PROGRAMA** 
Para analizar la participación femenina se elige el caso de Medio Oriente y Africa del Norte. Esta región es la que cuenta con el menor porcentaje de mujeres que trabajan y ademas con el más alto indice de discriminación laboral, medida que creé con el fin de analizar más a fondo la cuestión. 

Llamo Discriminación laboral a la medida de ratio entre desempleo femenino sobre desempleo masculino, donde los mayores valores de discriminación se dan por un alto desempleo femenino y un bajo desempleo masculino. Un país con Discriminación igual a 1 es un país donde no se discrimina por géneros. Los país con valores mayores a 1 son más discriminatorios hacia la mujer, cuanto mayor sea el numero.  Los países cuyo valor de discriminación es menor a 1, significaría discriminación hacia el hombre. Obviamente se entiende que hay una relación estrecha entre participación femenina en el mercado laboral y discriminación. Sin embargo, nuestra medida de discriminación, al incluir al desempleo tiene en consideración aquellos casos donde existe el deseo de trabajar pero no se consigue. Mientras que en el porcentaje de población activa femenina, no necesariamente hay una deseo de trabajar; sino que existe la posibilidad de salida voluntaria del mercado laboral.

Se ha seleccionado a Qatar como beneficiario del Programa "El trabajo como fuente de empoderamiento: concientización y educación sobre la igualdad de derechos en el acceso al trabajo para mujeres". En Qatar en 2019, las mujeres tienen 6 veces más posibilidades de estar desempleado que los varones. Con la colaboración de escuelas y organizaciones locales, el programa buscará difundir y educar en la importancia de la obtención de trabajo y la propia manutención como fuente de empoderamiento femenino. Para 2040, se buscará reducir la discriminación a 5 ptos, esperando encontrar menores valores para aquellas mujeres jóvenes, que recientemente hayan entrado a la edad trabajadora.









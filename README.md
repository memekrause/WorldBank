# Programas de Ayuda - Banco Mundial

**CONTEXTO** Cada año, el Banco Mundial tiene dos programas de ayuda para los cuales tiene que elegir dos píses beneificiarios. En 2024, estos programas se enfocaron en la desigualdad de genero. En particular, hay dos temas de relevancia en los cuales se quiere invertir: la participación de la mujer en el mercado laboral y la reducción de los embarazos adolescentes. 

**ANALISIS DE BASE DE DATOS** Esta trabajo analiza la base de datos del Banco Mundial, bigquery-public-data.world_bank_health_population, que contiene muchos indicadores de todos los paises a lo largo de los años 1990-2019.

Haciendo un primer acercamiento a la base de datos, me di cuenta que habian muchos datos nulos. Es decir, muchos indicadores que no fueron respondidos por muchos paises o en muchos años. Entonces el primer paso que hice fue analizar en promedio cuales fueron los indicadores que tenían más respuestas a lo largo de los años, contando los países que habían respondido. Agrupe los datos por indicator_name y year para contar cuántas respuestas no nulas hay para cada combinación de indicador y año; y luego, conte cuántos valores no nulos existen para cada indicator_name en cada año. Despues, calcule el promedio de respuestas anuales (yearly_response) para cada indicador. 

WITH yearly_responses AS (
  SELECT
      indicator_name,
      year,
      COUNT(value) as yearly_response
  FROM
      worldbank-434116.BronzeLayer.health_nutrition_population sd
  WHERE value IS NOT NULL
  GROUP BY
      indicator_name, year
),
average_responses AS (
  SELECT
      indicator_name,
      AVG(yearly_response) AS avg_yearly_responses
  FROM
      yearly_responses
  GROUP BY
      indicator_name
)
SELECT
    indicator_name,
    avg_yearly_responses
FROM
    average_responses
ORDER BY 
    avg_yearly_responses DESC
LIMIT 200;![image](https://github.com/user-attachments/assets/b68cc003-cf6a-4790-a63c-27fdb7e565f5)

Teniendo una lista amplia de "buenos" indicadores, elegí los que mas me interesaban, limitandome a indicadores que tuvieran un promedio anual de respuestas de 229 o más.
Tome los indicadores que podrían serme utiles para hacer un estudio de genero, y arme mi Silver Layer.Tuve que traspolar mis registros de indicator_name como una nueva columna por indicador, usando MAX(CASE que elige el valor mas alto por cada registro en las combinaciones unicas de pais-año-indicador.
Por otro lado, decidí hacer un inner join para incorporar la variable region de otra trabla worldbank-434116.BronzeLayer.country_summary. 

SELECT
    h.country_name,
    h.year,
    MAX(CASE WHEN h.indicator_name = 'Unemployment, total (% of total labor force)' THEN h.value END) AS unemployment_total,
    MAX(CASE WHEN h.indicator_name = 'Unemployment, female (% of female labor force)' THEN h.value END) AS unemployment_female,
    MAX(CASE WHEN h.indicator_name = 'Unemployment, male (% of male labor force)' THEN h.value END) AS unemployment_male,
    MAX(CASE WHEN h.indicator_name = 'Fertility rate, total (births per woman)' THEN h.value END) AS fertility_rate,
    MAX(CASE WHEN h.indicator_name = 'People using at least basic sanitation services (% of population)' THEN h.value END) AS basic_sanitation_services,
    MAX(CASE WHEN h.indicator_name = 'Domestic general government health expenditure (% of current health expenditure)' THEN h.value END) AS gov_health_expenditure,
    MAX(CASE WHEN h.indicator_name = 'Adolescent fertility rate (births per 1,000 women ages 15-19)' THEN h.value END) AS adolescent_fertility_rate,
    MAX(CASE WHEN h.indicator_name = 'Labor force, female (% of total labor force)' THEN h.value END) AS female_labor_force,
    MAX(CASE WHEN h.indicator_name = 'Maternal mortality ratio (modeled estimate, per 100,000 live births)' THEN h.value END) AS maternal_mortality_ratio,
    MAX(CASE WHEN h.indicator_name = 'Life expectancy at birth, total (years)' THEN h.value END) AS life_expectancy_total,
    MAX(CASE WHEN h.indicator_name = 'Life expectancy at birth, male (years)' THEN h.value END) AS life_expectancy_male,
    MAX(CASE WHEN h.indicator_name = 'Life expectancy at birth, female (years)' THEN h.value END) AS life_expectancy_female,
    c.region
FROM
    worldbank-434116.BronzeLayer.health_nutrition_population h
INNER JOIN
    worldbank-434116.BronzeLayer.country_summary c ON h.country_code = c.country_code
WHERE
    h.indicator_name IN ('Unemployment, total (% of total labor force)','Unemployment, female (% of female labor force)', 'Unemployment, male (% of male labor force)', 'Fertility rate, total (births per woman)', 'People using at least basic sanitation services (% of population)', 'Domestic general government health expenditure (% of current health expenditure)', 'Adolescent fertility rate (births per 1,000 women ages 15-19)', 'Labor force, female (% of total labor force)', 'Maternal mortality ratio (modeled estimate, per 100,000 live births)','Life expectancy at birth, total (years)', 'Life expectancy at birth, male (years)', 'Life expectancy at birth, female (years)')
    AND h.value IS NOT NULL
    AND c.region IS NOT NULL
GROUP BY
    h.country_name, h.year, c.region
ORDER BY
    h.year;![image](https://github.com/user-attachments/assets/ffcb47cf-e3ab-4522-995b-8b0780c39461)

Esta tabla de resultados fue la que guarde como mi Silver Layer y conecte a Power BI. En mi capa Gold, la tabla solo tuvo dos modificaciones que realice en PowerQuery: elimine la columna de mortality ratio, ya que todos sus registros eran 1; y elimine la columna de basic_sanitation porque la información no era relevante para el estudio. 


**
INSERTAR DIAGRAMA**

Una vez, en el ambiente de PowerBI, mis visualizaciones se enforcaron en encontrar los dos paises que estaban más "necesitados" de ayuda financiera para combatir la alta fertilidad adolescente y la baja participación femenina en el mercado laboral. 
A lo largo del ultimo siglo, las condiciones de vida mundiales han mejorado globalmente. La expectativa de vida fue aumentando considerablemente, tanto para hombres como para mujeres. En esta misma tendencia, la población femenina fue ingresando al mercado laboral y los embarazos adolescentes fueron disminuyendo. Sin embargo, la evolución no fue equitativa para todos los países del mundo y hay grandes poblaciones de mujeres que sufren de limitado acceso a la educación sexual y de acceso al mercado laboral.

Primero, se analizo el promedio por región; eligiendo dos regiones diferentes en las cuales hacer "zoom", por cada problematica.

**PRIMER BENEFICIARIO** Para fertilidad adolescente, se analiza el caso de Africa Subsahariana. Dentro de los 6 países de la región con mayor alta fertilidad adolescente, se ha elegido a Angola como beneficiario del Programa "Un 2040 sin embarazos adolescentes: Inversión en la atención primaria a mujeres de 15-19 años y en la expansión de Educación sexual integral". Dado que el Programa de ayuda se apoya en el sistema de salud, se ha priorizado a Angola por su mayor gasto público en salud. En Angola al 2019, hay 145 nacimientos por cada 1.000 mujeres de entre 15-19 años, que se proyecta disminuir a 120 para 2040.


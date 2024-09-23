# Programas de Ayuda - Banco Mundial

CONTEXTO Cada año, el Banco Mundial tiene dos programas de ayuda para los cuales tiene que elegir dos píses beneificiarios. En 2024, estos programas se enfocaron en la desigualdad de genero. En particular, hay dos temas de relevancia en los cuales se quiere invertir: la participación de la mujer en el mercado laboral y la reducción de los embarazos adolescentes. 

ANALISIS DE BASE DE DATOS Esta trabajo analiza la base de datos del Banco Mundial, bigquery-public-data.world_bank_health_population, que contiene muchos indicadores de todos los paises a lo largo de los años.
Haciendo un primer acercamiento a la base de datos, me di cuenta que habian muchos datos nulos. Es decir, muchos indicadores que no fueron respondidos por muchos paises o en muchos años. Entonces el primer paso que hice fue analizar en promedio cuales fueron los indicadores que tenían más respuestas a lo largo de los años, contando los países que habían respondido. Agrupe los datos por indicator_name y year para contar cuántas respuestas no nulas hay para cada combinación de indicador y año; y luego, conte cuántos valores no nulos existen para cada indicator_name en cada

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



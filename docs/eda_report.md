> **ANÁLISIS DEL SECTOR TIC**
>
> España y Europa

**EDA --- Análisis Exploratorio de Datos**

Documentación técnica del proceso exploratorio sobre los datasets del
proyecto

> **1. Introducción y Contexto del Proyecto**
>
> El presente documento recoge los hallazgos del análisis exploratorio
> de datos (EDA) realizado sobre el conjunto de fuentes utilizadas en el
> proyecto de análisis del sector de Información y Comunicación (TIC) a
> nivel nacional (España) y europeo.
>
> El objetivo del proyecto es caracterizar la estructura del sector TIC,
> analizar los niveles salariales, estudiar la distribución demográfica
> de los profesionales y mapear las habilidades técnicas más demandadas
> en el mercado laboral.

**Fuentes de datos utilizadas**

> Los datos provienen de cuatro grandes bloques:

-   Kaggle (ofertas de empleo y salarios): datasets de job posts de
    ciencia de datos de 2025 y un dataset consolidado de salarios
    2020--2024 del sector tech.

-   Dataset propio de horquillas salariales: recoge rangos de salario
    mínimo y máximo por rol y país europeo.

-   Eurostat (ISOC): múltiples indicadores estadísticos oficiales sobre
    el empleo TIC en Europa, con desagregaciones por edad, sexo, nivel
    educativo, sector económico y tamaño de empresa.

> **2. Inventario de Datasets**
>
> A continuación se describe el conjunto completo de fuentes analizadas
> durante el EDA, con sus dimensiones originales y una descripción del
> contenido.

  ------------------------------------------------------------------------------------------------------------
  **ID**   **Nombre del fichero**            **Filas**   **Cols**   **Fuente**         **Contenido**
  -------- --------------------------------- ----------- ---------- ------------------ -----------------------
  DF1      data_science_job_posts_2025.csv   944         13         Kaggle             Ofertas de empleo en
                                                                                       ciencia de datos
                                                                                       scraped de LinkedIn.
                                                                                       Incluye puesto,
                                                                                       empresa, skills,
                                                                                       salario y ubicación.

  DF2      Distribución ofertas tech por     4.533       19         Eurostat           Distribución trimestral
           tipo de profesión.csv                                    ISOC_SK_OJA2       de ofertas de empleo
                                                                                       online tech por
                                                                                       ocupación ISCO-08. 33
                                                                                       países europeos.

  DF3      Empresas por tamaño que contratan 3.253       23         Eurostat           \% de empresas que
           ICT.csv                                                  ISOC_SKE_ITSPE     emplean especialistas
                                                                                       TIC, segmentado por
                                                                                       tamaño de empresa y
                                                                                       sector NACE.

  DF4      eu_data_jobs_salaries.csv         120         5          Dataset propio     Horquillas salariales
                                                                                       (min/max en EUR) para
                                                                                       roles tech en países
                                                                                       europeos.

  DF5      Número/proporción de ofertas      7.239       17         Eurostat           Porcentaje trimestral
           ICT.csv                                                  ISOC_SK_OJA1       de ofertas de empleo
                                                                                       online dirigidas a
                                                                                       especialistas TIC. 33
                                                                                       países.

  DF6      Porcentaje especialistas por      2.752       19         Eurostat           \% de especialistas TIC
           edad.csv                                                 ISOC_SKS_ITSPA     empleados por tramo de
                                                                                       edad (15-34, 35-74) por
                                                                                       país y año.

  DF7      Porcentaje especialistas por      3.146       19         Eurostat           \% de especialistas TIC
           nivel educativo.csv                                      ISOC_SKS_ITSPE     por nivel educativo
                                                                                       (ISCED 2011) por país y
                                                                                       año.

  DF8      Porcentaje especialistas por      40.900      19         Eurostat           \% de especialistas TIC
           sector.csv                                               ISOC_SKS_ITSPN2    empleados por sector
                                                                                       económico (NACE Rev.
                                                                                       2). Mayor volumen del
                                                                                       proyecto.

  DF9      Porcentaje especialistas por      2.752       19         Eurostat           \% de especialistas TIC
           sexo.csv                                                 ISOC_SKS_ITSPS     por sexo
                                                                                       (Femenino/Masculino)
                                                                                       por país y año.

  DF10     Porcentaje profesionales TIC      1.376       9          Eurostat           \% de especialistas TIC
           sobre empleo total.csv                                   ISOC_SKS_ITSPT     sobre el empleo total
                                                                                       por país europeo y año.
                                                                                       Formato diferencial.

  DF11     Sectores que contratan empleados  23.180      23         Eurostat           \% de empresas con 10+
           ICT.csv                                                  ISOC_SKE_ITSPEN2   empleados que contratan
                                                                                       TIC, por sector NACE y
                                                                                       país.

  DF12     salaries_concat.csv               \~5.000+    11         Kaggle (concat)    Dataset concatenado de
                                                                                       salarios tech
                                                                                       2020--2024. Incluye
                                                                                       experiencia, tipo
                                                                                       empleo, salario USD y
                                                                                       ubicación.
  ------------------------------------------------------------------------------------------------------------

> **3. Metodología del EDA**
>
> El análisis exploratorio se realizó mediante una función centralizada
> (eda()) diseñada para estandarizar y sistematizar la revisión de cada
> dataset. La función aplica los mismos controles a todos los datasets,
> garantizando comparabilidad en el diagnóstico.

**Controles aplicados**

> La función eda() articula el análisis en cinco bloques:

  ---------------- -------------------------------------------------------
  **Bloque**       **Descripción**

  **1. Visión      Head, dimensiones, info() y conteo de filas duplicadas.
  general**        

  **2. Calidad de  Tabla resumen por columna: tipo, nulos absolutos y
  columnas**       porcentuales, cardinalidad. Clasificación de columnas
                   por umbral de nulos (bajo ≤5%, moderado 5--20%, alto
                   \>20%).

  **3. Análisis de Para categóricas: distribución de frecuencias
  nulos**          incluyendo NaN. Para numéricas: porcentaje de nulos,
                   histograma con KDE.

  **4.             Describe() para numéricas y categóricas. Flags de
  Estadísticas     valores negativos y varianza cero.
  descriptivas**   

  **5. Outliers**  Detección por rango intercuartílico (IQR × 1,5) con
                   boxplots para columnas afectadas.
  ---------------- -------------------------------------------------------

> **4. Hallazgos Generales por Tipología de Dataset**
>
> El EDA identificó patrones comunes y particularidades por tipo de
> fuente. A continuación se presentan los hallazgos agrupados por
> familia de datos.
>
> **4.1 Datasets de Ofertas de Empleo y Skills (Kaggle)**

**Estructura y tipado**

> El dataset de ofertas de empleo (DF1, 944 × 13) está compuesto
> íntegramente por variables de tipo object. Todas las variables
> numéricas asociadas al contexto empresarial (tamaño, revenue, salario)
> se encontraron almacenadas como texto, lo que evidencia la necesidad
> de parseo y normalización posterior.

**Calidad y valores nulos**

> La variable status presentó el volumen más significativo de nulos
> (27,1%), atribuible al método de scraping: no todas las ofertas
> publican la modalidad de trabajo. El resto de columnas críticas para
> el análisis (job_title, industry, skills) tienen cobertura casi
> completa o total.

  -----------------------------------------------------------------------
  **Nota** La columna skills contiene 201 observaciones con lista vacía
  (\[\]), lo que representa registros válidos estructuralmente pero sin
  información de habilidades. Estas no son nulos técnicos sino registros
  donde la empresa no especificó skills.

  -----------------------------------------------------------------------

**Distribución de categorías**

> La variable job_title está muy concentrada: el 90,7% de las 941
> registros válidos corresponden a \'Data Scientist\'. Los roles
> \'Machine Learning Engineer\' (8,5%), \'Data Engineer\' y \'Data
> Analyst\' son minoritarios. En cuanto a seniority_level, el perfil
> senior domina con un 66,7% de las ofertas.
>
> La variable revenue presenta mezcla de categorías textuales (Public,
> Private, Education) y valores numéricos con símbolo de divisa, lo que
> confirma inconsistencia semántica que requiere tratamiento
> diferenciado.

  -----------------------------------------------------------------------
  **Hallazgo clave** La columna salary almacena rangos en formato texto
  (ej: \'€100,472 - €200,938\') con 896 valores únicos sobre 944
  registros.

  -----------------------------------------------------------------------

> **4.2 Dataset de Salarios (salaries_concat.csv)**

**Estructura y alcance temporal**

> El dataset consolida registros de salarios del sector tech entre 2020
> y 2024. Incluye variables sobre nivel de experiencia, tipo de
> contrato, puesto de trabajo, salario en divisa local y convertido a
> USD, así como información geográfica de empresa y empleado.

**Inconsistencias detectadas**

> Se detectaron inconsistencias de codificación en variables como
> experience_level y employment_type, donde conviven tanto abreviaciones
> estándar (SE, MI, FT) como textos completos equivalentes (Senior, Mid,
> Full-Time). La variable remote_ratio presentó valores nulos puntuales.
>
> Los países en company_location y employee_residence mezclaban nombres
> completos con códigos ISO de 2 letras, lo que impide comparaciones
> directas sin homogeneización previa.
>
> **4.3 Dataset de Horquillas Salariales Europeas
> (eu_data_jobs_salaries.csv)**
>
> Este es el dataset más compacto del proyecto (120 × 5) y el más limpio
> estructuralmente. Contiene exclusivamente rangos salariales
> (salary_min, salary_max) en EUR para distintos roles tech y países
> europeos, sin valores nulos. La moneda es uniforme (EUR) en todos los
> registros.

  -----------------------------------------------------------------------
  **Nota** Al ser un dataset de referencia construido, no
  presenta problemas de calidad típicos de scraping o descarga
  automatizada. Sirve como tabla de lookup para contextualizar salarios
  del sector.

  -----------------------------------------------------------------------

> **4.4 Datasets Eurostat (ISOC)**

**Patrón estructural común**

> Todos los datasets de Eurostat comparten una estructura tabular
> homogénea heredada del formato de descarga SDMX/CSV. Incluyen columnas
> de metadatos administrativos (STRUCTURE, STRUCTURE_ID, STRUCTURE_NAME,
> freq, Time frequency) con valor constante en todo el dataset, y
> columnas de flag (OBS_FLAG, CONF_STATUS, Observation status,
> Confidentiality status) con un 100% de nulos en la mayoría de los
> casos.
>
> Esta arquitectura responde al formato de exportación estándar de
> Eurostat, no a errores de datos, y debe ser tratada sistemáticamente
> en el proceso de limpieza.

**Variable principal de análisis: OBS_VALUE**

> En todos los datasets Eurostat, la variable OBS_VALUE contiene el
> indicador cuantitativo del registro (porcentaje de especialistas,
> número de ofertas, etc.). Solo DF3 presentó 17 nulos en esta variable
> (0,5% del total), atribuibles a países con datos no disponibles para
> ciertos años o segmentos.

**Cobertura geográfica y temporal**

> Los datasets de Eurostat cubren entre 27 y 33 países europeos (UE-27
> más países adicionales como Suiza, Noruega e Islandia según el
> indicador). La cobertura temporal varía: los indicadores anuales
> comienzan generalmente en 2004 y llegan hasta 2023, mientras que los
> trimestrales (DF2, DF5) arrancan en 2019-Q4.
>
> El dataset de mayor volumen es DF8 (especialistas por sector NACE Rev.
> 2), con 40.900 filas, explicado por la combinación de país × sector
> económico × año.

  -----------------------------------------------------------------------
  **Hallazgo clave** Los datasets DF2 y DF5 (frecuencia trimestral)
  tienen la columna \'Time\' completamente vacía (100% nulos), mientras
  que el período se registra correctamente en \'TIME_PERIOD\'. Se trata
  de una redundancia del formato de exportación, no de pérdida de
  información.

  -----------------------------------------------------------------------

> **5. Calidad de Datos --- Resumen por Dataset**
>
> La siguiente tabla consolida los principales flags de calidad
> identificados durante el EDA. Solo se recogen variables con nivel de
> nulos moderado o alto, o aquellas con anomalías estructurales
> relevantes.

  -------------------------------------------------------------------------------------
  **Dataset**   **Variable**      **%       **Nivel**      **Observación**
                                  Nulos**                  
  ------------- ----------------- --------- -------------- ----------------------------
  DF1 (Job      status            27,1%     **ALTO**       Modalidad de trabajo no
  Posts)                                                   siempre publicada en la
                                                           oferta. Representa ausencia
                                                           de información en origen.

  DF1 (Job      seniority_level   6,4%      **MODERADO**   Nivel de seniority no
  Posts)                                                   especificado en la oferta
                                                           original.

  DF1 (Job      ownership         5,0%      **BAJO**       Tipo de propiedad de la
  Posts)                                                   empresa no disponible.

  DF1 (Job      revenue           1,6%      **BAJO**       Mezcla de categorías
  Posts)                                                   textuales y valores
                                                           numéricos. Inconsistencia
                                                           semántica más allá de los
                                                           nulos.

  DF2--DF9      Columnas          100%      **ALTO**       Columnas de metadatos y
  (Eurostat)    flag/meta                                  flags del formato de
                                                           exportación Eurostat, vacías
                                                           de forma sistemática.

  DF3 (Emp.     obs_value         0,5%      **BAJO**       17 filas sin valor de
  tamaño)                                                  indicador. Países con datos
                                                           no disponibles para
                                                           determinados años o
                                                           segmentos.

  DF12          remote_ratio      \<1%      **BAJO**       Nulos puntuales en la
  (Salarios)                                               proporción de teletrabajo.
  -------------------------------------------------------------------------------------

> **6. Análisis de Duplicados**
>
> El análisis de duplicados no detectó filas duplicadas en ninguno de
> los doce datasets analizados. Este resultado es coherente con la
> naturaleza de las fuentes:

-   Los datasets de Eurostat tienen combinaciones únicas de país ×
    período × dimensión de segmentación.

-   El dataset de ofertas de empleo tiene como identificador implícito
    la empresa y el puesto, y fue anonimizado previamente.

-   El dataset de salarios concatenado puede contener registros
    similares entre años, pero la combinación de variables (año, puesto,
    país, salario) actúa como clave natural.

> **7. Análisis de Outliers**
>
> El análisis de outliers mediante el método IQR (rango intercuartílico
> × 1,5) se aplicó únicamente a las variables numéricas de cada dataset.
> Los resultados más relevantes son:

**Datasets Eurostat (OBS_VALUE)**

> La variable OBS_VALUE en los datasets de porcentaje (DF6--DF9)
> presentó distribuciones asimétricas con colas en valores altos. Este
> comportamiento es esperable: la distribución del empleo TIC no es
> uniforme entre países, y los outliers reflejan diferencias
> estructurales reales entre economías (ej. Países Bajos, Suecia con
> cifras muy superiores a la media).
>
> No se identificaron outliers que pudiesen atribuirse a errores de
> entrada de datos. Los valores extremos se mantienen como observaciones
> válidas de interés analítico.

**Dataset de Salarios (DF12)**

> Las variables salary y salary_in_usd presentaron outliers por la parte
> alta, con registros de salarios muy elevados principalmente asociados
> a roles de liderazgo en grandes empresas tecnológicas de EE. UU. No se
> tratan como errores sino como segmento de mercado diferenciado.
>
> **8. Conclusiones y Decisiones Derivadas**
>
> El EDA confirmó que los datos son utilizables para los objetivos del
> proyecto, con particularidades de calidad manejables. Las principales
> conclusiones que orientan la fase de limpieza son:

  -----------------------------------------------------------------------
  **1** Los datasets de Eurostat tienen una estructura muy homogénea y
  predecible, lo que permite aplicar un pipeline de limpieza genérico. El
  mayor trabajo reside en la selección de columnas relevantes y el
  tratamiento de metadatos vacíos.

  -----------------------------------------------------------------------

  -----------------------------------------------------------------------
  **2** El dataset de ofertas de empleo (DF1) concentra la mayor
  complejidad de limpieza: variables numéricas almacenadas como texto,
  campos multivaluados (location, skills), fechas relativas y
  combinaciones de categorías y valores numéricos en la misma columna
  (revenue, company_size).

  -----------------------------------------------------------------------

  -----------------------------------------------------------------------
  **3** La homogeneización final de nombres de país, códigos de ocupación
  y categorías entre datasets se delega a Power Query, dado que es la
  herramienta más adecuada para transformaciones cruzadas entre fuentes
  de diferente naturaleza.

  -----------------------------------------------------------------------

  -----------------------------------------------------------------------
  **4** Los nulos en datasets Eurostat son en su mayoría estructurales
  (columnas del formato de exportación) y no informativos. No requieren
  imputación sino eliminación.

  -----------------------------------------------------------------------

  -----------------------------------------------------------------------
  **5** Las variables de skills y salary del dataset de Kaggle requieren
  extracción y normalización específica para ser explotables en el
  análisis. Estas transformaciones se documentan en el notebook de data
  cleaning.

  -----------------------------------------------------------------------

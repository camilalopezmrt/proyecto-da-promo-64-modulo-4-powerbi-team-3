> **ANÁLISIS DEL SECTOR TIC**
>
> España y Europa

**Data Cleaning --- Limpieza y Transformación**

Justificación de decisiones de limpieza, transformación y lo que se dejó
para fases posteriores

> **1. Introducción y Alcance**
>
> Este documento describe las decisiones de limpieza y transformación
> aplicadas a los datasets del proyecto de análisis del sector TIC, así
> como aquellas que se descartaron conscientemente o se delegaron a
> fases posteriores.
>
> El proceso de limpieza tiene como objetivo garantizar un nivel mínimo
> de calidad y consistencia que permita trabajar con los datos de forma
> fiable, sin entrar en transformaciones profundas que pertenecen al
> proceso de homogeneización entre fuentes (delegado a Power Query).

**Principios de la limpieza**

-   Intervenir lo mínimo necesario: solo se modifica lo que impide el
    análisis o introduce errores evidentes.

-   Documentar cada decisión: tanto lo que se hace como lo que no se
    hace y por qué.

-   Preservar los datos originales: las transformaciones se aplican
    sobre copias y el raw permanece inalterado en data/raw/.

-   Los datasets limpios se exportan a data/processed/ con nombres
    descriptivos estandarizados.

  -----------------------------------------------------------------------
  **Alcance** Este notebook trata los 12 datasets del proyecto de forma
  cohesionada. Las transformaciones específicas de homogeneización
  cruzada entre fuentes (valores de países, categorías de ocupación,
  unidades salariales) se realizarán en Power Query, donde la gestión de
  múltiples tablas es más eficiente.

  -----------------------------------------------------------------------

> **2. Pipeline de Limpieza**
>
> El proceso de limpieza se articuló en ocho pasos secuenciales. Cada
> paso se implementó como una función reutilizable que se aplicó
> iterativamente sobre todos los datasets donde era aplicable.

  -----------------------------------------------------------------------------------------
  **Paso**   **Transformación**   **Alcance**   **Justificación**     **Qué NO se hizo**
  ---------- -------------------- ------------- --------------------- ---------------------
  **1**      Normalización de     Todos los     Garantizar            No se renombraron
             nombres de columnas  datasets      consistencia en el    columnas.
                                                caracteres            
                                                especiales.           

  **2**      Eliminación de       Eurostat DF2, Columnas vacías son   No se eliminaron
             columnas             DF3, DF5--DF9 artefactos del        columnas con nulos
             completamente vacías               formato de            parciales sin
             (100% nulos)                       exportación Eurostat  análisis previo del
                                                (SDMX) sin            impacto.
                                                información           
                                                recuperable.          

  **3**      Eliminación de       Eurostat      Variables constantes  Se preservaron
             columnas constantes  DF2--DF11     como \'structure\',   excepciones como
             (un único valor)                   \'freq\' o            \'unit_of_measure\'
                                                \'time_frequency\' no aunque sea constante,
                                                aportan variabilidad  porque contextualiza
                                                analítica.            el indicador.

  **4**      Eliminación de       Eurostat      Códigos técnicos como No se eliminaron los
             columnas fuera de    DF2--DF11     isco08, isced11,      códigos si eran la
             alcance                            nace_r2 o unit        única representación
                                                (abrev.) coexisten    de la dimensión en
                                                con sus descripciones ese dataset.
                                                largas. Se conserva   
                                                la descripción y se   
                                                elimina el código     
                                                cuando no se usa en   
                                                el análisis.          

  **5**      Tratamiento de       Todos los     Se eliminan filas con No se imputaron
             valores nulos        datasets      obs_value nulo (no    valores numéricos.
                                                hay información       Los nulos en
                                                útil). En categóricas variables numéricas
                                                se imputa \'Unknown\' secundarias se
                                                para preservar el     mantienen para no
                                                registro.             introducir sesgo.

  **6**      Preparación del      DF1 (job      Requiere tratamiento  No se normalizaron
             dataset de skills    posts)        específico: filtrar   los nombres de skills
                                                registros sin skills, (ej. \'ml\' vs
                                                seleccionar columnas  \'machine
                                                relevantes y explotar learning\'). Queda
                                                la lista de skills a  para análisis de
                                                formato largo (una    texto posterior.
                                                skill por fila).      

  **7**      Limpieza de          Todos los     Eliminación de        No se unificaron
             variables            datasets      espacios y aplicación categorías
             categóricas                        de formato homogéneo  semánticamente
                                                (UPPER). Evita        equivalentes entre
                                                duplicados por        datasets. Eso se hace
                                                diferencias           en Power Query.
                                                tipográficas.         

  **8**      Agrupación y         DF1, DF2,     Se crean variables    No se agruparon
             simplificación de    DF7, DF8,     derivadas más         categorías en
             categorías           DF11          manejables:           datasets Eurostat
                                                company_size_group,   donde la granularidad
                                                occupation_short,     original es necesaria
                                                education_group,      para el análisis.
                                                sector_group.         
  -----------------------------------------------------------------------------------------

> **3. Paso 1 --- Normalización de Nombres de Columnas**

**Qué se hizo**

> Se aplicó una función de normalización a todos los datasets que
> transforma los nombres de columnas a snake_case estándar. Las
> transformaciones aplicadas son:

-   Eliminación de espacios al inicio y final.

-   Eliminación de acentos y caracteres Unicode (normalización NFKD →
    ASCII).

-   Conversión de CamelCase/PascalCase a snake_case.

-   Sustitución de caracteres no alfanuméricos por guión bajo.

-   Conversión a minúsculas y eliminación de guiones bajos duplicados o
    en los extremos.

-   Control de unicidad: en caso de colisión, se añade sufijo numérico.

**Efecto en los datasets**

> Los datasets de Kaggle (DF1, DF4, DF12) tenían nombres ya bien
> formados, por lo que el impacto fue mínimo. Los datasets de Eurostat
> son los más afectados, con columnas como \'International Standard
> Classification of Occupations 2008 (ISCO-08)\' que se normalizan a
> \'international_standard_classification_of_occupations_2008_isco_08\'.
> Estas columnas largas son candidatas a renombrarse en pasos
> posteriores.

**Qué no se hizo y por qué**

> No se renombraron columnas de Eurostat con nombres técnicos cortos
> como \'geo\' o \'unit\' en este paso. Aunque son mejorables
> semánticamente, la estandarización cruzada de nombres entre datasets
> se reserva para Power Query, donde se puede hacer con mayor control y
> visibilidad del mapeo completo.
>
> **4. Pasos 2, 3 y 4 --- Reducción de Columnas**
>
> Estos tres pasos trabajan conjuntamente para reducir la
> dimensionalidad de los datasets Eurostat, que incluyen múltiples
> columnas de metadatos sin valor analítico.

**Columnas vacías (100% nulos)**

> Los datasets de Eurostat con frecuencia trimestral (DF2, DF5) y parte
> de los anuales incluyen columnas vacías que corresponden a campos del
> esquema SDMX no disponibles en la exportación CSV. El EDA identificó
> hasta 6 columnas vacías en algunos datasets. Su eliminación es
> inmediata.

  -----------------------------------------------------------------------
  **Ejemplo representativo** DF2: se eliminan \'time\',
  \'observation_value\', \'obs_flag\',
  \'observation_status_flag_v2_structure\', \'conf_status\',
  \'confidentiality_status_flag\'. Ninguna de estas columnas tiene un
  solo valor no nulo en las 4.533 filas del dataset.

  -----------------------------------------------------------------------

**Columnas constantes**

> Los datasets de Eurostat incluyen columnas de metadatos del dataflow
> con el mismo valor en todas las filas: el identificador del conjunto
> de datos (STRUCTURE_ID), el nombre del dataflow (STRUCTURE_NAME), la
> frecuencia (freq) o la frecuencia temporal descriptiva
> (time_frequency). Estas columnas no aportan información diferencial y
> se eliminan.
>
> Se establecieron excepciones explícitas para preservar columnas como
> \'unit_of_measure\' o \'size_emp\' aunque sean constantes en ciertos
> datasets, porque contextualizan el significado del indicador y pueden
> ser útiles en cruce de tablas.

**Columnas fuera de alcance**

> Algunos datasets de Eurostat incluyen tanto el código ISCO-08,
> ISCED-11 o NACE Rev. 2 (ej. \'isco08\' = \'OC133\') como su
> descripción textual completa. Para el análisis se usa la descripción
> textual. El código, una vez eliminadas las columnas constantes, queda
> redundante y se elimina en este paso.
>
> También se eliminan flags de observación y confidencialidad
> (\'obs_flag\', \'conf_status\') cuando no se eliminaron en el paso
> anterior por tener algún valor no nulo puntual: no son variables
> analíticas sino metadatos de calidad del dato en origen, que no se van
> a explotar en este proyecto.

**Resultado de la reducción**

  -------------------------------------------------------------------------------
  **Dataset**   **Cols.        **Cols.     **Cols.        **Tipo de reducción**
                originales**   finales**   eliminadas**   
  ------------- -------------- ----------- -------------- -----------------------
  DF1 (Job      13             13          0              Sin reducción necesaria
  Posts)                                                  

  DF2 (Ofertas  19             6           13             Vacías (6) + Constantes
  por                                                     (6) + Fuera alcance (1)
  ocupación)                                              

  DF3 (Emp. por 23             7           16             Vacías (4) + Constantes
  tamaño)                                                 (9) + Fuera alcance (3)

  DF4 (Salarios 5              5           0              Sin reducción necesaria
  EU)                                                     

  DF5 (%        17             6           11             Vacías (6) + Constantes
  ofertas ICT)                                            (6)

  DF6--DF9      19 c/u         \~7--8 c/u  \~11--12 c/u   Vacías (4) + Constantes
  (Emp. por                                               (5) + Fuera alcance (3)
  perfil)                                                 

  DF10 (%       9              5           4              Columnas de metadatos
  empleo total)                                           (DATAFLOW, LAST UPDATE,
                                                          obs_flag, unit)

  DF11          23             \~7         \~16           Vacías + Constantes +
  (Sectores que                                           Fuera alcance
  contratan)                                              

  DF12          11             11          0              Sin reducción en este
  (Salarios                                               paso (tratamiento
  2020-2024)                                              específico)
  -------------------------------------------------------------------------------

> **5. Paso 5 --- Tratamiento de Valores Nulos**
>
> El tratamiento de nulos se diseñó con una regla diferenciada según el
> tipo de variable, aplicando el principio de intervención mínima.

**Regla 1: eliminar filas con obs_value nulo**

> En los datasets de Eurostat, obs_value es la variable principal de
> análisis (el indicador cuantitativo). Un registro sin valor en esta
> columna no tiene utilidad para ningún análisis posterior. Por esta
> razón, las filas con obs_value nulo se eliminan directamente.
>
> Solo DF3 presentó un número relevante de eliminaciones (17 filas, 0,5%
> del total), correspondientes a combinaciones de país × segmento × año
> para las que Eurostat no dispone de dato. El resto de datasets no tuvo
> eliminaciones.

**Regla 2: imputar \'Unknown\' en categóricas**

> Para el resto de columnas categóricas (object) con valores nulos, se
> optó por la imputación con el valor \'Unknown\' en lugar de eliminar
> la fila. Esta decisión se tomó por las siguientes razones:

-   El registro sigue siendo válido para el resto de dimensiones del
    dataset.

-   La eliminación de filas por nulos en columnas secundarias reduciría
    el dataset de forma innecesaria.

-   \'Unknown\' es una categoría explícita y reconocible que no
    distorsiona los análisis de distribución, ya que puede filtrarse
    fácilmente.

  -----------------------------------------------------------------------
  **Decisión conscientemente tomada** No se imputó ninguna variable
  numérica. Los nulos en variables numéricas
  secundarias (como remote_ratio en DF12) se mantienen para no introducir
  sesgo. La imputación de variables numéricas, si procede, se realizará
  en el momento del análisis específico que las necesite.

  -----------------------------------------------------------------------

**Tratamiento específico en DF1 (Job Posts)**

> Para el dataset de ofertas de empleo, los nulos en job_title (3
> registros, 0,3%) y location (2 registros) se imputan con \'Unknown\'
> siguiendo la misma regla general. La variable status, con un 27% de
> nulos, también recibe la imputación \'Unknown\', lo que se refleja en
> la distribución final del dataset limpio.
>
> **6. Paso 6 --- Preparación del Dataset de Skills**
>
> El dataset de ofertas de empleo (DF1) requirió un tratamiento
> específico para extraer y estructurar la información de habilidades,
> que es uno de los focos principales del análisis del proyecto.

**Qué se hizo**

-   Filtrado de registros: se eliminaron las filas donde la columna
    skills contenía una lista vacía (\[\]), ya que no aportan
    información sobre competencias requeridas. Se mantuvieron 743
    registros de los 944 originales.

-   Selección de columnas relevantes: se conservaron únicamente las
    variables necesarias para el análisis de skills: job_title,
    seniority_level, status, industry, company_size y skills.

-   Normalización de company_size: se eliminaron los separadores de
    miles y se convirtió a numérico cuando fue posible. Los valores no
    convertibles (ej. formatos con símbolo de divisa) se imputan como
    \'Unknown\'.

-   Explosión de la columna skills: mediante explode(), se transformó el
    dataset de forma que cada skill ocupa una fila independiente. El
    resultado es un dataset estructurado en formato largo, adecuado para
    análisis de frecuencia y co-ocurrencia.

  -----------------------------------------------------------------------
  **Resultado** El dataset de skills resultante (job_posts_ds_2025.csv)
  tiene una fila por cada combinación oferta--skill, permitiendo análisis
  directos de frecuencia, correlación con seniority o sector, y
  visualizaciones de ranking de habilidades.

  -----------------------------------------------------------------------

**Qué no se hizo y por qué**

> No se normalizaron semánticamente los nombres de las skills (ej.
> unificar \'ml\', \'machine learning\' y \'machine_learning\' como una
> sola categoría). Esta tarea requiere un análisis de texto específico y
> decisiones sobre sinónimos que se realizarán en la fase de análisis,
> donde el contexto es más claro y no se desea perder granularidad en
> esta fase.
>
> **7. Paso 7 --- Limpieza de Variables Categóricas**

**Qué se hizo**

> Se aplicó una limpieza básica a todas las columnas de tipo object en
> todos los datasets: eliminación de espacios al inicio y final
> (.strip()) y conversión a mayúsculas (.upper()). Esta operación
> garantiza que categorías equivalentes con distintas mayúsculas o
> espacios accidentales no generen duplicados en los análisis de
> agrupación.

**Qué no se hizo y por qué**

> No se unificaron categorías semánticamente equivalentes entre
> distintos datasets. Por ejemplo, el valor \'Germany\' en DF4 y \'DE\'
> en los datasets de Eurostat representan el mismo país pero con
> codificaciones diferentes. Esta homogeneización cruzada requiere un
> mapeo completo de equivalencias que es más manejable en Power Query,
> donde se puede gestionar como una tabla de referencia.
>
> Tampoco se realizó corrección ortográfica ni normalización de textos
> libres (como los valores de la columna location en DF1). Estos campos
> tienen una variabilidad tan alta que su tratamiento es propio de una
> fase de análisis geográfico específica.
>
> **8. Paso 8 --- Agrupación y Variables Derivadas**
>
> En este paso se crean nuevas columnas derivadas que simplifican
> categorías de alta cardinalidad y facilitan la visualización y el
> análisis. A diferencia del resto de pasos, este no modifica los
> valores originales sino que añade columnas nuevas.

**Variables creadas por dataset**

  -------------------------------------------------------------------------------
  **Dataset**     **Variable        **Variable           **Criterio de
                  original**        derivada**           agrupación**
  --------------- ----------------- -------------------- ------------------------
  DF1 (Job Posts) company_size      company_size_group   Rangos: Startup (0-50),
                  (numérico)                             SME (51-250), Mid-size
                                                         (251-1000), Large
                                                         (1001-5000), Enterprise
                                                         (\>5000).

  DF2             Descripción       occupation_short     Etiquetas cortas para
  (Ocupaciones)   ISCO-08 larga                          las 7 categorías de
                                                         ocupación TIC. Facilita
                                                         visualización en
                                                         gráficos.

  DF7 (Nivel      Descripción       education_group +    Tres grupos: Básica
  educativo)      ISCED-11 larga    education_short      (ED0-4), Media (ED5-6),
                                                         Superior (ED7-8).
                                                         Etiqueta corta para
                                                         visualización.

  DF8 (Sector)    Descripción NACE  sector_group         Agrupación en grandes
                  Rev. 2 larga                           sectores económicos
                                                         (TIC, Industria,
                                                         Servicios, etc.) a
                                                         partir del código NACE.

  DF11 (Sectores  Descripción NACE  sector_group         Mismo criterio que DF8
  contratantes)   Rev. 2                                 para mantener coherencia
                                                         entre datasets de empleo
                                                         sectorial.
  -------------------------------------------------------------------------------

  -----------------------------------------------------------------------
  **Nota** Las columnas originales de las que derivan estas variables se
  mantienen en el dataset. La versión derivada convive con la original
  para no perder granularidad en análisis que puedan requerirla.

  -----------------------------------------------------------------------

> **9. Tratamiento Específico --- Dataset de Salarios (DF12)**
>
> El dataset salaries_concat.csv requirió un tratamiento diferenciado
> dado que, a diferencia de los demás, combina datos de múltiples años
> de Kaggle y presenta inconsistencias de codificación internas.

**Qué se hizo**

-   Imputación de remote_ratio: los valores nulos se imputan con
    \'Unknown\', siguiendo el criterio general para variables
    categóricas secundarias.

-   Estandarización de experience_level: se unificaron variantes
    textuales con los códigos estándar del dataset (EN, MI, SE, EX). Por
    ejemplo, \'Senior\' → \'SE\', \'Entry\' → \'EN\'.

-   Estandarización de employment_type: se mapearon variantes como
    \'Full-Time\' → \'FT\', \'Part-Time\' → \'PT\'.

-   Estandarización de company_size: se mapearon \'Small\' → \'S\',
    \'Medium\' → \'M\', \'Large\' → \'L\'.

-   Normalización de países: se unificó la representación de países en
    company_location y employee_residence a códigos ISO de 2 letras para
    todos los valores que estaban en formato de nombre completo.

-   Creación de remote_type: variable derivada con tres categorías
    (On-site, Hybrid, Remote) a partir de remote_ratio.

-   Creación de role_category: agrupación de job_title en categorías
    analíticas más amplias (Data Science, ML/AI, Data Engineering,
    Analytics, Management).

-   Creación de salary_band: segmentación por cuartiles del salario en
    USD. Permite análisis de distribución sin dependencia del valor
    absoluto.

-   Creación de international_job: flag booleano que indica si la
    empresa y el empleado están en países distintos.

-   Deduplicación: se eliminaron duplicados exactos basados en las
    variables clave del dataset.

**Qué no se hizo y por qué**

> No se convirtieron los salarios a una divisa común distinta de USD. El
> campo salary_in_usd ya proporciona una base comparable. La conversión
> adicional a EUR (relevante para el contexto europeo del proyecto) se
> realizará en Power Query, donde se puede gestionar el tipo de cambio
> como parámetro configurable.
>
> No se filtraron registros por año ni por país para no perder datos que
> puedan ser útiles en análisis de tendencias temporales o en
> comparativas globales vs. europeas.
>
> **10. Exportación de Datasets Procesados**
>
> Tras completar el pipeline de limpieza, todos los datasets se
> exportaron en formato CSV al directorio data/processed/. La siguiente
> tabla recoge la correspondencia entre el dataset original y el nombre
> del fichero de salida.

  ---------------------------------------------------------------------------------------------
  **Dataset original**              **Fichero procesado**               **Observaciones**
  --------------------------------- ----------------------------------- -----------------------
  salaries_concat.csv               ds_salaries_2020_2024.csv           Tratamiento específico
                                                                        de salarios aplicado.

  eu_data_jobs_salaries.csv         eu_ds_jobs_salaries.csv             Sin cambios
                                                                        sustanciales. Columnas
                                                                        ya limpias.

  Porcentaje especialistas por      ict_employment_by_age.csv           Pipeline Eurostat
  edad.csv                                                              completo. 6 columnas
                                                                        finales.

  Porcentaje especialistas por      ict_employment_by_education.csv     Education_group y
  nivel educativo.csv                                                   education_short
                                                                        añadidos.

  Porcentaje especialistas por      ict_employment_by_gender.csv        Pipeline Eurostat
  sexo.csv                                                              completo.

  Porcentaje especialistas por      ict_employment_by_sector.csv        Sector_group añadido.
  sector.csv                                                            Mayor dataset del
                                                                        proyecto.

  Empresas por tamaño que contratan ict_hiring_by_company_size.csv      Pipeline Eurostat
  ICT.csv                                                               completo.

  Número/proporción de ofertas      ict_job_ads_share.csv               Pipeline Eurostat
  ICT.csv                                                               completo. Frecuencia
                                                                        trimestral.

  data_science_job_posts_2025.csv   job_posts_ds_2025.csv               Formato largo: una
                                                                        skill por fila. 743
                                                                        ofertas base.

  Distribución ofertas tech por     tech_jobs_by_role.csv               Occupation_short
  profesión.csv                                                         añadido.

  Porcentaje profesionales TIC      ict_share_total_employment_eu.csv   Formato diferencial
  empleo total.csv                                                      (DF10). Pipeline
                                                                        reducido.

  Sectores que contratan empleados  ict_hiring_by_sector.csv            Sector_group añadido.
  ICT.csv                                                               
  ---------------------------------------------------------------------------------------------

> **11. Decisiones Conscientemente No Tomadas**
>
> Este apartado recoge de forma explícita las transformaciones que se
> evaluaron pero se descartaron o delegaron, con la justificación de
> cada decisión.

**Homogeneización de nombres de países entre fuentes**

> Los datasets de Eurostat usan códigos ISO (\'ES\', \'DE\') mientras
> que algunos datasets de Kaggle usan nombres completos (\'Spain\',
> \'Germany\'). Esta unificación se delega a Power Query porque requiere
> una tabla de mapeo completa y es una tarea de integración entre
> fuentes, no de limpieza individual de cada dataset.

**Conversión de salarios a EUR**

> El dataset de salarios incluye salary_in_usd como referencia
> universal. La conversión a EUR, más relevante para el contexto europeo
> del proyecto, requiere un tipo de cambio de referencia que se
> gestionará como parámetro en Power Query.

**Normalización semántica de skills**

> Sinónimos y variantes de las mismas tecnologías (ej. \'pytorch\' vs
> \'PyTorch\', \'ml\' vs \'machine learning\') se dejan sin unificar.
> Esta normalización es propia de una fase de análisis de texto que
> requiere decisiones sobre qué considerar equivalente y puede variar
> según el análisis específico.


**Tratamiento de outliers**

> El EDA identificó outliers en variables como obs_value (datasets
> Eurostat) y salary_in_usd (dataset de salarios). Dado que los outliers
> en el contexto de este proyecto reflejan diferencias estructurales
> reales entre países y mercados (no errores de medición), no se tratan
> en la fase de limpieza. Su impacto se valorará en cada análisis
> específico.

**Imputación de valores numéricos**

> No se imputó ninguna variable numérica más allá de la eliminación de
> filas con obs_value nulo. Las imputaciones numéricas (media, mediana,
> forward fill) introducen sesgo cuyo efecto es difícil de controlar sin
> conocer el contexto analítico de uso. Se reserva para análisis
> específicos si fuera necesario.
>
> **12. Resumen de Decisiones**

  -----------------------------------------------------------------------
  **Se hizo** Pipeline completo de limpieza genérico aplicado a todos los
  datasets. Reducción de columnas en Eurostat (eliminación de metadatos,
  vacíos y constantes). Tratamiento de nulos con regla diferenciada.
  Normalización básica de categóricas. Preparación especializada de
  skills en formato largo. Variables derivadas de agrupación en datasets
  seleccionados. Limpieza específica del dataset de salarios 2020-2024.

  -----------------------------------------------------------------------

  -----------------------------------------------------------------------
  **Se delegó a Power Query** Homogeneización cruzada entre fuentes:
  nombres de países, códigos de ocupación, categorías equivalentes en
  distintos datasets. Conversión de salarios a EUR.

  -----------------------------------------------------------------------

  -----------------------------------------------------------------------
  **Se deja para análisis** Normalización semántica de skills. Parsing de
  la columna salary en job posts. Tratamiento de outliers. Imputación de
  variables numéricas secundarias.

  -----------------------------------------------------------------------

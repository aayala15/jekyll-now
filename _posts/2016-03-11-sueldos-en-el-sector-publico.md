---
layout: post
title: "Sueldos en el sector público: ¿Qué variables explican las diferencias en el nivel de remuneraciones?"
date: 2016-03-11
---
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>

En el año 2008 fue promulgada la Ley de Acceso a la Información. Esta ley exige a los distintos órganos del Estado de Chile la publicación mensual de información acerca del personal contratado y sus remuneraciones. Estos datos son de acceso público en del sitio web de Gobierno Transparente.

La información disponible provee detalles además de la labor y formación de los empleados, lo que hace a esta base de datos un interesante laboratorio para analizar las variables que afectan el nivel de remuneraciones. Evidentemente, la realidad del sector público es muy distinta a la del sector privado, por lo que se debe tener cuidado al generalizar los resultados aquí obtenidos. Los trabajadores públicos son en general de mayor calificación y se encuentran en su gran mayoría sindicalizados.

El primer paso es bajar la información desde la página de Gobierno Transparente. Dado que los datos no están consolidados, utilicé un script para buscar, ordenar y estandarizar la información desde los sitios web de las distintas entidades. La muestra incluye todos los empleados de planta, a contrata y a honorarios que ejercen funciones en ministerios y superintendencias. Los datos son a diciembre de 2015.

El objetivo es estimar un modelo que permita explicar la variación transversal del nivel de remuneraciones en el sector público. Considero variables explicativas relacionadas con las siguientes áreas:
- Repartición o entidad donde trabaja el empleado
- Tipo de contrato
- Formación
- Cargo

Dado que los datos disponibles en el sitio de Gobierno Transparente no se encuentran estandarizados, realizo un pre-análisis semántico de los textos que describen la formación y cargo del empleado a través del Natural Language Toolkit (NLTK) de Python.

Una vez estandarizada la información, procedo a estimar un modelo lineal del tipo,

$$
R_i = \alpha + \sum_j \beta_j D_{ij} + \varepsilon_i,
$$

donde $$R_i$$ es el sueldo mensual del trabajador i en millones de pesos y $$D_{i, j}$$ es una variable dummy que toma el valor 1 si el texto asociado al trabajador i incluye la palabra j.

El modelo es capaz de explicar aproximadamente el 50% de la varianza transversal. Gráficamente esto se puede ver en la siguiente figura, la cual muestra el sueldo de cada trabajador versus la predicción del modelo.

![Ajuste del modelo]({{ site.github.url }}/images/fit_sueldo.png)

Llama la atención la asimetría en la distribución de errores. Este es un hecho interesante, ya que significada la existencia de un número importante de casos donde el modelo subestima significativamente el sueldo de los trabajadores. La más probable explicación de este fenómeno es que la fijación del sueldo en el sector público está dada por variables que van más allá de la información sobre formación, cargo, entidad, etc. disponible en Gobierno Transparente.

Qué variables están asociadas a remuneraciones más altas en el sector público? A continuación presento gráficamente el valor estimado de los coeficientes más significativos en la regresión. Cabe señalar que dado que la mayoría de las variables son dummies, los valores presentados corresponden al efecto de la característica relativo al trabajador promedio. Estos son los resultados.

Relativo a trabajadores a contrata, los trabajadores de planta ganan en promedio aproximadamente 400 mil pesos más, mientras que los empleados a honorarios ganan significativamente menos.

![Tipo de contrato]({{ site.github.url }}/images/coef_contrato.png)

Como es de esperar ministros y directores de servicio son los que más ganan, seguidos por subsecretarios. Administrativos y auxiliares, en cambio, ganan relativo al trabajador promedio unos 400 mil y 900 mil pesos menos respectivamente.

![Tipo de cargo]({{ site.github.url }}/images/coef_cargo.png)

A pesar de que el mercado laboral en el sector público no es 100% competitivo, los resultados muestran que trabajadores de mayor formación perciban sueldos mayores. Por ejemplo, relativo a personas con educación universitaria, empleados con un doctorado ganan en promedio algo más de 1.5 millones extra, mientras que trabajadores con magister obtienen aproximadamente de dicho premio. Diplomados son valorados en 200 mil pesos mensuales.

![Grado educacional]({{ site.github.url }}/images/coef_titulo.png)

Egresar desde un cierto grupo de universidades también está asociado con premios/descuentos en el sueldo percibido. Dentro del sector público, un título de la Universidad de Chile aumenta en promedio el honorario bruto en algo más de 600 mil pesos. La siguen la Universidad de Santiago y la Universidad Católica. Por el contrario, relativo al universitario promedio, los egresados desde la Universidad Finis Terrae y la Universidad de Antofagasta perciben 200 mil y 180 mil pesos menos, respectivamente. Cabe señalar que no todas las universidades fueron incluidas como variables explicativas, ya que en algunos casos no había un número suficiente de casos como para obtener estimaciones precisas.

![Universidades]({{ site.github.url }}/images/coef_universidad.png)

Sin duda otra variable muy importante a la hora de intentar explicar las remuneraciones es la carrera que estudió el trabajador. La siguiente figura muestra las coeficientes estimadas para las carreras que más se repetían en la base de datos. Ingenieros civiles, abogados e ingenieros comerciales son los que perciben las remuneraciones más altas. Mientras que los empleados egresados de carreras técnicas como dibujo técnico, técnico jurídico y secretariado perciben sueldos menores que las del trabajador promedio.

![Tipo de carrera]({{ site.github.url }}/images/coef_carrera.png)

Hay reparticiones que pagan mejor que otras? A continuación presento los valores de los coeficientes estimados para las variables dummies asociadas a la repartición. Cabe recordar que el modelo controla por formación, cargo y carrera, por lo que los resultados aquí expuestos son premios por trabajar en cierto ministerio que no son explicados por dichos controles. Ahora, dado que la cantidad de información provista por cada entidad puede ser distinta, los resultados aquí expuestos pueden estar sesgados si por ejemplo la Dipres provee mucho menos información que la subsecretaria de Justicia.

![Tipo de entidad]({{ site.github.url }}/images/coef_entidad.png)

Finalmente, utilizo los datos para hacer un análisis de la dispersión de los sueldos en el sector público. Para ello calculo tres coeficientes de gini: el gini total del sector público, el gini al interior de cada ministerio y el gini entre entidades. Los resultados se pueden ver en el gráfico a continuación. El gini total en el sector público es aproximadamente 0.32, significativamente más bajo que el del sector privado. Cabe destacar que el gini intra ministerios es casi idéntico al gini total. Es quiere decir que gran parte de la desigualdad en el sector público viene dada por diferencias en función y formación y no por las caracteristicas del empleador.

![Tipo de entidad]({{ site.github.url }}/images/gini_summary.png)

Por último, el siguiente gráfico muestra el coeficiente de gini dentro de cada ministerio. Los valores son relativos al gini total del sector público, por lo que números positivos (negativos) indican que dicho ministerio presenta niveles de desigualdad mayores (menores) que el promedio del sector público.

![Tipo de entidad]({{ site.github.url }}/images/gini_entidad.png)
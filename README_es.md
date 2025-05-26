# Generador de Horarios
Generador de horarios para la universidad implementado en Python usando *algoritmos genéticos*.

### Resumen
Este proyecto implementa una de las posibles soluciones para generar horarios universitarios.
La solución propuesta se basa en métodos de computación evolutiva, utiliza la *estrategia evolutiva (1+1)* y el *templado simulado*.
El éxito de la solución se estima en el cumplimiento de las restricciones y criterios dados.
Los resultados de las pruebas del algoritmo muestran que se satisfacen todas las restricciones estrictas, mientras que los criterios adicionales se optimizan hasta cierto punto.

### Problema
La tarea consiste en encontrar una solución genérica que facilite la generación de horarios para la universidad (este problema específico está ajustado a la `Facultad de Informática de Belgrado`).

Cada clase en la facultad se representa como un bloque (dura un número arbitrario de horas, generalmente de 1 a 4).
Para impartir cada clase se requiere: *profesor*, *aula*, *hora de inicio*, *duración* y *grupos* que asisten a la clase. También se sabe de antemano qué grupos asisten a qué clase y todas las aulas son del mismo tamaño (cada grupo cabe en un aula).
<br />La enseñanza se realiza en la facultad de 9 a.m. a 9 p.m. todos los días laborables.

Los **datos de entrada** para cada clase son el *nombre del profesor*, *asignatura*, *tipo*, *duración* y lista de *aulas permitidas*.
<br />Los **datos de salida** son el aula y la hora de cada clase. La hora se determina por el día (de lunes a viernes) y la hora de inicio de la clase.

### Restricciones

1. Los recursos no pueden superponerse en el tiempo
    - Ningún profesor puede impartir dos clases al mismo tiempo
    - Ningún grupo puede asistir a dos clases al mismo tiempo
    - Ningún aula puede albergar dos clases al mismo tiempo
    > Nota: bajo el término "mismo tiempo" no se entiende solo el inicio de la clase, se debe tener en cuenta la duración de la clase.
          Si el recurso está ocupado en el momento `T1` y la clase dura `t1`, entonces el recurso solo puede volver a ocuparse en el momento `T2 = T1 + t1`.
2. La clase debe tener lugar en una de las aulas permitidas
3. Si la asignatura tiene varias formas de enseñanza, el orden preferido para cada grupo es *clases teóricas*, *ejercicios* y *ejercicios de laboratorio*.

Las restricciones `1` y `2` deben cumplirse, mientras que el límite `3` es "flexible" y se permite su incumplimiento.
<br />Criterios adicionales para estimar la solución (utilizados también para la función de coste):
  * Cumplir las restricciones estrictas (1 y 2)
  * Cumplir las restricciones flexibles (3)
  * Minimizar el "tiempo muerto" total para cada grupo (eliminando la pausa entre clases)
  * Minimizar el "tiempo muerto" total para cada profesor (eliminación de la pausa entre clases)
  * Proporcionar una hora en la semana lectiva en la que nadie tenga clases

### Solución

El algoritmo para el horario se representa como una tabla con sus columnas como aulas y filas los horarios permitidos para las clases, mientras que los elementos de la tabla son la clase específica que se imparte en el aula adecuada y en el horario dado.
La tabla es una matriz con dimensiones 60 x número de aulas, donde 60 corresponde al número total de horarios posibles durante la semana (5 días laborables, 12 horas al día).

Un elemento de la matriz puede estar vacío si no se imparte ninguna enseñanza en el horario y aula dados o puede contener el índice de clase apropiado.
<br />El índice de la clase se determina por: *nombre del profesor*, *asignatura*, *tipo* (clases teóricas, ejercicios o ejercicios de laboratorio), *duración*, *aulas permitidas* y *grupos*.
Debido a la diferente duración de las clases, los mismos índices aparecen en la tabla varias veces y siempre están en bloques verticales (ocupan campos consecutivos en la misma columna).

La representación del horario se muestra en la siguiente imagen.

La idea básica de este enfoque es la simplicidad para cambiar los términos y las aulas simplemente cambiando el bloque en la matriz que ocupa la clase.
La matriz de horarios permite que siempre se satisfaga una de las restricciones estrictas, mientras que el cálculo del coste se realiza fácilmente.
Debido a la forma de colocar las clases en la matriz, se asegura que como máximo una clase esté en un aula en un momento dado, el algoritmo nunca colocará una clase en un campo no vacío.

Además, la comprobación de si la clase está en el aula adecuada es simplemente examinar si la clase puede estar en esa columna.
Las superposiciones de profesores y grupos se examinan comprobando el resto de las clases que están en la misma fila.
La restricción flexible y los criterios adicionales, como el tiempo muerto para los grupos, se calculan basándose en estructuras adicionales en tiempo lineal.

Además de la idea general para resolver el problema, para la optimización y el cumplimiento de las restricciones se utilizan dos métodos de computación evolutiva, la *estrategia evolutiva (1+1)* y el *templado simulado*.

### Algoritmo

El algoritmo consta de los siguientes pasos:
1. **Carga y procesamiento de datos**
  <br />Las clases se cargan desde el archivo JSON y se asignan a las estructuras apropiadas.
  Además, se realiza una mezcla aleatoria de datos.
2. **Configuración del modelo**
  <br />Creación de la matriz de horarios y diccionarios que representan los campos vacíos, los campos llenos en una matriz, el orden de las clases para cada tipo de clase y grupo, un tiempo muerto de grupo vacío y un tiempo muerto de profesor vacío.
3. **Generación de la población inicial**
  <br />Las clases se organizan en espacios en blanco verticalmente consecutivos de la matriz para que estén en el aula apropiada.
4. **Estrategia evolutiva (1 + 1) para restricciones estrictas**
  <br />La selección de clases que cambian de lugar en el horario se basa en el coste de las clases calculando solo las restricciones estrictas. Se eligen las clases con el coste más alto y con una cierta probabilidad mutan. La mutación es cambiar el campo en la matriz buscando un campo libre que cumpla con todas las restricciones rígidas. También se utiliza la notación de Schwefel.
5. **Templado simulado para criterios adicionales**
  <br />Optimiza el horario generado previamente minimizando el tiempo muerto para los grupos durante todos los días y la existencia de una hora libre en la que no haya enseñanza. En cada iteración, un número aleatorio de clases muta, es decir, cambian de lugar en la matriz para que continúen cumpliendo las restricciones estrictas. Para el programa de enfriamiento, se utiliza un descenso geométrico de la temperatura.
6. **Representación del horario y estadísticas**

### Pruebas y Resultados
El algoritmo se probó en 3 horarios diferentes (`input1.txt`, `input2.txt`, `input3.txt`) y para cada uno se ejecutó 10 veces.

La asignación de clases se realiza primero mediante un algoritmo evolutivo, donde el parámetro sigma toma el valor `2`, el número de ejecuciones `5`, mientras que el estancamiento máximo es `200`.
Después de eso, el horario se optimiza mediante el algoritmo de templado simulado donde la temperatura inicial es `0.5` y el número de iteraciones es `2500`.

Los resultados de todas las ejecuciones se muestran en la `Tabla 1`. Además, para cada archivo de entrada, se ha seleccionado un horario y las estadísticas detalladas se muestran en la `Tabla 2`.
Este horario se seleccionó en función del menor tiempo muerto promedio para grupos y profesores, sujeto a la disponibilidad de existencia de una hora libre en la semana.

|               | *input1.txt*  | *input2.txt*  | *input3.txt*  |
| ------------- | ------------- | ------------- | ------------- |
| Cumplimiento de restricciones estrictas  | 100% | 100%  | 100%  |
| Cumplimiento de restricciones flexibles  | 58.42%  | 44.56% | 47.52% |
| Tiempo muerto promedio para grupos <br />(todos los días juntos)| 2.43 | 3.35 | 4.27|
| Tiempo muerto promedio para profesores <br />(todos los días juntos)| 0.22 | 1.04 | 1.22|
| Existencia de hora libre  | 100% | 100%  | 100%  |

***Tabla 1:** Estadísticas obtenidas por el promedio de 10 ejecuciones de prueba.*<br />
<br />

|               | *input1.txt*  | *input2.txt*  | *input3.txt*  |
| ------------- | ------------- | ------------- | ------------- |
| Cumplimiento de restricciones estrictas  | 100% | 100%  | 100%  |
| Cumplimiento de restricciones flexibles  | 51.58%  | 51.02% | 46.11% |
| Tiempo muerto máximo para grupos <br />(en un día) | 4 | 6 | 6|
| Tiempo muerto total para grupos <br />(todos los días juntos)| 31 | 145 | 184 |
| Tiempo muerto promedio para grupos <br />(todos los días juntos)| 1.29 | 3.30 | 4.18|
| Tiempo muerto máximo para profesores <br />(en un día) | 7 | 6 | 8|
| Tiempo muerto total para profesores <br />(todos los días juntos)| 17 | 64 | 110 |
| Tiempo muerto promedio para profesores <br />(todos los días juntos)| 0.45 | 1.03 | 1.75|
| Hora libre | Viernes 9AM | Jueves 8PM  | Viernes 8PM |

***Tabla 2:** Estadísticas de los mejores horarios de 10 ejecuciones de prueba.*

### Uso
Ejecutar `scheduler.py` ejecuta el código que ejecuta el algoritmo para los datos que se cargan desde el archivo cuya ruta se encuentra en la función `main` en el mismo archivo.

# Laboratorio 3: Aplicación inteligente adaptativa

## Identificación

- **Nombre:** Cristopher Dauros.
- **Asignatura:** OII436 - Ingeniería Web Avanzada
- **Fecha:** 9 de Noviembre de 2026.
- **Repositorio:** [Crisss333/OII436_Laboratorio3](https://github.com/Crisss333/OII436_Laboratorio3).

## Objetivo y alcance

Se implementó un recomendador de recursos de aprendizaje por consola. Se representaron
los intereses y las etiquetas de los recursos mediante TF-IDF, se calcularon afinidades
con similitud del coseno y se actualizó automáticamente el perfil del usuario activo al
registrar **Me interesa** sobre un recurso del catálogo completo.

Se completaron las cuatro funciones indicadas en `motor.py`. Se conservaron `app.py`
y los dos archivos de datos proporcionados. Se mantuvieron los mismos **12 candidatos**
antes y después de cada interacción, incluido el recurso valorado. El estado se conserva
en memoria durante la ejecución; al reiniciar se recuperan los intereses iniciales y se
inician historiales vacíos. La entrega corresponde a la versión de consola prevista en
el enunciado; Angular es una extensión opcional.

## Entorno y reproducción

Se creó y utilizó un entorno virtual `.venv` con **Python 3.12.14**. Se recomienda
Python **3.12** para reproducir las versiones fijadas en `requirements.txt`. Las
bibliotecas principales utilizadas fueron NumPy 2.5.4, Pandas 3.1.1 y scikit-learn
1.9.1; el archivo de dependencias también registra sus dependencias transitivas.

### Obtener el proyecto y crear el entorno

Con Git y Python 3.12 disponibles, se ejecutan los siguientes comandos:

```bash
git clone https://github.com/Crisss333/OII436_Laboratorio3.git
cd OII436_Laboratorio3
```

En macOS o Linux:

```bash
python3.12 -m venv .venv
source .venv/bin/activate
```

El comando de activación correspondiente al entorno de trabajo en macOS es
`source .venv/bin/activate`.

En Windows, se crea el entorno con:

```powershell
py -3.12 -m venv .venv
```

Activación en PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Activación en CMD:

```bat
.venv\Scripts\activate.bat
```

Si el intérprete ya se invoca como `python` y corresponde a Python 3.12, también
puede utilizarse `python -m venv .venv` para crear el entorno.

### Instalar, comprobar y ejecutar

Con el entorno activado:

```bash
python --version
python -c "import sys; print(sys.executable)"
python -m pip install -r requirements.txt
python -m pip check
python app.py --comprobar
python app.py --demo
```

La ruta mostrada por `sys.executable` debe corresponder al Python del entorno `.venv`.
La comprobación realizada informó **12 recursos, 4 usuarios y una matriz TF-IDF de
12 filas por 17 términos**. El modo `--demo` parte de perfiles nuevos, ejecuta T1-T4,
muestra sus resultados y genera los dos archivos de `evidencias/`. Devuelve código
de salida 0 cuando las cuatro comprobaciones cumplen.

Las versiones instaladas se registraron con:

```bash
python -m pip freeze > requirements.txt
```

Para repetir la ejecución desde una copia nueva, se siguen los comandos de clonación,
creación, activación e instalación anteriores. No se requieren servicios externos,
credenciales ni descargas de datos: el catálogo y los perfiles están incluidos. La
instalación inicial de dependencias requiere acceso al índice de paquetes de Python.

### Modo interactivo

```bash
python app.py
```

Las opciones disponibles son `r` para recomendar, `c` para consultar el catálogo,
`m` para registrar **Me interesa**, `u` para cambiar de usuario y `s` para salir.
Se puede reproducir la secuencia de adaptación introduciendo cada entrada en una
línea cuando la consola la solicite:

```text
u1
r
c
m
R04
r
u
u2
r
s
```

Primero se consultan las recomendaciones de u1 y el catálogo completo. Después se
valora R04, aunque no figure entre los tres resultados iniciales. Se muestra el perfil
actualizado y se recalculan las recomendaciones en la misma sesión. Al cambiar a u2,
se consulta su propio estado. Un identificador de recurso desconocido produce un
mensaje y deja el perfil y el historial sin cambios.

Para terminar la sesión del entorno virtual se utiliza `deactivate`.

## Catálogo, perfiles y validaciones

`datos/recursos.csv` contiene las columnas `id`, `titulo`, `descripcion` y `etiquetas`.
**Solo `etiquetas` interviene en el cálculo**: cada fila se representa en el espacio
TF-IDF aprendido del catálogo. El identificador permite seleccionar y desempatar;
el título y la descripción presentan el contenido al usuario.

La carga suministrada elimina espacios exteriores, exige las cuatro columnas,
rechaza un catálogo vacío y rechaza valores vacíos en cualquiera de esos campos.
Si se repite un `id`, se lanza un error: los identificadores deben ser únicos. La
aplicación informa el error; no se completan datos ni se descartan duplicados
silenciosamente. Se conservaron los identificadores y las etiquetas originales.

`datos/perfiles.json` contiene el nombre y los intereses iniciales de cada usuario:

| Usuario | Intereses iniciales | Representación inicial |
|---|---|---|
| u1 | `python datos pandas` | Vector TF-IDF con términos reconocidos. |
| u2 | `angular componentes interfaces` | Vector TF-IDF distinto al de u1. |
| u3 | Texto vacío | Vector nulo de 17 componentes. |
| u4 | `zzxyz123` | Vector nulo: el término no pertenece al vocabulario. |

`crear_estado` crea un vector independiente para cada usuario y `crear_historial`
crea una lista independiente por usuario. Los intereses iniciales permanecen en los
datos cargados; la adaptación se aplica sobre el **vector actual** almacenado en
`estado`, y el evento se conserva en `historial`.

El vocabulario se aprende una sola vez al iniciar la aplicación mediante
`construir_indice`. Los 17 términos obtenidos fueron: `angular`, `api`,
`componentes`, `consultas`, `datos`, `formularios`, `funciones`, `graficos`, `http`,
`interfaces`, `pandas`, `python`, `sql`, `tablas`, `validacion`, `variables` y `web`.

## Implementación del motor

| Función completada | Operación realizada y salida |
|---|---|
| `crear_perfil` | Se aplicó `vectorizador.transform([intereses]).toarray()[0]` para obtener un vector NumPy 1D en el espacio del catálogo. |
| `calcular_puntuaciones` | Se calculó `cosine_similarity([perfil], matriz)[0]`, con una puntuación por recurso en el orden de las filas. Un perfil nulo produce puntuaciones cero. |
| `recomendar` | Se seleccionaron solo puntuaciones positivas, se ordenaron por puntuación descendente e ID ascendente y se conservaron hasta `top_k` resultados, con valor predeterminado 3. Se utilizó `construir_resultado` para añadir título y explicación. |
| `actualizar_perfil` | Se combinó el vector actual con el del recurso valorado y se devolvió un vector 1D normalizado a norma L2 unitaria. |

La regla de actualización utilizada fue:

```text
mezcla = 0.4 * perfil_actual + 0.6 * vector_recurso
perfil_nuevo = mezcla / norma_L2(mezcla)
```

La normalización se implementó con `normalize([mezcla], norm="l2")[0]`. Los pesos
0,4 y 0,6 son los parámetros didácticos del laboratorio. Se mantiene una magnitud
comparable del perfil para actualizaciones sucesivas; la similitud del coseno compara
la dirección de los vectores. Cada nueva interacción se aplica al vector actualizado
de la sesión, sin reconstruirlo desde los intereses iniciales.

Las puntuaciones se conservaron sin redondear durante los cálculos y el ordenamiento.
La consola presenta cuatro decimales; las tablas de comparación siguientes muestran
seis. Los archivos de evidencia conservan las puntuaciones generadas. Por precisión
de punto flotante, la similitud inicial de R02 quedó como `1.0000000000000002` en el
registro y se presenta como `1.0000`, equivalente a 1 dentro de la precisión numérica.

## Resultados reales de la demostración

Se ejecutó `python app.py --demo` desde el entorno virtual. Los valores de esta sección
proceden de [reporte_demo.json](evidencias/reporte_demo.json) y
[puntuaciones_antes_despues.csv](evidencias/puntuaciones_antes_despues.csv).

### Recomendaciones iniciales de u1

| Puesto | ID | Título | Puntuación | Explicación |
|---|---|---|---:|---|
| 1 | R02 | Análisis de datos con Pandas | 1.0000 | Términos compartidos: datos, pandas, python. |
| 2 | R12 | Conectar Python con datos SQL | 0.4965 | Términos compartidos: datos, python. |
| 3 | R03 | Gráficos con Python | 0.4326 | Términos compartidos: datos, python. |

Los resultados comparten los intereses de datos y Python declarados por u1. R02
coincide con los tres términos iniciales y aparece primero.

### Recomendaciones iniciales de u2

| Puesto | ID | Título | Puntuación | Explicación |
|---|---|---|---:|---|
| 1 | R07 | Componentes e interfaces en Angular | 1.0000 | Términos compartidos: angular, componentes, interfaces. |
| 2 | R08 | Formularios e interfaces en Angular | 0.5677 | Términos compartidos: angular, interfaces. |
| 3 | R09 | Consumir una API desde Angular | 0.3087 | Términos compartidos: angular. |

Se observó un ranking distinto sobre el mismo catálogo: R07 coincide con los tres
términos de u2; R08 comparte Angular e interfaces; R09 comparte Angular.

### Perfil de u1 antes y después de valorar R04

Se registró **Me interesa** en R04, «API web con Python», cuyas etiquetas son
`python web api`. Los pesos no nulos antes o después fueron:

| Término | Peso antes | Peso después |
|---|---:|---:|
| api | 0.000000 | 0.446269 |
| datos | 0.465091 | 0.232455 |
| pandas | 0.753247 | 0.376477 |
| python | 0.465091 | 0.637116 |
| web | 0.000000 | 0.446269 |

Los otros 12 términos del vocabulario permanecieron con peso cero. Se incorporaron
`api` y `web`, aumentó el peso de `python` y se conservaron `datos` y `pandas` con
menor peso relativo. Los intereses iniciales del JSON no se editaron.

El historial de u1 pasó de `[]` al siguiente registro:

```json
[
  {
    "numero": 1,
    "accion": "Me interesa",
    "recurso": "R04"
  }
]
```

### Recomendaciones de u1 después de la interacción

| Puesto | ID | Título | Puntuación | Explicación |
|---|---|---|---:|---|
| 1 | R04 | API web con Python | 0.8752 | Términos compartidos: api, python, web. |
| 2 | R02 | Análisis de datos con Pandas | 0.6880 | Términos compartidos: datos, pandas, python. |
| 3 | R09 | Consumir una API desde Angular | 0.4958 | Términos compartidos: api, web. |

El ranking cambió de **R02, R12, R03** a **R04, R02, R09**. R04 se conservó como
candidato y pasó al primer lugar por su mayor afinidad con el vector actualizado.

### Puntuaciones de los mismos 12 candidatos

| ID | Puntuación antes | Puntuación después | Diferencia (después - antes) |
|---|---:|---:|---:|
| R01 | 0.186096 | 0.254928 | +0.068832 |
| R02 | 1.000000 | 0.688010 | -0.311990 |
| R03 | 0.432619 | 0.404430 | -0.028189 |
| R04 | 0.251037 | 0.875178 | +0.624141 |
| R05 | 0.000000 | 0.437777 | +0.437777 |
| R06 | 0.000000 | 0.437777 | +0.437777 |
| R07 | 0.000000 | 0.000000 | +0.000000 |
| R08 | 0.000000 | 0.000000 | +0.000000 |
| R09 | 0.000000 | 0.495760 | +0.495760 |
| R10 | 0.205290 | 0.102605 | -0.102685 |
| R11 | 0.205290 | 0.102605 | -0.102685 |
| R12 | 0.496522 | 0.464168 | -0.032353 |

**R05 no recibió una valoración** y su puntuación aumentó de `0.000000` a
`0.437777`. Sus etiquetas `web api http` comparten los términos `web` y `api`
incorporados al perfil mediante R04. El término `http` no se añadió al perfil.

También R09 pasó de `0.000000` a `0.495760` sin ser valorado y entró en las tres
recomendaciones. Sus etiquetas son `angular web api`: la nueva coincidencia se debe
a `web` y `api`. **Valorar R04 no incorporó `angular` al perfil de u1**; el peso de
ese término se mantuvo en cero. Por eso R07 y R08 conservaron puntuación cero.

Este cambio demuestra la adaptación: una interacción produjo una modificación
numérica del perfil y alteró las puntuaciones de recursos diferentes del valorado,
sin retirar ni añadir candidatos ni modificar sus etiquetas. El descenso de R02,
R03, R10, R11 y R12 refleja una dirección del perfil menos concentrada en datos,
aunque los intereses originales continúan representados.

### Independencia de u2

La prueba T3 comparó u2 antes y después de la interacción de u1. Se obtuvieron
`perfil_sin_cambios`, `puntuaciones_sin_cambios`, `recomendaciones_sin_cambios` e
`historial_sin_cambios` con valor `true` en el reporte.

| Elemento de u2 | Antes de u1/R04 | Después de u1/R04 |
|---|---|---|
| Perfil, pesos no nulos | `componentes: 0.657512`, `interfaces: 0.564679`, `angular: 0.498813` | Los mismos pesos. |
| Historial | `[]` | `[]` |
| Ranking | R07, R08, R09 | R07, R08, R09 |
| Puntuaciones del ranking | 1.0000; 0.5677; 0.3087 | 1.0000; 0.5677; 0.3087 |
| Puntuaciones de los 12 candidatos | Calculadas sobre el perfil inicial de u2. | Iguales según la comparación numérica de T3. |

La interacción se asoció exclusivamente al perfil e historial de u1.

### Perfiles sin términos reconocidos

| Usuario | Entrada | Puntuaciones | Recomendaciones | Salida observada |
|---|---|---|---|---|
| u3 | Intereses vacíos | Las 12 en cero. | `[]` | `No hay coincidencias positivas. Declare intereses presentes en el catálogo.` |
| u4 | `zzxyz123` | Las 12 en cero. | `[]` | `No hay coincidencias positivas. Declare intereses presentes en el catálogo.` |

No se rellenó el ranking con recursos de puntuación cero y la ejecución continuó
sin excepción en ambos casos. El catálogo puede explorarse con la opción `c`.

## Pruebas funcionales T1-T4

| Caso | Entrada o interacción | Resultado esperado | Resultado observado | Estado | Interpretación |
|---|---|---|---|---|---|
| T1 | Solicitar recomendaciones iniciales de u1 y u2. | Rankings coherentes y distintos, hasta tres resultados positivos, orden descendente y desempate por ID ascendente. | u1: R02 (1.0000), R12 (0.4965), R03 (0.4326). u2: R07 (1.0000), R08 (0.5677), R09 (0.3087). Las listas fueron válidas en la comprobación suministrada. | **Cumple** | Los intereses diferentes generan afinidades distintas sobre los mismos recursos. El motor usa ID ascendente como segunda clave para los empates. |
| T2 | Registrar u1/R04 con **Me interesa** y volver a recomendar en la misma ejecución. | Cambiar el perfil y la puntuación de al menos un recurso no valorado, conservando los 12 candidatos. | Se incorporaron `api` y `web`; R04 pasó a 0.8752 y al primer puesto. R05 aumentó de 0 a 0.437777 y R09 de 0 a 0.495760. Se conservaron R01-R12 y se registró un evento en u1. | **Cumple** | La modificación del vector explica los cambios de puntuación, incluido el efecto en recursos no valorados. |
| T3 | Consultar u2 después de la interacción de u1. | Mantener el vector, historial y puntuaciones de u2. | Las cuatro comprobaciones de T3 fueron verdaderas. Se mantuvieron el perfil de u2, su historial vacío y el ranking R07, R08, R09 con las mismas puntuaciones. | **Cumple** | Los usuarios conservan estados independientes. |
| T4 | Recomendar para u3 vacío y u4 con `zzxyz123`. | Listas vacías, puntuaciones cero y mensaje informativo sin fallo. | Ambos obtuvieron `[]`, las 12 puntuaciones cero y el mensaje indicado en la sección anterior. | **Cumple** | La ausencia de coincidencias no se convierte en relevancia artificial. |

Las cuatro pruebas suministradas finalizaron con **Cumple**. No se detectaron
incumplimientos funcionales en esos casos. Se conservó la aplicación de demostración
para que las evidencias fueran generadas por la ejecución del motor completado.

### Comprobaciones complementarias

Se contrastaron las similitudes de los cuatro usuarios con el producto escalar
dividido por las normas, calculado directamente con NumPy. Se obtuvieron valores
coincidentes con el motor. También se verificaron los siguientes comportamientos:

- Un catálogo temporal con puntuaciones exactamente empatadas e identificadores
  desordenados produjo **R10, R20 y R30** en ese orden para el límite predeterminado.
  Se comprobó el desempate por ID, el máximo de tres y la exclusión de ceros sin
  modificar los datos de la entrega.
- La mezcla devolvió un vector de norma L2 igual a 1 y conservó intactos los
  vectores de entrada. Una segunda interacción utilizó el perfil ya actualizado.
- El identificador desconocido `R99` dejó sin cambios todos los vectores e
  historiales. En la consola se mostró el mensaje correspondiente.
- Al cambiar de usuario se conservaron los estados independientes; una ejecución
  nueva restauró los perfiles iniciales y los historiales vacíos.
- `app.py`, `datos/recursos.csv` y `datos/perfiles.json` se compararon con el
  material suministrado y resultaron idénticos byte a byte.

Se realizó además una reproducción con un entorno virtual nuevo e instalación
desde el `requirements.txt` congelado. `pip check` no detectó incompatibilidades,
`--comprobar` y `--demo` terminaron correctamente y los dos archivos de evidencia
resultaron idénticos byte a byte a los de la ejecución original. Se comprobó que
`sys.executable` apuntaba al entorno activado y que su `sys.prefix` difería del
intérprete base.

## Preguntas conceptuales

### 1. ¿Qué información describe el contenido de los recursos y qué información constituye el perfil del usuario? Identifique ambas en los archivos del proyecto.

El contenido se describe en `datos/recursos.csv`: cada recurso contiene un `id`,
un `titulo`, una `descripcion` y sus `etiquetas`. En el algoritmo se representan
exclusivamente las etiquetas, mediante la matriz TF-IDF construida por
`construir_indice` en `motor.py`.

El perfil inicial se encuentra en `datos/perfiles.json`, donde se asocian un nombre
y un texto de intereses a cada identificador de usuario. En `app.py`, los intereses
se transforman en un vector mediante `crear_estado` y `motor.crear_perfil`. Durante
la sesión, el perfil operativo comprende el vector actual de `estado` y los eventos
separados en `historial`; los intereses cargados permanecen como punto de partida.
Por tanto, el recurso describe lo que ofrece el catálogo y el perfil representa las
preferencias del usuario en el mismo espacio de términos.

### 2. ¿Por qué se utiliza fit_transform sobre el catálogo y transform sobre los intereses del usuario? ¿Qué problema causaría ajustar un vectorizador diferente para cada perfil?

`fit_transform` aprende del catálogo el vocabulario y las frecuencias documentales
que determinan los pesos IDF, y produce su matriz de representaciones. Se ejecuta
una sola vez sobre `recursos["etiquetas"]`. `transform` reutiliza ese vocabulario,
esos pesos y el mismo orden de columnas para representar los intereses.

Si se ajustara otro vectorizador por usuario, podrían cambiar el número y el orden
de las características, así como sus pesos IDF. Los vectores dejarían de compartir
un significado comparable por componente; incluso podrían tener dimensiones
incompatibles para calcular el coseno. Las palabras de intereses ajenas al
vocabulario se ignoran al transformar, como ocurre con `zzxyz123`.

### 3. ¿Por qué el mismo catálogo puede producir recomendaciones distintas para u1 y u2? ¿Una similitud de 0,80 significa que existe un 80 % de probabilidad de que el recurso guste? Justifique.

Las representaciones de los recursos son las mismas, pero cada usuario aporta un
vector de intereses distinto. El vector de u1 pondera `python`, `datos` y `pandas`;
el de u2 pondera `angular`, `componentes` e `interfaces`. Al comparar cada uno
con el catálogo, cambian las coincidencias y el orden de afinidad. Se observó R02
primero para u1 y R07 primero para u2.

Una similitud de 0,80 expresa cercanía entre las direcciones de los vectores, según
sus características TF-IDF. No equivale a una probabilidad del 80 % de agrado: no
se entrenó ni calibró un modelo probabilístico de preferencias. Tampoco demuestra
comprensión semántica; la afinidad depende de los términos representados.

### 4. ¿En qué se diferencia modificar manualmente los intereses iniciales de la adaptación implementada? Identifique el evento que provoca el cambio, el estado modificado y su efecto observable.

Editar manualmente el campo `intereses` del JSON cambia el punto de partida de una
futura carga. La adaptación implementada sucede automáticamente durante la sesión,
sin editar ese archivo, cuando el usuario registra **Me interesa**.

En la secuencia evaluada, el evento fue u1/R04. `registrar_interes` validó el
identificador, obtuvo el vector de R04 y llamó a `actualizar_perfil`. Se sustituyó
únicamente `estado["u1"]` por la mezcla normalizada del perfil actual y el recurso;
se añadió el evento a `historial["u1"]`. A continuación se recalcularon las
puntuaciones y recomendaciones. El perfil incorporó `api` y `web`, el ranking pasó
a R04, R02, R09 y R05 aumentó su afinidad sin recibir una valoración. El estado de
u2 se mantuvo igual y los mismos 12 recursos siguieron siendo candidatos.

### 5. Relacione su implementación con las etapas del ciclo MAPE-K estudiado en clase: Monitorizar, Analizar, Planificar, Ejecutar y Conocimiento. Indique una función, operación o dato de su programa para cada etapa.

La correspondencia es conceptual: las etapas se integran en las funciones del
programa, sin constituir cinco módulos independientes.

| Etapa | Correspondencia en la implementación |
|---|---|
| **Monitorizar** | En `interactivo`, se recoge la opción `m`, el usuario activo y el `recurso_id` de la acción **Me interesa**. En `--demo` se provoca el mismo evento mediante `registrar_interes`. El historial registra la interacción aceptada. |
| **Analizar** | `registrar_interes` valida el usuario y `obtener_indice` comprueba el recurso. Se recuperan el perfil actual y el vector del recurso para identificar las características de la interacción; `calcular_puntuaciones` permite evaluar su afinidad con el catálogo. |
| **Planificar** | La política explícita en `actualizar_perfil` determina el nuevo perfil con `0.4 * perfil_actual + 0.6 * vector_recurso` y normalización L2. Esta regla fija cómo debe adaptarse el estado a la interacción. |
| **Ejecutar** | `registrar_interes` asigna el vector resultante a `estado[usuario_id]` y agrega el evento al historial. Después se llama a `recomendar`, se recalculan las afinidades y se muestran los nuevos resultados. |
| **Conocimiento** | Se utilizan el catálogo, el vocabulario y pesos del vectorizador, la matriz TF-IDF, los intereses iniciales, los vectores actuales y los historiales independientes. Las evidencias permiten inspeccionar los cambios observados. |

### 6. Indique una limitación del mecanismo implementado y proponga una mejora concreta. Explique qué dato adicional o qué cambio del algoritmo exigiría esa mejora.

El mecanismo depende de coincidencias literales entre etiquetas y términos del
perfil. Un usuario que describa un interés con sinónimos ausentes del vocabulario
puede recibir poca o ninguna afinidad aunque exista contenido pertinente. Tampoco
se utiliza la descripción más extensa de los recursos para ampliar esa evidencia.

Una mejora concreta sería incorporar un diccionario de equivalencias revisado para
el dominio, por ejemplo `analítica tabular` hacia `datos pandas`, y normalizar con él
las etiquetas y los intereses antes de representarlos. Se requeriría ese diccionario
adicional, versionado y documentado, y aplicar la misma transformación al construir
el índice y al crear los perfiles. Se deberían añadir casos de prueba con sinónimos
y revisar que no se introduzcan asociaciones incorrectas. Esta mejora se propone
como trabajo posterior; en la entrega se conservaron las etiquetas originales y el
algoritmo exigido para reproducir los casos suministrados.

## Contenido del repositorio

```text
OII436_Laboratorio3/
├── motor.py
├── app.py
├── datos/
│   ├── recursos.csv
│   └── perfiles.json
├── evidencias/
│   ├── reporte_demo.json
│   └── puntuaciones_antes_despues.csv
├── README.md
├── requirements.txt
└── .gitignore
```

Los nueve archivos anteriores contienen la implementación, los datos, las evidencias
y las instrucciones de reproducción. `.gitignore` excluye `.venv/`, `__pycache__/`,
archivos de bytecode, archivos de entorno y temporales. Las dependencias instaladas
permanecen fuera del control de versiones.

El repositorio contiene los archivos necesarios para reproducir la
aplicación y consultar sus evidencias.

## Referencias técnicas

- [Google for Developers: filtrado basado en contenido](https://developers.google.com/machine-learning/recommendation/content-based/basics).
- [scikit-learn: TfidfVectorizer, fit_transform y transform](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html).
- [scikit-learn: cosine_similarity](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.pairwise.cosine_similarity.html).
- [scikit-learn: normalización de vectores](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.normalize.html).

El catálogo, los perfiles, la aplicación de consola, las validaciones y el mecanismo
de demostración proceden del material base entregado para el Laboratorio 3. Las
cuatro funciones y la interpretación de las evidencias se completaron conforme a
la regla de actualización y a los requisitos del enunciado.

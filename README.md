# Clash_royale_cards_CuochDB_database

# Dataset usado
El dataset que escojimos clash_royale_cards_1.json contiene información completa sobre las cartas del juego Clash Royale, incluyendo unidades (troops), edificios (buildings) y hechizos (spells). Consiste en una colección de objetos JSON que representan cada carta del juego con sus atributos principales como costo de elixir, rareza, tipo, estadísticas de combate y datos de uso.

Ejemplo de uno de los objetos JSON
<img width="1305" height="382" alt="image" src="https://github.com/user-attachments/assets/9ad7e0ee-b0d1-4f35-a0dc-1e056113d28a" />

# Diccionario de Datos

| Campo | Tipo de Dato | Descripción | Valores de Ejemplo |
| :--- | :--- | :--- | :--- |
| `name` | String | Nombre de la carta. | `"Knight"`, `"Archers"`, `"Void"` |
| `id` | Number | Identificador único de la carta. | `26000000`, `28000023` |
| `elixirCost` | Number | Costo de elixir para usar la carta. | `3`, `5`, `1` |
| `rarity` | String | Rareza de la carta. | `"common"`, `"epic"`, `"rare"` |
| `type` | String | Tipo de carta (Tropa, Hechizo o Edificio). | `"troop"`, `"spell"`, `"building"` |
| `iconUrls` | Object | Objeto que contiene las URLs de las imágenes. | `{ "medium": "..." }` |
| `mobility` | String | Si la tropa se mueve por tierra, aire o no se aplica. | `"ground"`, `"air"`, `"both"` |
| `targets` | String | A qué tipo de unidades puede atacar la carta. | `"ground"`, `"air"`, `"both"` |
| `attack_type` | String | Tipo de ataque individual o en area. | `"single"`, `"splash"` |
| `hitpoints` | Number/Null | Puntos de vida base de la carta (o `null` si es hechizo). | `1766`, `787`, `null` |
| `usage` | Number | Porcentaje de uso o popularidad de la carta. | `8.88`, `4.0`, `0.8` |
| `hp_level` | String/Null | Nivel de puntos de vida. | `"medium"`, `null` |
| `groupCard` | Boolean | Indica si es una carta de grup. | `true`, `false` |
| `maxEvolutionLevel` | Number/Null | Nivel máximo de evolución de la carta. | `1`, `0`, `null` |

# Base de datos usada

El dataset de cartas de Clash Royale se adapta al modelo de **Base de Datos de Documentos**, que es el modelo central utilizado por **CouchDB**.

### 1. Características del Modelo

| Característica | Base de Datos de Documentos | Aplicación al Dataset |
| :--- | :--- | :--- |
| **Formato** | Utiliza JSON | El dataset ya está en formato **JSON**, el formato nativo para CouchDB. |
| **Atomicidad/Independencia** | **Documentos Auto-contenidos.** | Cada carta (`"Knight"`, `"Archers"`) es un documento independiente que contiene **toda** su información. |
| **Flexibilidad del Esquema** | **Schema-less (esquema flexible).** | No todas las cartas necesitan los mismos campos (ej. los hechizos no tienen `hitpoints`), lo cual es manejado naturalmente por el modelo de documentos. |

### 2. Estructura y Modelado

| Elemento NoSQL | Descripción en el Modelo |
| :--- | :--- |
| Base de Datos | `clash_royale_cards` (Contenedor de todos los documentos). |
| Documentos | Las 120 cartas (tropas, edificios y hechizos) son los documentos individuales. |
| Relaciones | Las relaciones son implícitas y se muestran atraves de los valores de los campos. |
| Diseño | Denormalización completa. Todos los datos de una carta están integrados en un solo documento para optimizar la velocidad de lectura|

# Herramientas Utilizadas

Se utilizaron las siguientes herramientas de línea de comandos y la interfaz web para preparar el dataset, realizar la ingesta y gestionar la base de datos CouchDB:

| Herramienta | Tipo | Propósito y Uso |
| :--- | :--- | :--- |
| jq | Procesador JSON de Línea de Comandos | Se utilizó para modificar el archivo JSON de origen (`clash_royale_cards_1.json`), transformando el array principal de `{"items": [...]}` al formato requerido por CouchDB: `{"docs": [...]}`. |
| cURL | Cliente de Transferencia de Datos | Herramienta esencial para interactuar con la API REST de CouchDB. Se utilizó para realizar la solicitud POST al endpoint `/_bulk_docs`, enviando el dataset transformado para la carga de documentos. |
| Fauxton | Interfaz Web de CouchDB | Se utilizó como herramienta gráfica para crear la base de datos (`clash_royale_cards`) y para la creación y gestión de las Vistas MapReduce (Documentos de Diseño), permitiendo la consulta avanzada de los datos. |

# Proceso de Importación de Datos
El proceso de importación implicó los siguientes pasos:

Descarga del archivo JSON con los datos de https://www.kaggle.com/datasets/niteshkakkar/clash-royal-cards-data
<img width="593" height="288" alt="image" src="https://github.com/user-attachments/assets/88293ba8-d28d-48d2-9ec9-c55865d08d5f" />

Adaptar el archivo JSON descarga borrando una coma demas al enlistar los datos y usando el comando jq "{ "docs": .items }" "clash_royale_cards_1.json" para cambiar la palabra items a docs

<img width="369" height="238" alt="image" src="https://github.com/user-attachments/assets/4d95a26c-b3f4-4f59-9183-0e03621d3b76" />

Pasar la salida de este comando a cURL por medio de un pipeline, leer la salida y crear todos los documentos JSON en la base de datos, para esto se uso el anterior comando y el comando curl -X POST por lo que el comando final usado es: 

jq "{ "docs": .items }" "clash_royale_cards_1.json" | curl -X POST "http://admin:root@localhost:5984/clash_royale_cards/_bulk_docs" -H "Content-Type: application/json" --data @- 

que como resultado muestra ok si se creo de forma exitosa algun documento
<img width="1457" height="127" alt="image" src="https://github.com/user-attachments/assets/3c7698ae-fde8-4d43-8c09-f02d003d0d3b" />

# Operaciones CRUD de CouchDB

Como estamos manejando nuestra base de datos con fauxtoneste realiza las siguientes operaciones a través de la API REST usando solicitudes HTTP.

## 1. Crear

CouchDB asigna el `_id` y el `_rev` automaticamente, fauxton seleccionamos "Create Documents" y llenamos el texto JSON del documento que deseemos agregar luego fauxton procesa la solicitud HTTP de la siguiente forma, el metodo de hacerlo desde HTTP es con **POST**

| # | Método | Endpoint de la API | Cuerpo de la Solicitud (JSON) |
| :--- | :--- | :--- | :--- |
| 1 | `POST` | `/clash_royale_cards` | `{"name": "Electro Wizard", "elixirCost": 4, "rarity": "legendary"}` |
| 2 | `POST` | `/clash_royale_cards` | `{"name": "Heal Spirit", "elixirCost": 1, "type": "spell"}` |
| 3 | `POST` | `/clash_royale_cards` | `{"name": "Tesla", "elixirCost": 4, "type": "building", "hitpoints": 1200}` |
| 4 | `POST` | `/clash_royale_cards` | `{"name": "P.E.K.K.A", "elixirCost": 7, "rarity": "epic"}` |
| 5 | `POST` | `/clash_royale_cards` | `{"name": "New Card 5", "elixirCost": 8, "rarity": "champion"}` |

## Ejemplo: 

<img width="621" height="115" alt="image" src="https://github.com/user-attachments/assets/f6369aee-dc9e-40c2-9531-b8f66b9baad8" />

## Resultado:

<img width="448" height="222" alt="image" src="https://github.com/user-attachments/assets/6c5f54fa-b6a2-4b5d-9951-ad19d6ad05fa" />

## 2. Leer

Los siguientes ejemplos que mostramos son de vistas creadas desde fauxton, para crear una hay que seleccionar el simbolo de mas al lado de Design Documents y seleccionar New view, el metodo de hacerlo desde HTTP es con **GET**

| # | Método | Endpoint de la API | Propósito de la Consulta |
| :--- | :--- | :--- | :--- |
| 1 | `GET` | `.../_design/cards/_view/by_type?key="troop"` | Recuperar todas las cartas de tipo "troop". |
| 2 | `GET` | `.../_design/cards/_view/by_usage_and_attack?startkey=["splash", 5.0]` | Recuperar todas las cartas de "splash" con un uso superior al 5.0%. |
| 3 | `GET` | `/_all_docs?limit=5` | Leer los primeros 5 documentos de la base de datos. |
| 4 | `GET` | `.../by_rarity_and_cost` | Leer la vista completa por rareza y costo. |
| 5 | `GET` | `.../by_rarity_and_cost?key=["rare", 4]` | Leer cartas de rareza "rare" con costo exacto de 4. |

### 1. Recuperar todas las cartas de tipo "troop".

<img width="369" height="134" alt="image" src="https://github.com/user-attachments/assets/d01e8324-2d5f-4880-a92e-28b5b7d305c4" />
<img width="auto" height="134" alt="image" src="https://github.com/user-attachments/assets/8ae4d670-5dbf-4cf0-b448-f7bb4eadeb95" />

### 2.Recuperar todas las cartas de "splash" con un uso superior al 5.0%.

<img width="auto" height="134" alt="image" src="https://github.com/user-attachments/assets/9f2c76c3-ef92-41fe-a2e2-9ac0eab02d5e" />
<img width="auto" height="134" alt="image" src="https://github.com/user-attachments/assets/b982c07e-9f89-4aaa-b199-c2080f2b5475" />

### 3.Leer los primeros 5 documentos de la base de datos.
En fauxton dentro de la funcion de la vista no podemos limitar nuestros resultados pero podemos seleccionar la opción  en el menu options para que use los limite al hacer la solicitud HTTP

<img width="" height="134" alt="image" src="https://github.com/user-attachments/assets/6432ff7d-2910-4154-b41e-f958259441a2" />
<img width="" height="300" alt="image" src="https://github.com/user-attachments/assets/d87a82f2-c091-4dd5-8bd0-2afa2a3fd6eb" />
<img width="1082" height="320" alt="image" src="https://github.com/user-attachments/assets/d3cb5f1d-05ae-46ba-8ffd-78eae31ce82d" />

### 4.Leer la vista completa por rareza y costo.

<img width="" height="134" alt="image" src="https://github.com/user-attachments/assets/e24f249b-3612-4490-b7a6-40333519eff1" />
<img width="" height="200" alt="image" src="https://github.com/user-attachments/assets/ef8d1031-3c04-4bc5-b93f-140d39957387" />

### 5.Leer cartas de rareza "rare" con costo exacto de 4.

<img width="350" height="136" alt="image" src="https://github.com/user-attachments/assets/4acc02fc-846e-434f-a82b-fed5f66e5eb1" />
<img width="600" height="134" alt="image" src="https://github.com/user-attachments/assets/583e7049-977f-4056-9806-3f30fee57696" />

## 3. Actualizar

Los siguientes ejemplos que mostramos son de actualizaciones desde fauxton, para actualizar un documento se selecciona el documento que quieres cambiar, se te abrira el documento con sus elementos y su `_rev` generada esta no debe de ser modificada, realizas los cambios nesesarios y lo guardas de ahi fauxton realiza la solicitud HTTP que es con `PUT` e incluye el `_rev` actual.

| # | Método | Endpoint de la API | Descripción del Cambio (JSON parcial) |
| :--- | :--- | :--- | :--- |
| 1 | `PUT` | `/clash_royale_cards/26000057` | Cambiar el costo de elixir a 3. (`... "elixirCost": 3, ...}`) |
| 2 | `PUT` | `/clash_royale_cards/26000057` | Ajustar puntos de vida a 700. (`... "hitpoints": 700, ...}`) |
| 3 | `PUT` | `/clash_royale_cards/26000057` | Agregar campo `is_evolved: true`. |
| 4 | `PUT` | `/clash_royale_cards/26000057` | Cambiar la rareza a "epic". (`... "rarity": "epic", ...}`) |
| 5 | `PUT` | `/clash_royale_cards/26000057` | Actualizar URL del icono. (`... "iconUrls": {... nueva URL ...}, ...}`) |

### Ejemplo cambiando los hitpoints a 700

<img width="1152" height="583" alt="image" src="https://github.com/user-attachments/assets/e4e6fe0d-68ea-40ff-9051-3cab730efd69" />
<img width="1137" height="585" alt="image" src="https://github.com/user-attachments/assets/ef4b5f1c-b01a-47b1-94cc-998652c820c2" />

### Resultado una nueva version del documento

<img width="1139" height="578" alt="image" src="https://github.com/user-attachments/assets/c3574ad0-5b54-487c-841e-c584c5eda952" />

## 4. Eliminar

Los siguientes ejemplos que mostramos son de eliminaciones se hicieron desde fauxton, para borrar un documento simplemente se debe de entrar a un documento y seleccionar la opción Delete y seleccionar que estas seguro ahi fauxton realiza la solicitud HTTP `DELETE` que incluye el `_rev` como parámetro de consulta.

| # | Método | Endpoint de la API | Notas |
| :--- | :--- | :--- | :--- |
| 1 | `DELETE` | `/clash_royale_cards/26000057?rev=1-cf490f23...` | Eliminar el documento de la Máquina Voladora. |
| 2 | `DELETE` | `/clash_royale_cards/26000010?rev=5-abcdef12...` | Eliminar una carta antigua (ID ficticio). |
| 3 | `DELETE` | `/clash_royale_cards/TEMP_CARD_001?rev=1-z9y8x7...` | Eliminar un documento creado temporalmente. |
| 4 | `DELETE` | `/clash_royale_cards/28000005?rev=2-a1b2c3d4...` | Eliminar un hechizo. |
| 5 | `DELETE` | `/clash_royale_cards/P.E.K.K.A_ID?rev=1-revpack...` | Eliminar el documento P.E.K.K.A (asumiendo que fue creado). |

### Ejemplo borrar carta agregada 'Electro Wizard'

<img width="1769" height="269" alt="image" src="https://github.com/user-attachments/assets/de496dc8-8ce1-409e-ba41-c8b2c752bb22" />

### Resultado

<img width="483" height="146" alt="image" src="https://github.com/user-attachments/assets/766025fd-67f3-4a89-8327-4f501494eff0" />


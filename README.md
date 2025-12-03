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

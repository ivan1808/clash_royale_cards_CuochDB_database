# Clash_royale_cards_CuochDB_database

# Dataset usado
El dataset que escojimos clash_royale_cards_1.json contiene información completa sobre las cartas del juego Clash Royale, incluyendo unidades (troops), edificios (buildings) y hechizos (spells). Consiste en una colección de objetos JSON que representan cada carta del juego con sus atributos principales como costo de elixir, rareza, tipo, estadísticas de combate y datos de uso.

Ejemplo de uno de los objetos JSON
<img width="1305" height="382" alt="image" src="https://github.com/user-attachments/assets/9ad7e0ee-b0d1-4f35-a0dc-1e056113d28a" />



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

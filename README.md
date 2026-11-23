# Segment Anything Model (SAM) — Guía Comprensiva

Bienvenido a este repositorio. Antes de pasar al código y a nuestro ejemplo práctico, esta guía explica de forma clara, completa y sin jerga innecesaria **qué es SAM (Segment Anything Model)**, **por qué es importante** y **cómo funciona por dentro**.

---

## 1. ¿Qué es SAM en palabras sencillas?

**SAM** (siglas en inglés de *Segment Anything Model*, o *"Modelo para Segmentar Cualquier Cosa"*) es un sistema de Inteligencia Artificial creado por **Meta AI** diseñado para **identificar y recortar con precisión cualquier objeto dentro de una imagen**.

Si tuviéramos que describirlo con una analogía cotidiana, SAM funciona como unas **tijeras mágicas inteligentes**: le muestras una fotografía, señalas algo con un clic (o dibujas un recuadro a su alrededor) y el modelo separa exactamente ese objeto del resto de la escena, borde por borde, en cuestión de milisegundos.

Lo más sorprendente es que **no necesita haber visto ese tipo de objeto antes** para poder recortarlo bien. Puede recortar desde un perro o un automóvil hasta una herramienta rara, una célula bajo un microscopio o una piedra en el fondo del mar.

---

## 2. ¿Qué significa "segmentar" una imagen?

En el mundo de la visión por computadora (la rama de la IA que enseña a las máquinas a "ver"), existen distintos niveles de detalle para entender una foto:

| Tarea | ¿Qué responde? | Ejemplo en una foto con un gato en un sofá |
| :--- | :--- | :--- |
| **Clasificación** | *¿Qué hay en la foto?* | "En esta imagen hay un gato." |
| **Detección de objetos** | *¿Dónde está aproximadamente?* | Dibuja un rectángulo alrededor del gato. |
| **Segmentación** | *¿Qué píxeles exactos pertenecen al objeto?* | Pinta únicamente la silueta exacta del gato, siguiendo sus orejas, bigotes y cola, sin tocar el sofá. |

El resultado de la segmentación se llama **máscara**: una especie de "plantilla" o "silueta" que indica exactamente qué parte de la imagen corresponde al objeto elegido y qué parte corresponde al fondo.

---

## 3. ¿Por qué SAM marcó un antes y un después?

Antes de que existiera SAM, crear una IA capaz de recortar objetos era un proceso lento y muy rígido:

* **El enfoque tradicional (especialistas rígidos):** Si querías un programa que recortara bicicletas, tenías que reunir miles de fotos de bicicletas, pedirle a personas que trazaran manualmente el borde de cada una y entrenar un modelo solo para eso. Si al día siguiente querías recortar tazas de café, ese modelo no servía: había que repetir el trabajo desde cero.
* **El enfoque de SAM (un modelo generalista):** En lugar de memorizar listas cerradas de objetos ("perro", "auto", "silla"), SAM aprendió el **concepto general de qué hace que algo sea un objeto**: cómo se comportan los bordes, las sombras, las texturas, las formas y la profundidad.

Gracias a esto, SAM tiene una capacidad llamada ***Zero-Shot*** (aprendizaje sin ejemplos previos): puedes mostrarle una foto de un objeto totalmente nuevo que jamás apareció en su entrenamiento y aun así sabrá exactamente dónde empieza y dónde termina.

---

## 4. ¿Cómo te comunicas con SAM? (Las indicaciones o *Prompts*)

Así como a un asistente de texto le das instrucciones escribiendo una pregunta, a SAM le das **pistas visuales** (llamadas *prompts*) para decirle qué te interesa recortar:

1. **Con un clic (puntos):**
   * **Punto positivo:** *"Recorta el objeto que está justo aquí."*
   * **Punto negativo:** *"Pero excluye esta parte"* (muy útil si el recorte incluyó algo extra que no querías).
2. **Con una caja (rectángulo):** Dibujas un recuadro rápido alrededor del objeto y SAM ajusta el recorte exactamente a su silueta real.
3. **Con un trazo aproximado:** Puedes darle una silueta borrosa o incompleta y SAM la pule hasta dejar los bordes limpios.
4. **Modo automático ("todo a la vez"):** Si no le das ninguna pista, SAM puede escanear toda la fotografía por su cuenta y separar automáticamente cada objeto que detecte, como si desmontara la foto en piezas de un rompecabezas.

---

## 5. ¿Cómo funciona SAM por dentro?

Aunque por detrás usa redes neuronales avanzadas, su arquitectura se puede entender fácilmente como un **equipo de tres especialistas** que trabajan en cadena:

```mermaid
flowchart LR
    A["Imagen original"] --> B["1. El Observador (Image Encoder)"]
    P["Tu pista: clic o caja"] --> C["2. El Intérprete (Prompt Encoder)"]
    B -->|"Mapa mental (calculado una vez)"| D["3. El Recortador (Mask Decoder)"]
    C -->|"Pista traducida"| D
    D --> E["Recorte final (Máscara + confianza)"]
```

### Paso 1: El Observador de Imágenes (*Image Encoder*)
Es el especialista que realiza el trabajo pesado. Mira la imagen completa una sola vez y construye un **"mapa mental"** muy detallado (técnicamente llamado *embedding*) donde resume formas, colores, contornos y relaciones entre partes de la foto.

* **¿Por qué esto es inteligente?** Porque este análisis profundo **solo se hace una vez por imagen**. Una vez creado el mapa mental, puedes hacer 10, 20 o 50 clics distintos sobre la misma foto sin tener que volver a analizarla desde cero.

### Paso 2: El Intérprete de Pistas (*Prompt Encoder*)
Mientras tú haces clic en un punto o dibujas un rectángulo, este especialista toma tu indicación y la traduce al mismo "idioma matemático" en el que está escrito el mapa mental de la imagen. Es un proceso prácticamente instantáneo.

### Paso 3: El Recortador Instantáneo (*Mask Decoder*)
Este especialista junta dos cosas: el mapa mental del Observador y tu pista traducida por el Intérprete. Con esa combinación, decide en milisegundos qué píxeles forman el objeto que señalaste y genera la **máscara** (el recorte final).

---

### ¿Qué pasa cuando una pista es ambigua?

Imagina que en una foto de una persona haces clic sobre su camisa. ¿Qué quieres recortar realmente?
* ¿Solo el botón donde cayó el clic?
* ¿La camisa completa?
* ¿A la persona entera?

Un sistema rígido tendría que adivinar y se equivocaría dos de cada tres veces. SAM resuelve esto de una manera muy natural: **propone varias opciones a la vez** (normalmente 3 niveles: un detalle pequeño, una parte mediana y el objeto completo) y acompaña cada propuesta con una **puntuación de confianza** que indica qué tan seguro está de que ese recorte tiene sentido.

---

## 6. ¿Cómo logró SAM aprender tanto?

Para dominar el arte de recortar cualquier cosa, SAM necesitaba practicar con una cantidad gigantesca de ejemplos. Meta creó un ciclo de entrenamiento llamado **Motor de Datos (*Data Engine*)** que evolucionó en tres etapas:

1. **Etapa asistida por humanos:** Al principio, personas reales recortaban objetos en fotografías con una ayuda básica del modelo, corrigiendo sus errores. Cada corrección hacía al modelo más preciso.
2. **Etapa semiautomática:** Cuando SAM mejoró, comenzó a recortar automáticamente los objetos más evidentes, mientras los humanos se enfocaban únicamente en los objetos difíciles o pequeños para enseñar al modelo a pulir sus fallas.
3. **Etapa totalmente automática:** Finalmente, SAM se volvió tan capaz que pudo analizar millones de fotografías por su cuenta y generar recortes de alta calidad sin intervención humana.

El resultado fue **SA-1B**, una biblioteca masiva de entrenamiento con **más de 11 millones de imágenes y más de 1.100 millones de recortes (máscaras)**, una escala cientos de veces mayor que la de bases de datos anteriores.

---

## 7. ¿Para qué se usa SAM en la vida real?

La capacidad de aislar objetos con facilidad tiene aplicaciones prácticas en muchísimas áreas:

* **Edición de fotos y diseño:** Eliminar fondos en un clic, cambiar el color de una prenda, borrar personas u objetos no deseados, o aplicar desenfoques selectivos.
* **Medicina e imágenes clínicas:** Ayudar a especialistas a delimitar órganos, tejidos o posibles anomalías en radiografías, resonancias o tomografías (siempre como herramienta de apoyo).
* **Agricultura y medio ambiente:** Analizar fotos de drones o satélites para medir hectáreas de cultivo, contar árboles, detectar zonas inundadas o mapear techos y edificios.
* **Robótica y vehículos autónomos:** Ayudar a un robot a distinguir exactamente dónde termina una caja que debe recoger, o a un vehículo a separar la carretera, los peatones y las señales.
* **Ciencia y biología:** Contar células bajo un microscopio, seguir fauna en cámaras de naturaleza o analizar partículas en laboratorio.

---

## 8. Evolución: SAM 1 vs. SAM 2 vs. SAM 3

Desde su lanzamiento original, Meta ha evolucionado SAM en tres generaciones muy claras:

| Versión | Qué analiza | Cómo le das instrucciones | Gran avance |
| :--- | :--- | :--- | :--- |
| **SAM 1** | Solo imágenes | Clics, cajas y máscaras | Primer recortador universal *Zero-Shot* para cualquier objeto en una foto. |
| **SAM 2** | Imágenes y **videos** | Clics, cajas y máscaras | Añadió memoria temporal para seguir un objeto frame a frame en video aunque se mueva o se oculte brevemente. |
| **SAM 3** | Imágenes y videos | **Texto**, ejemplos visuales, clics, cajas y máscaras | Entiende **conceptos en lenguaje natural** (ej. *"rayo"*, *"nieve"*, *"casco amarillo"*), encuentra **todas las instancias** a la vez y verifica si el objeto realmente existe (*Presence Head*) para reducir falsos positivos. |

---

## 9. Qué hace y qué NO hace SAM (Limitaciones importantes)

Para usar SAM correctamente, es muy importante entender sus límites:

* **SAM 1 y SAM 2 recortan formas, pero no les ponen nombre:** saben separar la silueta exacta de un perro, pero por sí solos no saben que se llama "perro". **SAM 3** superó esta limitación al entender texto y conceptos, aunque en clasificaciones muy sutiles (por ejemplo, distinguir entre distintos tipos de hielo o escarcha) sigue siendo muy útil combinarlo con un clasificador especializado (como una **CNN**).
* **Bordes ultrafinos o confusos:** Aunque es sorprendentemente bueno, puede tener dificultades con elementos muy delgados o difusos como cabellos sueltos, redes, humo, objetos transparentes (vidrio) o escenas con muy poca luz.
* **Exigencia de hardware:** Ejecutar el codificador visual es mucho más fluido con una tarjeta gráfica (GPU). En **SAM 3**, usar media precisión (`bfloat16`/`float16`) reduce el uso a ~**2 GB de VRAM** para imágenes estáticas, por lo que corre cómodamente en GPUs de consumo (6–8 GB o más).

---

## 10. Glosario rápido

Si escuchas estos términos en artículos o código sobre SAM, aquí tienes su traducción al español cotidiano:

| Término técnico | Significado sencillo |
| :--- | :--- |
| **Segmentación** | Separar una imagen píxel por píxel para aislar cada objeto con su contorno exacto. |
| **Máscara (*Mask*)** | La silueta final que representa qué píxeles pertenecen al objeto recortado. |
| **Prompt** | La pista o instrucción que le das al modelo (un clic, un recuadro o una frase de texto en SAM 3). |
| **Embedding** | El "mapa mental" numérico que el modelo construye para entender el contenido de la foto. |
| **Zero-Shot** | Capacidad de resolver una tarea con objetos nuevos sin haber sido reentrenado específicamente para ellos. |
| **Presence Head** | Módulo de SAM 3 que verifica si el concepto pedido realmente está en la imagen antes de dibujar una máscara. |
| **Modelo Fundacional** | Un modelo de IA entrenado a gran escala con conocimientos generales que sirve como base para múltiples tareas diferentes. |

---

## 11. Prototipo en este repositorio: Detector de Clima Explicable (SAM 3 + CNN)

Este repositorio implementa un prototipo en Jupyter Notebook ([notebooks/weather_sam3_cnn.ipynb](notebooks/weather_sam3_cnn.ipynb)) para **determinar el clima actual a partir de una fotografía** usando el dataset [Weather Image Recognition (Kaggle)](https://www.kaggle.com/datasets/jehanbhathena/weather-dataset) (6,862 imágenes en 11 categorías climáticas).

### ¿Cómo funciona nuestro prototipo?
1. **SAM 3 Zero-Shot:** consulta 11 conceptos climáticos reutilizando `get_vision_features()` una sola vez por foto para obtener puntajes de presencia y **máscaras visuales explicativas** (resaltando nieve, rayos, arcoíris, rocío, niebla, etc.).
2. **CNN (`ResNet18` con Transfer Learning):** se entrena en pocos minutos en GPU local para distinguir con alta precisión clases visualmente cercanas (`frost`, `glaze`, `rime`, `snow`).
3. **Fusión Híbrida (CNN + SAM 3):** combina ambas señales para lograr alta precisión **manteniendo la explicabilidad visual** de SAM 3.

### Guía rápida de instalación en local (Windows + GPU NVIDIA)

1. **Crea y activa un entorno virtual con Python 3.12+:**
   ```powershell
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1
   ```

2. **Instala PyTorch con soporte CUDA y el resto de dependencias:**
   ```powershell
   pip install torch torchvision --index-url https://download.pytorch.org/whl/cu126
   pip install -r requirements.txt
   ```

3. **Activa el acceso a `facebook/sam3` en Hugging Face:**
   * Acepta los términos del modelo en [huggingface.co/facebook/sam3](https://huggingface.co/facebook/sam3).
   * Inicia sesión desde tu terminal:
     ```powershell
     huggingface-cli login
     ```

4. **Ejecuta el notebook:**
   ```powershell
   jupyter notebook notebooks/weather_sam3_cnn.ipynb
   ```
   *(También puedes abrir `notebooks/weather_sam3_cnn.ipynb` directamente en VS Code seleccionando el intérprete `.venv`. El dataset de Kaggle se descargará automáticamente mediante `kagglehub`.)*

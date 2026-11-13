# 🌱 HortiAmiga

HortiAmiga es un asistente inteligente multimodal para la orientación y el cuidado básico de hortalizas en huertos urbanos. El sistema procesa consultas escritas en español y analiza fotografías de hojas de jitomate para ofrecer información preventiva en un lenguaje accesible.

El proyecto fue desarrollado para la materia **Desarrollo de Aplicaciones con Inteligencia Artificial** de la Universidad Tecmilenio. La versión actual integra procesamiento de lenguaje natural, clasificación supervisada de texto y una red neuronal convolucional con transferencia de aprendizaje.

> HortiAmiga es un prototipo académico. Las predicciones visuales son orientativas y no sustituyen un diagnóstico profesional de agronomía o fitosanidad.

## 🎯 Objetivo

Implementar y evaluar una primera versión multimodal de HortiAmiga que pueda:

- Clasificar intenciones conversacionales relacionadas con huertos urbanos.
- Responder consultas sobre cultivos, temporadas, riego, iluminación, sustratos y problemas frecuentes.
- Clasificar imágenes de hojas de jitomate mediante ResNet50.
- Combinar los resultados del componente textual y visual en una sola respuesta.

## 🧩 Problema atendido

Las personas que comienzan un huerto urbano suelen encontrar información dispersa o demasiado técnica sobre los cuidados de sus cultivos. También pueden tener dificultades para interpretar visualmente cambios en las hojas.

HortiAmiga organiza estas consultas en una interfaz conversacional y complementa la orientación escrita con un clasificador de imágenes especializado en cuatro condiciones de hojas de jitomate.

## 👥 Usuarios objetivo

Personas principiantes interesadas en iniciar o mantener un huerto urbano en balcones, ventanas, patios, azoteas o macetas.

## 🏗️ Arquitectura general

```text
                         ┌────────────────────────────┐
Mensaje escrito ────────►│ Clasificador de intención  │
                         │ TF-IDF + SVM lineal        │
                         └─────────────┬──────────────┘
                                       │
Usuario                                ├────► Integración ────► Respuesta
                                       │      multimodal        preventiva
                         ┌─────────────┴──────────────┐
Fotografía ─────────────►│ Clasificador visual       │
                         │ ResNet50                  │
                         └────────────────────────────┘
```

## 💬 Componente textual

### 📚 Conjunto de datos

Se construyó un conjunto balanceado de **250 mensajes**, con 25 ejemplos para cada clase:

| Intención | Descripción |
|---|---|
| `saludo` | Inicio de la conversación |
| `iniciar_huerto` | Primeros pasos, materiales y preparación |
| `recomendar_hortaliza` | Selección de cultivos según el espacio |
| `consultar_temporada` | Época o calendario de siembra |
| `consultar_riego` | Agua, frecuencia y humedad |
| `consultar_iluminacion` | Sol, sombra y horas de luz |
| `consultar_sustrato` | Tierra, composta y drenaje |
| `consultar_problema` | Síntomas y plagas frecuentes |
| `despedida` | Cierre de la conversación |
| `fallback` | Consultas fuera del dominio |

El conjunto se encuentra en [`datos/intenciones_hortiamiga.csv`](datos/intenciones_hortiamiga.csv).

### 🧹 Preprocesamiento

El pipeline textual aplica:

1. Conversión a minúsculas.
2. Normalización Unicode y eliminación de acentos.
3. Tokenización con `RegexpTokenizer`.
4. Eliminación de stopwords en español.
5. Stemming con `SnowballStemmer`.
6. Representación mediante unigramas y bigramas.

Se conservaron negaciones como `no`, `ni` y `sin` porque pueden modificar el significado de una consulta.

### ⚖️ Modelos comparados

- Bag of Words + Naive Bayes multinomial.
- TF-IDF + SVM lineal.

El conjunto se dividió de manera estratificada en 80 % para entrenamiento y 20 % para prueba. También se utilizó validación cruzada estratificada de cinco particiones.

### 📊 Resultados textuales

| Modelo | Accuracy | Precision macro | Recall macro | F1 macro |
|---|---:|---:|---:|---:|
| **TF-IDF + SVM lineal** | **0.640** | **0.691** | **0.640** | **0.648** |
| BoW + Naive Bayes | 0.620 | 0.618 | 0.620 | 0.598 |

Validación cruzada:

| Modelo | F1 macro promedio | Desviación estándar |
|---|---:|---:|
| **TF-IDF + SVM lineal** | **0.686** | **0.042** |
| BoW + Naive Bayes | 0.607 | 0.048 |

Se seleccionó **TF-IDF + SVM lineal** por su mejor F1 macro y mayor estabilidad entre particiones. Las principales confusiones se presentaron entre `iniciar_huerto` y `recomendar_hortaliza`. La clase `fallback` también resultó difícil porque agrupa mensajes de muchos temas diferentes.

## 👁️ Componente visual

### 🖼️ Conjunto de imágenes

El componente visual utiliza un subconjunto de [PlantVillage](https://www.tensorflow.org/datasets/catalog/plant_village), descargado mediante TensorFlow Datasets.

Se seleccionaron hasta 800 imágenes de cada clase:

| Clase | Imágenes seleccionadas |
|---|---:|
| Hoja saludable | 800 |
| Tizón temprano | 800 |
| Tizón tardío | 800 |
| Mancha bacteriana | 800 |
| **Total** | **3,200** |

La división utilizada fue:

- 70 % entrenamiento: 2,240 imágenes.
- 15 % validación: 480 imágenes.
- 15 % prueba: 480 imágenes.

### 🔧 Preparación de imágenes

Las imágenes se redimensionan a 224 × 224 píxeles. Durante el entrenamiento se aplican volteo horizontal, rotaciones moderadas, zoom y variaciones de contraste. El aumento de datos no se aplica a validación o prueba.

### 🧠 ResNet50

El modelo se construyó con ResNet50 preentrenada en ImageNet:

```text
Imagen 224 × 224 × 3
        ↓
Aumento de datos
        ↓
Preprocesamiento de ResNet50
        ↓
ResNet50 sin clasificador original
        ↓
GlobalAveragePooling2D
        ↓
Dropout 0.30
        ↓
Dense con cuatro salidas y softmax
```

El entrenamiento se realizó en dos etapas:

1. Base ResNet50 congelada y entrenamiento de la nueva cabeza clasificadora.
2. Fine-tuning de las últimas 30 capas con una tasa de aprendizaje de `1e-5`.

### 📈 Resultados visuales finales

El modelo clasificó correctamente **464 de 480 imágenes** de prueba.

| Métrica | Resultado |
|---|---:|
| Accuracy | 0.9667 |
| Precision macro | 0.9664 |
| Recall macro | 0.9665 |
| F1 macro | 0.9658 |
| F1 ponderado | 0.9666 |

Resultados por clase:

| Clase | Precision | Recall | F1 | Soporte |
|---|---:|---:|---:|---:|
| Saludable | 0.9224 | 1.0000 | 0.9596 | 107 |
| Tizón temprano | 0.9643 | 0.9153 | 0.9391 | 118 |
| Tizón tardío | 0.9787 | 0.9857 | 0.9822 | 140 |
| Mancha bacteriana | 1.0000 | 0.9652 | 0.9823 | 115 |

La principal dificultad ocurrió en hojas con síntomas leves. Siete imágenes de tizón temprano fueron clasificadas como saludables y tres como tizón tardío.

## 🔗 Integración multimodal

La función principal acepta texto, imagen o ambos elementos:

```python
procesar_consulta(texto=None, ruta_imagen=None)
```

- Solo texto: clasifica la intención y genera una respuesta conversacional.
- Solo imagen: clasifica la condición visual y comunica el resultado.
- Texto e imagen: combina la intención con la predicción visual.

Ejemplo de una prueba con una fotografía externa:

```text
Puedo darte una orientación preventiva. Si tienes una hoja de jitomate,
adjunta una fotografía. Resultado visual: tizón temprano (73.6 %).
La imagen se parece a casos de tizón temprano. Retira hojas muy afectadas,
evita mojar el follaje y vigila la propagación. El análisis visual es
orientativo y no constituye un diagnóstico profesional.
```

El 73.6 % representa el puntaje de confianza del modelo, no la certeza de un diagnóstico.

## 🛠️ Tecnologías utilizadas

- Python 3.10 o superior
- Google Colab
- NLTK
- pandas y NumPy
- scikit-learn
- TensorFlow y Keras
- TensorFlow Datasets
- Matplotlib y Seaborn
- ResNet50
- GitHub

## ▶️ Ejecución en Google Colab

Se recomienda ejecutar el notebook multimodal en Colab con GPU.

1. Abre [`HortiAmiga_multimodal.ipynb`](HortiAmiga_multimodal.ipynb) en Google Colab.
2. Selecciona **Entorno de ejecución → Cambiar tipo de entorno de ejecución**.
3. Selecciona una GPU disponible.
4. Ejecuta las celdas desde el inicio.
5. Carga `datos/intenciones_hortiamiga.csv` cuando el notebook lo solicite.
6. Espera la descarga de PlantVillage y los pesos de ResNet50.
7. Ejecuta el entrenamiento con la base congelada.
8. Ejecuta el fine-tuning.
9. Conserva las métricas, matrices, curvas y ejemplos incorrectos.

PlantVillage descarga aproximadamente 828 MiB. Conviene conservar varios gigabytes libres en la sesión de Colab.

La instalación inicial incluye `importlib_resources` para evitar problemas de compatibilidad al descargar PlantVillage:

```python
!pip -q install -U tensorflow-datasets importlib_resources nltk seaborn scikit-learn joblib
```

## ⚠️ Limitaciones

- El conjunto textual contiene solamente 25 mensajes por clase.
- `fallback` no representa un tema semántico único.
- El modelo visual solo reconoce cuatro condiciones de hojas de jitomate.
- PlantVillage contiene fondos relativamente controlados.
- El desempeño puede disminuir con fotografías reales de celular.
- Las salidas softmax no son probabilidades diagnósticas calibradas.
- El sistema todavía no conserva contexto entre varios turnos.
- La orientación generada no sustituye la valoración profesional.

## 🔮 Mejoras futuras

- Recopilar mensajes escritos por usuarios reales.
- Ampliar las frases de intenciones con menor recall.
- Activar `fallback` mediante un umbral de confianza.
- Incorporar fotografías de huertos reales.
- Agregar más cultivos y condiciones visuales.
- Calibrar los puntajes del clasificador visual.
- Incorporar memoria conversacional.
- Desarrollar una interfaz web accesible desde dispositivos móviles.

## 📌 Archivos principales

- [`HortiAmiga_chatbot_reglas.ipynb`](HortiAmiga_chatbot_reglas.ipynb): prototipo basado en reglas del Avance 1.
- [`HortiAmiga_multimodal.ipynb`](HortiAmiga_multimodal.ipynb): clasificación textual, ResNet50 e integración multimodal.
- [`datos/intenciones_hortiamiga.csv`](datos/intenciones_hortiamiga.csv): conjunto textual balanceado.

## ✍️ Autor

**Nombre:** Jorge Pinales

**Matrícula:** 7292050

**Institución:** Universidad Tecmilenio  

**Materia:** Desarrollo de Aplicaciones con Inteligencia Artificial  

## 📚 Referencias

- He, K., Zhang, X., Ren, S. y Sun, J. (2016). *Deep Residual Learning for Image Recognition*. https://arxiv.org/abs/1512.03385
- Hughes, D. P. y Salathé, M. (2015). *An Open Access Repository of Images on Plant Health to Enable the Development of Mobile Disease Diagnostics*. https://arxiv.org/abs/1511.08060
- Keras. *ResNet and ResNetV2*. https://keras.io/api/applications/resnet/
- Natural Language Toolkit Project. https://www.nltk.org/
- Scikit-learn. https://scikit-learn.org/stable/
- TensorFlow Datasets. *PlantVillage*. https://www.tensorflow.org/datasets/catalog/plant_village

## 🎓 Uso académico

Este proyecto fue desarrollado con fines educativos. Los resultados no deben utilizarse como diagnóstico profesional ni como única base para aplicar tratamientos a un cultivo.

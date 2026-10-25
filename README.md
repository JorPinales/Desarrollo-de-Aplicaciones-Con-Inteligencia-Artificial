# 🌱 HortiAmiga

**HortiAmiga** es un chatbot basado en reglas que brinda orientación básica a personas interesadas en comenzar y cuidar un huerto urbano.

Este proyecto corresponde al primer avance de la materia **Desarrollo de Aplicaciones con Inteligencia Artificial**. Fue desarrollado con Python y NLTK como base para incorporar posteriormente técnicas más avanzadas de procesamiento de lenguaje natural, clasificación de imágenes e integración web.

## Problema

Las personas que comienzan un huerto urbano suelen tener dudas sobre qué hortalizas cultivar, cuándo sembrarlas, cuánta luz necesitan, cómo regarlas y qué hacer ante problemas comunes.

Aunque existe información en internet, suele encontrarse dispersa o utilizar términos técnicos. Esto dificulta que una persona sin experiencia encuentre una orientación sencilla y rápida.

HortiAmiga busca atender esta necesidad mediante respuestas automáticas sobre cuidados básicos de hortalizas y organización inicial de huertos urbanos.

## Objetivo

Diseñar e implementar un chatbot funcional basado en reglas que identifique intenciones y entidades en mensajes escritos en español para proporcionar orientación básica sobre huertos urbanos.

## Usuarios objetivo

Personas interesadas en iniciar o mantener un huerto urbano, especialmente principiantes que cultivan en espacios pequeños como:

- Balcones
- Patios
- Ventanas
- Azoteas
- Macetas

## Alcance

El chatbot puede atender consultas relacionadas con:

- Inicio de un huerto urbano
- Recomendación de hortalizas
- Temporadas de siembra
- Riego
- Iluminación
- Sustratos
- Problemas frecuentes en los cultivos
- Saludos y despedidas

El prototipo ofrece recomendaciones generales y preventivas. No sustituye la valoración de especialistas en agricultura, agronomía o control profesional de plagas.

## Intenciones conversacionales

El chatbot reconoce las siguientes intenciones:

| Intención | Descripción |
|---|---|
| `saludo` | Reconoce saludos del usuario |
| `iniciar_huerto` | Orienta sobre los primeros pasos para crear un huerto |
| `recomendar_hortaliza` | Recomienda cultivos según el espacio disponible |
| `consultar_temporada` | Informa sobre temporadas de siembra |
| `consultar_riego` | Proporciona recomendaciones básicas de riego |
| `consultar_iluminacion` | Orienta sobre las necesidades de luz |
| `consultar_sustrato` | Brinda información sobre sustratos |
| `consultar_problema` | Atiende síntomas y problemas frecuentes |
| `despedida` | Reconoce el cierre de la conversación |
| `fallback` | Responde cuando el mensaje no pertenece al alcance |

## Entidades reconocidas

HortiAmiga puede identificar algunos datos dentro del mensaje:

- **Cultivo:** jitomate, lechuga, rábano, cilantro y albahaca.
- **Espacio:** balcón, ventana, patio, azotea y maceta.
- **Iluminación:** sol directo, semisombra y sombra.
- **Problema:** hojas amarillas, manchas, marchitez, crecimiento lento, pulgón, mosca blanca y cochinilla.

Ejemplo:

> Mi albahaca tiene hojas amarillas.

El sistema puede detectar:

- Intención: `consultar_problema`
- Cultivo: `albahaca`
- Problema: `hojas amarillas`

## Arquitectura conceptual

El funcionamiento general del asistente es:

```text
Usuario
   ↓
Mensaje escrito
   ↓
Preprocesamiento con NLTK
   ↓
Detección de intención
   ↓
Extracción de entidades
   ↓
Consulta de la base de conocimiento
   ↓
Generación de respuesta
   ↓
Usuario

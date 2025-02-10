# Pre-entrega-2


Introducción:

Nombre del proyecto: EvaluAI: Sistema Inteligente de Evaluación Personalizada

Presentación del problema a abordar: 

Los modelos de evaluación, en cualquier contexto académico, presentan varios retos que afectan tanto a estudiantes como a docentes. Por un lado, representan una alta carga de trabajo en la creación y calificación de evaluaciones, y por otro, pueden generar resultados que no reflejan realmente el nivel de aprendizaje del estudiante. Esto hace que el proceso de evaluación no siempre cumpla con su objetivo de guiar y fortalecer el aprendizaje.

Desarrollo de la propuesta de solución: https://docs.google.com/presentation/d/1tBzzcWsBP75z4YzUfANiHHSaVlYiCEyAuHfUPpCEUeA/edit?usp=sharing

Justificación de la viabilidad del proyecto: 

https://docs.google.com/document/d/1ox35gb39z97HDUgE6H1lNncJlEBfzp-L6LBJw2pXW58/edit?tab=t.0

Objetivos: 

El proyecto busca automatizar la evaluación académica de niños de entre 10 y 15 años mediante IA. Para ello, realiza las siguientes tareas:

1) Carga un documento de referencia proporcionado por el profesor.
2) Solicita información al estudiante sobre su edad y un tema de interés para contextualizar la evaluación.
3) Genera una pregunta basada en el documento, adaptada al contexto del tema y la edad del estudiante.
4) Evalúa la respuesta del estudiante y asigna una calificación binaria (10 puntos si es correcta, 0 si es incorrecta).
5) Genera una imagen personalizada con la calificación obtenida, representada con el tema elegido.

El enfoque es proporcionar una evaluación inmediata, personalizada y visualmente atractiva, facilitando el aprendizaje infantil a través de IA generativa.


Metodología: 

1. One-shot Prompting (Predominante)
Este método se usa en:

- Generación de la pregunta: Se le da un ejemplo implícito en la estructura del prompt, asegurando que la pregunta sea clara y tenga una respuesta numérica.
- Evaluación de la respuesta: Se le indica explícitamente al modelo que devuelva solo "10" o "0", minimizando respuestas ambiguas.
- Generación de la imagen: Se estructura la descripción de la imagen en un solo intento para obtener un resultado directo.

2. Contextual Prompting
Se emplea al personalizar la pregunta con base en:

- La información extraída del documento .txt.
- La edad del estudiante, asegurando una dificultad adecuada.
- El tema elegido por el estudiante para hacer la evaluación más atractiva.

3. Instruction-Based Prompting
- Se le dan instrucciones claras y detalladas al modelo sobre su rol (ejemplo: "Eres un profesor especializado en evaluación infantil").
- Se delimita el tipo de respuesta esperada para evitar desviaciones.

Implementación [CODIGO]: 

pip install openai==0.28
import openai
import requests
from io import BytesIO
from PIL import Image
from google.colab import files
import re

# Configurar la API de OpenAI (debes agregar tu clave aquí)
openai.api_key = "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"

# Función para cargar el archivo .txt
def cargar_documento():
    print("Sube un documento .txt para analizar el contexto")
    uploaded = files.upload()
    for filename in uploaded.keys():
        with open(filename, "r", encoding="utf-8") as file:
            return file.read()

# Función para preguntar datos al estudiante
def preguntar_estudiante():
    nombre = input("¿Cuál es tu nombre? ")
    tema = input("¿Sobre qué serie/libro/personaje histórico/artista quieres que se realice la evaluación? ")
    edad = input("¿Cuál es tu edad? ")
    return nombre, tema, edad

# Función para generar la evaluación
def generar_evaluacion(tema, edad, documento):
    prompt = f"""Crea una única pregunta basada en el siguiente contenido:

    {documento}

    La pregunta debe estar contextualizada en el tema {tema} y adecuada para un niño de {edad} años. 
    La respuesta debe ser un número o una cantidad específica, sin ambigüedades."""

    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": "Eres un profesor experto en evaluación infantil."},
            {"role": "user", "content": prompt}
        ]
    )
    return response["choices"][0]["message"]["content"].strip()

# Función para evaluar la respuesta
def evaluar_respuesta(pregunta, respuesta_estudiante):
    prompt = f"""Basándote en la siguiente pregunta:
    
    {pregunta}
    
    La respuesta del estudiante fue: "{respuesta_estudiante}".
    
    Extrae solo el número correcto de la respuesta esperada y compáralo con la respuesta del estudiante. Si la respuesta es exacta, asigna 10 puntos. 
    Si no lo es, asigna 0 puntos.
    Devuelve solo el número 10 o 0, sin explicaciones adicionales.
    La forma en la que se debe evaluar es basado en la respuesta ejemplo: 'Tengo 3 cajas con 8 manzanas cada caja 
    ¿Cuantas manzanas totales tengo?' la respuesta esperada es '24', la respuesta del estudiante puede ser '24' o '24 manzanas' en cualquiera de los dos casos la respuesta es correcta por que la multiplicación es correcta, solo debe calificarse como incorrecto si y solo si el número compartido por el estudiante es diferente al esperaro
    es muy importante no compartir la respuesta
    """

    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": "Eres un profesor automatizado de matemáticas para niños."},
            {"role": "user", "content": prompt}
        ]
    )
    puntaje = int(re.findall(r'\d+', response["choices"][0]["message"]["content"])[0])
    return puntaje

# Función para generar la imagen
def generar_imagen(tema, puntaje,edad):
    prompt_imagen = f"{tema} sosteniendo un papel con la calificación {puntaje}/10 en su mano, estilo de dibujo mas acorde a la {edad} del estudiante"
    response_imagen = openai.Image.create(
        prompt=prompt_imagen,
        size="1024x1024",
        model="dall-e-3"
    )
    
    url_imagen = response_imagen['data'][0]['url']
    imagen = Image.open(BytesIO(requests.get(url_imagen).content))
    imagen.save("imagen_generada.png")
    print(f"Haz click en la siguiente URL para ver tu calificación: {url_imagen}")

# Flujo principal
def main():
    documento = cargar_documento()
    nombre, tema, edad = preguntar_estudiante()
    
    print(f"Hola {nombre}, prepararemos una evaluación sobre {tema}.")
    pregunta = generar_evaluacion(tema, edad, documento)
    print("Aquí está tu evaluación:")
    print(pregunta)
    
    respuesta_estudiante = input("Tu respuesta: ")
    puntaje = evaluar_respuesta(pregunta, respuesta_estudiante)
    print(f"Tu calificación final es {puntaje}/10")
    
    generar_imagen(tema, puntaje,edad)

if __name__ == "__main__":
    main()


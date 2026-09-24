# Agente CV de Cesar

Agente conversacional que responde preguntas sobre la formación, habilidades, proyectos y logros de Cesar Abraham Sireno Ojeda.

**Agente desplegado:** https://agente-cv-cesar.onrender.com  
**Página para probar la API:** https://agente-cv-cesar.onrender.com/docs

## ¿Qué hace?

Permite consultar mi trayectoria mediante preguntas escritas. Por ejemplo, puede explicar cómo funcionaba mi proyecto integrado con Jira, qué es AMAEL o qué tecnologías he utilizado.

El agente utiliza la información de `datos_cv.txt` para responder. Si le preguntan por un dato que no está documentado, sus instrucciones le indican que lo reconozca en vez de inventarlo.

## ¿Cómo funciona?

1. Una persona escribe una pregunta en la página de pruebas `/docs`, o un cliente envía la pregunta mediante una petición HTTP a `POST /v1/responses`.
2. `main.py`, que es el servidor creado con FastAPI, recibe la petición. Extrae la pregunta y lee el archivo `datos_cv.txt`.
3. `main.py` utiliza la biblioteca de OpenAI para enviar al modelo `gpt-4.1-mini` la pregunta, los datos del CV y las instrucciones sobre cómo responder. 
La conexión utiliza la clave configurada en `OPENAI_API_KEY`.
4. OpenAI devuelve el texto generado a `main.py`. El servidor lo coloca en la respuesta de `/v1/responses` y se lo entrega a quien hizo la pregunta.

La petición puede contener una sola pregunta o una lista de mensajes anteriores. Si se envían los mensajes anteriores, el agente puede entender una pregunta de seguimiento como "¿Qué tecnologías utilizó en ese proyecto?".

El servidor no guarda las conversaciones.
La aplicación que utiliza el agente debe enviar el historial cuando quiera continuar una conversación.

El endpoint admite dos formas de entrega: una respuesta completa en JSON o una respuesta por partes cuando la petición incluye `stream: true`.

## Archivos principales

- `main.py`: contiene el servidor, la conexión con OpenAI y el endpoint del agente.
- `datos_cv.txt`: contiene la información profesional que utiliza el agente.
- `requirements.txt`: indica las bibliotecas de Python necesarias.
- `.gitignore`: evita subir archivos locales que no deben formar parte del repositorio.

## Cómo ejecutarlo en Windows

Necesitas Python, Git y una clave de API de OpenAI.

### 1. Descargar el proyecto

Abre PowerShell y ejecuta:

```powershell
git clone https://github.com/cesarsempai/agente_cv_cesar.git
cd agente_cv_cesar
```

### 2. Crear un entorno virtual

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Si PowerShell impide activar el entorno virtual, ejecuta primero:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned
```

Después repite el comando de activación.

### 3. Instalar las dependencias

```powershell
python -m pip install -r requirements.txt
```

### 4. Configurar la clave de OpenAI

El programa necesita la variable de entorno `OPENAI_API_KEY`. 
Puedes configurarla en la terminal actual con estos comandos:

```powershell
$secreto = Read-Host "Pega tu clave de OpenAI" -AsSecureString
$env:OPENAI_API_KEY = [System.Net.NetworkCredential]::new("", $secreto).Password
```

Al escribir la clave, los caracteres no se mostrarán normalmente en pantalla. **No escribas una clave real en este README, en `main.py` ni en un archivo que vayas a subir a GitHub.**

Esta configuración de la terminal no se conserva al cerrar esa ventana de PowerShell.

### 5. Iniciar el servidor

```powershell
python -m uvicorn main:app --reload
```

Abre `http://127.0.0.1:8000/docs` para probar la API. Para detener el servidor, vuelve a la terminal y pulsa `Ctrl+C`.

## Cómo probarlo

En `http://127.0.0.1:8000/docs`, abre `POST /v1/responses`, pulsa **Try it out** y escribe en **Request body**:

```json
{
  "input": "¿Cómo funcionaba el proyecto integrado con Jira?"
}
```

Pulsa **Execute**. Si la petición se procesa correctamente, aparecerá el código `200`. Dentro de **Response body**, el campo `text` contiene la respuesta del agente.

También puedes comprobar cómo trata un dato no documentado:

```json
{
  "input": "¿En qué empresa trabajó Cesar en 2023?"
}
```

En este caso, el agente debe indicar que no tiene información para afirmar ese empleo.

Para comprobar que el endpoint acepta respuestas por partes, puedes enviar:

```json
{
  "input": "¿Qué es AMAEL?",
  "stream": true
}
```

En ese caso, la respuesta se entrega como una secuencia de eventos en lugar de un único objeto JSON.

La ruta `GET /health` permite comprobar si el servidor está activo. Devuelve `{"status":"ok"}`.


## Decisiones técnicas

**FastAPI.** Elegí FastAPI para crear un endpoint HTTP y contar con `/docs`, una página desde la que puedo revisar y probar las peticiones.

**Archivo `datos_cv.txt`.** Separé los datos de mi trayectoria del código, así puedo revisar el contenido del perfil sin modificar la lógica del servidor.

**Modelo `gpt-4.1-mini`.** Utilizo un modelo de lenguaje para redactar respuestas naturales a partir de las preguntas y los datos del perfil. 
El código incluye instrucciones para responder en español, distinguir proyectos académicos de experiencia laboral y reconocer cuando falta información.

**Historial enviado en cada petición.** El servidor no almacena conversaciones, esto simplifica la aplicación: quien envía las preguntas también envía los mensajes anteriores cuando necesita hacer una pregunta de seguimiento.

**Endpoint `/v1/responses`.** Devuelve respuestas completas y admite respuestas por partes. 
Esto permite que otras aplicaciones se conecten con el agente mediante peticiones HTTP.

## Despliegue y operación

El servidor está desplegado como un servicio web de Python en Render. 

Para iniciarlo allí se utiliza:
```text
python -m uvicorn main:app --host 0.0.0.0 --port $PORT
```

Render instala las dependencias desde `requirements.txt`.
La variable `OPENAI_API_KEY` se configura en el entorno del servicio y no se guarda en el repositorio.

Puedo revisar los registros de despliegue y las peticiones desde el panel de Render. 
La ruta `/health` sirve para comprobar que la aplicación está activa. 
En el plan gratuito, el servicio puede suspenderse tras un periodo de inactividad, por lo que la primera petición posterior puede tardar más.

## Seguridad y límites

El repositorio no incluye claves de API. `.gitignore` excluye `.env`, el entorno virtual y el archivo local utilizado para guardar la clave.

El agente recibe instrucciones para usar únicamente la información del perfil y no inventar datos, estas instrucciones ayudan a orientar sus respuestas, pero no garantizan que un modelo nunca cometa errores.

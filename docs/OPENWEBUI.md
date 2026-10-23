# Open WebUI y voz

## Versiones y nombres de modelo

La instalación local consultada el 22 de octubre de 2026 tiene Open WebUI **0.12.3**. En la interfaz, el modelo aparece con el nombre **Sakura**. Ese nombre visible es una etiqueta; el identificador del modelo/proveedor LLM que la etiqueta apunta no se pudo confirmar en los archivos de proyecto disponibles, así que no se inventa aquí.

Qwen3-TTS Fast es el servicio de voz independiente. Su modelo reportado en los registros/configuración es `qwen3-tts-clone-0.6b`. Open WebUI lo llama mediante un endpoint compatible con OpenAI; no es el modelo LLM que genera la respuesta.

## Arranque manual de Open WebUI

Abre PowerShell en el directorio de la instalación de Open WebUI. Ajusta las rutas al equipo local; conserva la ruta de datos y la misma clave secreta entre reinicios:

```powershell
Set-Location '<carpeta-de-instalacion-de-Open-WebUI>'

$env:DATA_DIR = '<carpeta-de-instalacion-de-Open-WebUI>\data'
$env:WEBUI_SECRET_KEY = (Get-Content -LiteralPath '<archivo-local-de-WEBUI_SECRET_KEY>' -Raw).Trim()

$env:WHISPER_VAD_FILTER = 'true'
$env:WHISPER_LANGUAGE = 'es'
$env:WHISPER_MULTILINGUAL = 'true'
$env:WHISPER_COMPUTE_TYPE = 'int8'

$env:PYTHONUTF8 = '1'
$env:PYTHONIOENCODING = 'utf-8:backslashreplace'
$env:FROM_INIT_PY = 'true'
$env:WEBUI_URL = 'https://<IP-o-host-del-PC>:8081'
$env:CORS_ALLOW_ORIGIN = $env:WEBUI_URL

& '.\Scripts\python.exe' '-X' 'utf8' '.\run_open_webui_https.py'
```

La instalación que se usó tenía un lanzador `run_open_webui_https.py` que inicia Uvicorn en `0.0.0.0:8081` con `tls/server.crt` y `tls/server.key`. Mantén el proceso de PowerShell abierto; `Ctrl+C` lo detiene. La clave secreta, la carpeta `data`, `server.key` y cualquier copia de la base de datos son privadas y no pertenecen al repositorio.

`PYTHONUTF8`, `PYTHONIOENCODING` y `-X utf8` se añadieron para que Whisper/Python pueda procesar caracteres en español sin errores de codificación `charmap`. `WHISPER_LANGUAGE=es` fija el idioma; `WHISPER_VAD_FILTER`, `WHISPER_MULTILINGUAL` y `WHISPER_COMPUTE_TYPE=int8` son los ajustes de transcripción que se estaban usando.

## No perder perfil, chats ni ajustes

Open WebUI guarda los datos de usuario en `DATA_DIR`. Si se inicia con una carpeta diferente o vacía, puede aparecer de nuevo el formulario de creación de la cuenta administradora aunque los datos anteriores sigan en otra ubicación. Verifica `DATA_DIR` antes de crear otra cuenta. No borres la carpeta `data` ni regeneres `WEBUI_SECRET_KEY` de forma accidental.

## TTS es opcional: configurar Qwen3-TTS u otro proveedor

SakuraWebApp no incluye ni requiere un motor TTS específico. El texto a voz se configura en Open WebUI; puedes usar cualquier proveedor compatible con la versión de Open WebUI que tengas, o dejar la función desactivada. Revisa el endpoint, modelo, credenciales y requisitos de red del proveedor elegido. El ejemplo siguiente documenta Qwen3-TTS Fast porque es el que se usó en esta instalación; puedes omitirlo si usarás otro servicio.

### Ejemplo: Qwen3-TTS Fast

En una segunda terminal de Windows, inicia el servidor WSL y deja la ventana abierta mientras uses voz:

```powershell
wsl.exe -d Ubuntu -- /home/<usuario-linux>/qwen3-tts-fast-env/bin/python /mnt/d/Sakura/Voz/qwen3-tts/server_fast.py
```

El proceso reporta Uvicorn en `127.0.0.1:5052`. En los registros observados, las llamadas a `POST /v1/audio/speech` devolvieron `200 OK`.

En la configuración de Text-to-Speech de Open WebUI:

- Proveedor/engine: API compatible con OpenAI.
- Base URL: `http://127.0.0.1:5052/v1` si Windows puede alcanzar el puerto local de WSL.
- Modelo: `qwen3-tts-clone-0.6b`.
- La API Fast observada ignora los valores de voz `alloy` y modelo enviados por algunos clientes y utiliza su configuración de clonación/servidor.

Si no hay audio, verifica cada tramo: proceso Qwen vivo, endpoint alcanzable desde Windows, configuración TTS de Open WebUI y volumen multimedia del teléfono. La voz entrante (Whisper) y la voz saliente (Qwen3-TTS) son componentes separados.

## Variables de URL y CORS

`WEBUI_URL` y `CORS_ALLOW_ORIGIN` deben usar el mismo origen que abre el teléfono, incluido `https`, host y puerto. En la app pública, la dirección es una propiedad de Gradle y se pasa con `-PSAKURA_WEBUI_URL=...`. No uses `localhost`/127.0.0.1 para el teléfono: en el teléfono significaría el propio teléfono, no Windows.

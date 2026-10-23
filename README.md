# SakuraWebApp

Guía de operación para una app Android nativa que abre Open WebUI dentro de un WebView. La app conserva la sesión del WebView y se conecta al servidor que tú alojas.

> Este repositorio contiene documentación. No incluye el código de la app, APK, base de datos de Open WebUI, certificados ni claves privadas.

## Componentes

- **SakuraWebApp:** app Android con WebView, JavaScript, almacenamiento web y cookies habilitados.
- **Open WebUI:** servidor local que sirve la interfaz y conserva usuarios, perfiles, chats y ajustes en su directorio de datos.
- **Qwen3-TTS Fast:** servicio opcional de voz, iniciado desde WSL y expuesto localmente en el puerto `5052`.
- **Versión local documentada:** SakuraWebApp `0.1.7`. Es el APK de depuración generado localmente; este repositorio no distribuye el binario.

## Requisitos

- Windows con PowerShell y la instalación local de Open WebUI.
- WSL con Ubuntu para Qwen3-TTS, si se usa voz.
- Android con la app SakuraWebApp instalada.
- El teléfono y el servidor deben poder comunicarse por la red. Una dirección privada de la red local normalmente no es accesible desde datos móviles, a menos que haya una VPN o una ruta configurada.
- Para usar micrófono/cámara desde una página web, el origen debe ser un contexto seguro. HTTPS necesita un certificado confiable para el nombre o la IP usados en el teléfono y en la computadora.

## Iniciar Qwen3-TTS (opcional)

Abre una terminal de Windows y ejecuta el servidor de WSL. Ajusta la ruta de Linux si el entorno virtual está en otro lugar:

```powershell
wsl.exe -d Ubuntu -- /home/<usuario-linux>/qwen3-tts-fast-env/bin/python /mnt/d/Sakura/Voz/qwen3-tts/server_fast.py
```

Deja esa terminal abierta mientras uses la síntesis de voz. El servicio usado en esta configuración escucha en `127.0.0.1:5052`. En los registros, una solicitud correcta aparece como `POST /v1/audio/speech ... 200 OK`.

En los ajustes de voz de Open WebUI, usa el proveedor compatible con OpenAI y la dirección local configurada para el servicio, normalmente:

- Base URL: `http://127.0.0.1:5052/v1`
- Modelo: `qwen3-tts-clone-0.6b`

La disponibilidad de `127.0.0.1` entre Windows y WSL depende de la configuración de WSL. Si Open WebUI no alcanza el servicio, comprueba primero desde Windows que el puerto `5052` responde y configura la dirección que Windows puede alcanzar.

## Iniciar Open WebUI con HTTPS

Desde PowerShell, configura las variables para tu instalación. Sustituye las rutas y el host de ejemplo por los valores de tu equipo. Mantén la clave secreta en un archivo local fuera de GitHub.

```powershell
Set-Location 'D:\<ruta>\Open WebUI'

$env:DATA_DIR = 'D:\<ruta>\Open WebUI\data'
$env:WEBUI_SECRET_KEY = (Get-Content -LiteralPath '<ruta-local-al-archivo-de-clave>' -Raw).Trim()

$env:PYTHONUTF8 = '1'
$env:PYTHONIOENCODING = 'utf-8:backslashreplace'

$env:WHISPER_VAD_FILTER = 'true'
$env:WHISPER_LANGUAGE = 'es'
$env:WHISPER_MULTILINGUAL = 'true'
$env:WHISPER_COMPUTE_TYPE = 'int8'

$env:FROM_INIT_PY = 'true'
$env:WEBUI_URL = 'https://<host-o-ip-del-servidor>:8081'
$env:CORS_ALLOW_ORIGIN = $env:WEBUI_URL

& '.\Scripts\python.exe' '-X' 'utf8' '.\run_open_webui_https.py'
```

La terminal permanece ocupada mientras el servidor corre. Para detenerlo, pulsa `Ctrl+C` en esa terminal.

### Conservación de perfil y chats

- Usa siempre el mismo `DATA_DIR` al iniciar el servidor.
- Conserva el contenido de ese directorio y la misma `WEBUI_SECRET_KEY`.
- No borres `data` ni cambies a un directorio vacío: Open WebUI puede mostrar el asistente de creación de administrador y parecer que perdió el perfil.
- No publiques la clave secreta, la base de datos ni copias del directorio de datos.

## Certificado HTTPS

Si el certificado no es confiable o no corresponde al host usado, Chrome y Android pueden mostrar una advertencia, y las funciones de micrófono/cámara pueden ser bloqueadas. Instala y confía en la autoridad certificadora en cada dispositivo, y asegúrate de que el certificado incluya en sus nombres alternativos (SAN) el host o la IP que se abre. No cambies la dirección entre la URL de Open WebUI y la que usa la app.

Un error `ERR_SSL_PROTOCOL_ERROR` suele indicar que se está usando HTTPS contra un servicio que espera HTTP, o viceversa. Confirma el esquema y el certificado del servidor antes de reinstalar la app.

## Compilar el APK

Desde la carpeta del proyecto Android:

```powershell
.\gradlew.bat assembleDebug
```

El APK de depuración queda bajo:

```
app/build/outputs/apk/debug/
```

Instala la nueva versión encima de la anterior para conservar los datos locales del WebView. Desinstalar o borrar los datos de la app puede cerrar la sesión guardada. El APK de depuración no es una versión firmada para publicación en Google Play.

## Funciones de la app documentadas

- Abre Open WebUI dentro de la app, sin iniciar Chrome.
- WebView con JavaScript, DOM storage y cookies para conservar la sesión web.
- Selección de archivos desde la página web.
- Integración de permisos multimedia de Android/WebView.
- Reproducción de audio web sin requerir una segunda interacción, necesaria para escuchar las respuestas de voz.
- Pantalla de conexión/carga con opción de reintento en las versiones trabajadas.

El soporte final de voz depende también de los permisos del teléfono, de HTTPS confiable y de que Open WebUI y el servicio TTS estén funcionando.

## Solución de problemas

### La página queda en blanco o no conecta

1. Comprueba que Open WebUI sigue ejecutándose y que abre desde una computadora en la misma red.
2. Comprueba que el teléfono puede alcanzar el host configurado; una IP privada no funciona desde datos móviles sin VPN/ruta.
3. Verifica que la app y el servidor usan la misma dirección y el mismo esquema (`http` o `https`).
4. Si hay HTTPS, valida que el certificado sea confiable y coincida con el host.
5. En la app, pulsa **Reintentar** después de restablecer la red o iniciar el servidor.

### Aparece el asistente para crear la cuenta administrativa

El servidor probablemente se inició con otro `DATA_DIR` vacío. Deténlo y vuelve a iniciarlo con el directorio que contiene los datos originales y la misma clave secreta. No crees una cuenta nueva ni elimines datos hasta confirmar la ruta.

### Sakura no escucha o el navegador indica permiso denegado

- Concede permiso de micrófono a SakuraWebApp en Android.
- Concede el permiso solicitado por la página dentro de WebView.
- Abre la página mediante HTTPS con certificado confiable; HTTP o un certificado inválido puede impedir el acceso a dispositivos.
- Revisa los ajustes de micrófono y transcripción de Open WebUI.

### La respuesta se muestra pero no se escucha

- Sube el volumen multimedia del teléfono.
- Comprueba que Qwen3-TTS sigue activo y que el registro muestra respuestas HTTP `200`.
- Confirma que Open WebUI apunta al endpoint y modelo correctos.
- El servicio debe seguir abierto mientras se genera audio.

### Error de codificación de Whisper/charmap

Asegúrate de que las variables `PYTHONUTF8` y `PYTHONIOENCODING` estén configuradas antes de iniciar Open WebUI, y ejecuta Python con `-X utf8`.

## Pendiente

La activación por palabra de inicio, como “Oye, Sakura”, no está implementada.

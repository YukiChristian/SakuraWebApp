# SakuraWebApp

SakuraWebApp convierte tu instalación de Open WebUI en una experiencia de app Android: mantiene un WebView dentro de la aplicación, conserva su sesión web y vuelve a mostrar la página sin depender de una pestaña de Chrome que se descarte al salir. Open WebUI y sus datos siguen ejecutándose y almacenándose en tu propio equipo.

## Objetivo

Crear una app web funcional alrededor de Open WebUI que Android pueda mantener como una aplicación instalada. La app ofrece un acceso directo, conserva las cookies y el almacenamiento de la sesión, maneja permisos de micrófono y selector de archivos, y muestra estados de carga/conexión. Open WebUI sigue siendo la interfaz y el servidor; SakuraWebApp es el contenedor Android.

## Estado documentado

- App: SakuraWebApp **0.1.7** (versionCode 18).
- Open WebUI: **0.12.3**, versión verificada en la instalación local.
- Modelo visible en Open WebUI: **Sakura**. El identificador/proveedor del modelo LLM que sirve esa etiqueta no está incluido en la documentación disponible.
- Voz: servicio Qwen3-TTS Fast compatible con API de OpenAI, modelo **qwen3-tts-clone-0.6b**, puerto local **5052**.
- Android: minSdk 23, targetSdk 36, compileSdk 37.
- La activación por palabra “Oye, Sakura” queda como idea futura; no está implementada.

## Código y guía

Este repositorio contiene el proyecto Android y guías de configuración. No contiene el APK generado, perfiles o base de datos de Open WebUI, claves secretas, certificados TLS privados ni la CA local de la instalación. La URL de servidor del código es un ejemplo: especifica la dirección de tu propia instalación al compilar.

- [Configuración de Open WebUI y Qwen3-TTS](docs/OPENWEBUI.md)
- [HTTPS y certificados](docs/HTTPS.md)
- [Implementación Android, ícono, permisos y archivos](docs/ANDROID.md)

## Compilar

En Windows, con Android Studio y el JDK que requiere el proyecto, ejecuta desde la carpeta raíz:

```powershell
.\gradlew.bat assembleDebug -PSAKURA_WEBUI_URL="https://<IP-o-host-del-servidor>:8081/"
```

El host es el equipo que ejecuta Open WebUI, no el teléfono. Usa una dirección que el teléfono pueda alcanzar por Wi-Fi o VPN. Una IP privada de la red local no es accesible desde datos móviles sin una ruta/VPN configurada.

El APK de depuración queda en `app/build/outputs/apk/debug/`. Instala una actualización encima de la app existente para conservar los datos locales; desinstalarla puede borrar la sesión del WebView. La compilación debug no es un APK firmado para Google Play.

## Privacidad del repositorio

No subas `local.properties`, `data/`, bases de datos, `.webui_secret_key`, archivos `.key`, directorios TLS, APKs o certificados privados. La CA pública de una instalación local se agrega solo al build local cuando haga falta; ver [HTTPS.md](docs/HTTPS.md).

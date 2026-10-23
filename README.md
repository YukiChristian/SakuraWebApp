# 🌸 SakuraWebApp 📱

SakuraWebApp empaqueta la interfaz de Open WebUI en una aplicación Android instalada. El objetivo es ofrecer una experiencia de app completa y conservar la sesión web y el estado de navegación mientras Android mantiene la aplicación abierta, en lugar de depender de una pestaña de Chrome que puede descartarse al cambiar de aplicación y recargarse al volver.

## 🎯 Objetivo general

Crear una aplicación Android basada en Open WebUI que se sienta y funcione como una app: abrir directamente el chat, conservar cookies y almacenamiento web, manejar micrófono y selección de archivos, y mostrar estados claros de carga y conexión. Open WebUI sigue proporcionando la interfaz y el servidor; SakuraWebApp la presenta dentro de un WebView integrado en Android.

La app intenta mantener la página cargada en memoria mientras su proceso sigue activo, para volver al chat con fluidez. Android puede cerrar procesos en segundo plano cuando necesita memoria; por eso no se promete que la página permanezca siempre viva. En ese caso, la app puede volver a cargar Open WebUI, mientras que las cookies y el almacenamiento web ayudan a conservar el inicio de sesión. Los chats, el perfil y la configuración pertenecen al servidor y permanecen en su carpeta de datos.

## ✨ Qué incluye

- Aplicación Android con WebView, sesión persistente y navegación hacia atrás.
- Pantallas de carga y de error de conexión con opción para reintentar.
- Selector de archivos con selección múltiple.
- Permisos de micrófono y audio gestionados por Android y WebView.
- Icono de Sakura. La variante negra se muestra abajo; el arte original está en [assets/sakura-original.png](assets/sakura-original.png).
- **TTS opcional:** Qwen3-TTS Fast es el proveedor usado en esta instalación de ejemplo. No es requisito; puedes configurar otro proveedor de texto a voz compatible con tu instancia de Open WebUI, o no usar TTS.

<div align="center">
  <img src="assets/sakura-icon-black.png" alt="Flor de Sakura sobre fondo negro" width="160">
</div>

## 🖼️ Capturas

Las capturas muestran el inicio de sesión, la pantalla principal, el modo de voz, los ajustes de Open WebUI y el icono instalado en el lanzador de Android.

<details>
<summary>Ver capturas de SakuraWebApp</summary>

| Iniciar sesión | Pantalla principal |
|:--:|:--:|
| <img src="docs/images/login.png" alt="Inicio de sesión de Open WebUI" width="240"> | <img src="docs/images/home.png" alt="Pantalla principal de Sakura" width="240"> |

| Modo de voz | Ajustes de interfaz |
|:--:|:--:|
| <img src="docs/images/voice-mode.png" alt="Modo de voz de Sakura" width="240"> | <img src="docs/images/settings.png" alt="Ajustes de Open WebUI en Android" width="240"> |

| Icono en el lanzador |
|:--:|
| <img src="docs/images/launcher-preview.png" alt="Icono SakuraWebApp en el lanzador de Android" width="240"> |

</details>

## 📦 Versiones documentadas

- App: SakuraWebApp **0.1.7** (versionCode 18).
- Open WebUI: **0.12.3**, versión verificada en la instalación local.
- Modelo visible en Open WebUI: **Sakura**. El identificador/proveedor del modelo LLM que sirve esa etiqueta no está incluido en la documentación disponible.
- Voz de ejemplo: Qwen3-TTS Fast compatible con API de OpenAI, modelo **qwen3-tts-clone-0.6b**, puerto local **5052**. Puedes reemplazarlo por otro TTS.
- Android: minSdk 23, targetSdk 36, compileSdk 37.
- La activación por palabra “Oye, Sakura” queda como idea futura; no está implementada.

## 🚀 Pasos para usar o adaptar el proyecto

1. Prepara una instancia de Open WebUI y confirma que el teléfono pueda abrirla en la red.
2. Configura HTTPS y un certificado confiable para el host que usarás. La guía explica cómo trabajar con una CA local.
3. Clona este repositorio y compila la app con la URL de tu servidor.
4. Instala el APK en Android y concede el permiso de micrófono si usarás voz.
5. Configura el TTS que prefieras dentro de Open WebUI si quieres respuestas habladas. No necesitas instalar el servidor Qwen de este ejemplo.

Las guías detallan la instalación del servidor, las variables, los certificados, la URL que hay que cambiar, los permisos y los problemas de carga de archivos:

- [Configuración de Open WebUI y TTS](docs/OPENWEBUI.md)
- [HTTPS y certificados](docs/HTTPS.md)
- [Implementación Android, ícono, permisos, archivos y capturas](docs/ANDROID.md)

## 🛠️ Compilar

Necesitas Android Studio, Android SDK y el JDK requerido por el proyecto. En Windows, desde la raíz del proyecto:

```powershell
.\gradlew.bat assembleDebug -PSAKURA_WEBUI_URL="https://<IP-o-host-del-servidor>:8081/"
```

El host es el equipo que ejecuta Open WebUI, no el teléfono. Usa una dirección que el teléfono pueda alcanzar por Wi-Fi o VPN. Una IP privada de la red local no es accesible por datos móviles sin una ruta o VPN configurada.

El APK de depuración queda en `app/build/outputs/apk/debug/`. Instala una actualización encima de la app existente para conservar los datos locales; desinstalarla puede borrar la sesión del WebView. La compilación debug no es un APK firmado para Google Play.

## 🔐 Privacidad

Este repositorio contiene el código Android, capturas de la app y documentación. No contiene el APK generado, perfiles o bases de datos de Open WebUI, claves secretas, certificados TLS privados ni la CA local de una instalación. No subas `local.properties`, `data/`, bases de datos, `.webui_secret_key`, archivos `.key`, directorios TLS, APKs o certificados privados. La CA pública de una instalación local se agrega solo al build local cuando haga falta; ver [HTTPS.md](docs/HTTPS.md).

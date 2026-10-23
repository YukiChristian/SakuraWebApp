# HTTPS y certificados de Sakura

## Por qué se cambió de HTTP a HTTPS

Open WebUI se abría primero por HTTP en una dirección privada. Los navegadores modernos limitan APIs de dispositivos como `navigator.mediaDevices.getUserMedia` (micrófono) a contextos seguros; la excepción de loopback no ayuda a un teléfono que accede a una IP de LAN. Por eso Android/WebView podía mostrar “Permiso denegado accediendo a los dispositivos” incluso cuando el permiso de micrófono de Android estaba activo.

Se configuró Open WebUI con HTTPS en el puerto 8081. HTTPS también verifica que el servidor al que se conecta el teléfono sea el servidor esperado. Si el certificado está firmado por una CA local, cada cliente debe confiar en esa CA y el certificado de servidor debe tener en SAN exactamente la IP/DNS que se visita. El error `ERR_SSL_PROTOCOL_ERROR` puede aparecer cuando cliente y servidor no coinciden en usar HTTP o HTTPS.

## Certificados locales observados

El lanzador local de Uvicorn usaba `tls/server.crt` y `tls/server.key`. La app Android tenía una copia del certificado de la autoridad certificadora local bajo `app/src/main/res/raw/sakura_ca.crt` y lo añadía a `network_security_config.xml` como ancla de confianza. Es una CA local para este servidor, no una CA pública.

No se encontró un script de generación en el proyecto. Por eso no es posible asegurar qué herramienta/comandos exactos produjeron los archivos actuales. La receta OpenSSL de abajo es una forma reproducible de generar el mismo tipo de estructura. Genera una CA nueva; no recupera ni necesita la clave privada anterior.

## Receta reproducible con OpenSSL

Ejecuta esto en un directorio privado. Sustituye `<HOST>` por el DNS/IP que abrirán teléfono y PC. Protege la clave de la CA y no la copies a GitHub:

```sh
# Clave privada de la CA, cifrada con una frase de paso.
openssl genpkey -algorithm RSA -aes-256-cbc -pkeyopt rsa_keygen_bits:4096 -out sakura-root-ca.key

# Certificado público de la CA.
openssl req -x509 -new -sha256 -key sakura-root-ca.key -days 3650 \
  -out sakura-root-ca.crt -subj "/CN=Sakura Local Root CA" \
  -addext "basicConstraints=critical,CA:TRUE" \
  -addext "keyUsage=critical,keyCertSign,cRLSign"

# Clave privada y solicitud del certificado TLS del servidor.
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out server.key
openssl req -new -key server.key -out server.csr -subj "/CN=<HOST>"
```

Crea `server.ext` con SAN y extensiones de servidor. Conserva solo las entradas DNS/IP que uses realmente:

```ini
basicConstraints=critical,CA:FALSE
keyUsage=critical,digitalSignature,keyEncipherment
extendedKeyUsage=serverAuth
subjectAltName=IP:<IP-LAN>,DNS:<NOMBRE-DNS>
```

Firma el certificado del servidor con la CA:

```sh
openssl x509 -req -in server.csr -CA sakura-root-ca.crt -CAkey sakura-root-ca.key \
  -CAcreateserial -out server.crt -days 825 -sha256 -extfile server.ext
```

Instala `server.crt` y `server.key` como archivos locales en `tls/` junto al lanzador de Uvicorn. Copia solo el certificado público `sakura-root-ca.crt` a `app/src/main/res/raw/sakura_ca.crt`; nunca copies `sakura-root-ca.key` ni `server.key` al repositorio. La app solo debe confiar en esa CA para el dominio de tu servidor. Referencia: [manuales oficiales de OpenSSL](https://docs.openssl.org/3.0/man1/openssl-req/).

## Configurar Android para la CA

La plantilla del repositorio confía solo en CA del sistema y no trae tu CA privada. Si tu servidor usa la CA local:

1. Crea `app/src/main/res/raw/` y coloca `sakura_ca.crt` (certificado público PEM/DER).
2. En `network_security_config.xml`, crea una regla `domain-config` para el mismo host de `SAKURA_WEBUI_URL`.
3. En `trust-anchors`, conserva `<certificates src="system" />` y agrega `<certificates src="@raw/sakura_ca" />`.
4. Revisa SAN, URL, `WEBUI_URL` y `CORS_ALLOW_ORIGIN`; todos deben coincidir.
5. Vuelve a compilar e instalar.

La app cancela los errores SSL de WebView. No añadas una excepción que acepte cualquier certificado. En el PC, importa la CA pública en el almacén de confianza del sistema/navegador si también quieres navegar directamente desde Chrome. Android permite configurar CA personalizadas por app en [Network Security Configuration](https://developer.android.com/privacy-and-security/security-config).

## Seguridad y datos que no se publican

La CA pública no permite firmar certificados sin su clave privada, pero la clave de la CA sí permitiría suplantar al servidor. Por eso se excluyen claves, archivos `tls/`, certificados locales y datos de Open WebUI del repositorio público. El certificado público local también se omite para no fijar el repo a una red privada.

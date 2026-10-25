PokeDex - Despliegue y Configuración de Seguridad en Azure App Service
En este documento se detalla el procedimiento técnico paso a paso para publicar la aplicación **PokeDex** (construida en Angular) en **Azure App Service**, incluyendo la integración del archivo de políticas de seguridad y reescritura de rutas `web.config` incluido dentro de los assets del proyecto para obtener la calificación (**A**) en el escaneo de [Security Headers](https://securityheaders.com/ "null").

  

## 1. Preparación de Archivos y Configuración de Seguridad

  

### A. Archivo de Seguridad `web.config`

  

El archivo de políticas de seguridad y enrutamiento para Single Page Applications (SPA) se encuentra alojado dentro de la carpeta `src/web.config`. Contiene las siguientes directivas HTTP de seguridad y reglas de reescritura para IIS:

  

```

<?xml version="1.0" encoding="utf-8"?>

<configuration>

<system.webServer>

<rewrite>

<rules>

<rule name="Angular Routes" stopProcessing="true">

<match url=".*" />

<conditions logicalGrouping="MatchAll">

<add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />

<add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />

</conditions>

<action type="Rewrite" url="/" />

</rule>

</rules>

</rewrite>

<httpProtocol>

<customHeaders>

<!-- Security Headers para A+ -->

<add name="Strict-Transport-Security" value="max-age=31536000; includeSubDomains; preload" />

<add name="X-Frame-Options" value="DENY" />

<add name="X-Content-Type-Options" value="nosniff" />

<add name="Referrer-Policy" value="strict-origin-when-cross-origin" />

<add name="Permissions-Policy" value="camera=(), microphone=(), geolocation=(), payment=()" />

<!-- CSP completa que permite llamadas a la PokeAPI y recursos externos -->

<add name="Content-Security-Policy" value="default-src 'self' 'unsafe-inline' 'unsafe-eval' https: data: blob:; connect-src 'self' https://pokeapi.co https://*.pokeapi.co; img-src 'self' data: blob: https: https://raw.githubusercontent.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' data: https://fonts.gstatic.com;" />

</customHeaders>

</httpProtocol>

</system.webServer>

</configuration>

```

  

### B. Inclusión en `angular.json`

  

Para asegurar que Angular incluya el archivo de configuración en la carpeta compilada de salida (`dist/pokedex-angular/`), este se encuentra declarado en la matriz de assets en `angular.json`:

  

```

"assets": [

"src/favicon.ico",

"src/og-image.jpg",

"src/assets",

"src/404.html",

"src/web.config"

]

```

  

## 2. Compilación del Proyecto y Preparación para Azure

  

1. **Instalación de Dependencias:**

```

npm install

```

2. **Compilación para Producción:**

```

npm run build

```

_El resultado de la compilación se generará dentro del directorio `dist/pokedex-angular/` e incluirá el archivo `web.config`._

  

## 3. Creación y Configuración de la Web App en Azure Portal

  

1. **Creación de Azure App Service:**

- Ingresa a [Azure Portal](https://portal.azure.com/ "null").

- Busca **Servicios de aplicaciones** y haz clic en **+ Crear** -> **Aplicación web**.

- **Suscripción:** `Azure for Students`.

- **Grupo de Recursos:** `rg-pokedex-prod`.

- **Nombre:** `PokeDex Ovi`

- **Publicar:** Código

- **Pila de entorno de ejecución:** Node 22 LTS.

- **Sistema operativo:** Windows (IIS).

- **Region:** Brazil South.

- **Plan de hospedaje:** F1 (Free / Gratuito).

- Haz clic en **Revisar y crear** y posteriormente en **Crear**.

  

## 4. Publicación / Despliegue de la Aplicación

  

Puedes desplegar moviendo la carpeta compilada comprimida`dist/pokedex-angular/` mediante **Azure CLI**:

  

```

az webapp deploy --resource-group rg-pokedex --name pokedexovi --src-path dist/pokedex-angular.zip --type zip

  

```

  

## 5. Verificación de Seguridad y Escaneo (Nota A)

  

1. Obtén la URL pública asignada por Azure: `https://pokedexovi-g0bverg3b8gvbwf3.brazilsouth-01.azurewebsites.net/`.

2. Ingresa a [Security Headers](https://securityheaders.com/ "null").

3. Pega la URL pública y presiona **Scan**.

4. **Resultado:** Se verifica la obtención de la calificación **A**, confirmando que las cabeceras inyectadas mediante el archivo `web.config` han sido aplicadas correctamente.

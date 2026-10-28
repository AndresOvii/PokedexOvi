# PokeDex - Mi Procedimiento de Despliegue y Configuración de Seguridad en Azure App Service

En este documento explico detalladamente el procedimiento técnico que realicé para publicar mi aplicación **PokeDex** (desarrollada en Angular) en **Azure App Service**, asegurando la inclusión del archivo de seguridad y enrutamiento `web.config` en mis assets para obtener la calificación **A** en el análisis de [Security Headers](https://securityheaders.com/).

## 1. Preparación de Archivos y Configuración de Seguridad

### A. Mi Archivo de Seguridad `web.config`

Ubiqué mi archivo de políticas de seguridad y enrutamiento para Single Page Applications (SPA) dentro de la carpeta `src/web.config`. En este archivo configuré las reglas de reescritura para IIS y las directivas HTTP de seguridad:

```xml
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

### B. Inclusión en mi `angular.json`

Para garantizar que Angular incluyera mi archivo de configuración dentro de la carpeta de salida (`dist/pokedex-angular/`) al momento de compilar, lo declaré dentro del arreglo de `assets` en mi `angular.json`:

JSON

```
"assets": [
  "src/favicon.ico",
  "src/og-image.jpg",
  "src/assets",
  "src/404.html",
  "src/web.config"
]
```

### C. Configuración de Entornos (`environment.ts` y `environment.prod.ts`)

Ajusté la propiedad `imagesPath` en mis entornos para utilizar rutas relativas directas (`assets/images`), asegurando que todas las imágenes y sprites de los Pokémon cargaran sin problemas desde mi servidor en Azure:

TypeScript
```
export const environment = {
  production: true,
  pokeApi: 'https://pokeapi.co/api/v2',
  pokeApiGraphQL: 'https://beta.pokeapi.co/graphql/v1beta',
  homeAngular: 'https://angular.io/',
  homePokeApi: 'https://pokeapi.co/',
  keilerLinkedin: 'https://www.linkedin.com/in/keilermora/',
  pokedexGithub: 'https://github.com/keilermora/pokedex-angular',
  imagesPath: 'assets/images',
  language: 'en',
  languageId: 9,
}; 
```

## 2. Compilación del Proyecto y Preparación para Azure

1.  **Instalé las dependencias de mi proyecto:**
    
    Bash
    
    ```
    npm install
    
    ```
    
2.  **Generé la compilación para producción:**
    
    Bash
    
    ```
    npm run build
    
    ```
    
    _Con esto se generó la carpeta `dist/pokedex-angular/` conteniendo mi código optimizado, el archivo `web.config` y los assets de imágenes._
    

## 3. Creación y Configuración de mi Web App en Azure Portal

1.  **Creación del recurso en Azure App Service:**
    
    -   Ingresé a mi consola en [Azure Portal](https://portal.azure.com/).
        
    -   Busqué la opción **Servicios de aplicaciones** y seleccioné **+ Crear** -> **Aplicación web**.
        
    -   **Suscripción:**  `Azure for Students`.
        
    -   **Grupo de Recursos:**  `rg-pokedex-prod`.
        
    -   **Nombre de la App:**  `PokeDex Ovi`
        
    -   **Publicar:** Código.
        
    -   **Entorno de ejecución:** Node 22 LTS.
        
    -   **Sistema operativo:** Windows (IIS).
        
    -   **Región:** Brazil South.
        
    -   **Plan de precios:** F1 (Free / Gratuito).
        
    -   Presioné **Revisar y crear** y posteriormente **Crear**.
        

## 4. Publicación / Despliegue de la Aplicación

Comprimí en un archivo `.zip` los archivos generados dentro de mi carpeta `dist/pokedex-angular/` (incluyendo mi `index.html`,  `web.config` y la carpeta `assets`) y ejecuté el despliegue a través de el apartado herramientas de desarrollo, herramientas avanzadas, presione ir, luego me dirigí a debug console, cmd, luego a tools, zip push deploy y monte el zip comprimido en el espacio libre.

## 5. Verificación de Seguridad y Escaneo (Nota A)

1.  Obtuve la URL pública otorgada por Azure:  `https://pokedexovi-g0bverg3b8gvbwf3.brazilsouth-01.azurewebsites.net/`.
    
2.  Ingresé a la plataforma [Security Headers](https://securityheaders.com/).
    
3.  Pegué mi URL pública y ejecuté el escaneo (**Scan**).
    
4.  **Resultado obtenido:** Verifiqué con éxito la nota **A**, confirmando que mis cabeceras HTTP de seguridad declaradas en mi `web.config` fueron procesadas y aplicadas de forma correcta por IIS.
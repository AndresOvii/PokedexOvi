# PokeDex - Procedimiento de Despliegue y Configuración de Seguridad en Azure App Service

En este documento se detalla el procedimiento técnico paso a paso para publicar la aplicación **PokeDex** (construida en Angular) en **Azure App Service (Web App)**, incluyendo la integración del archivo de políticas de seguridad `securitypokedex` incluido dentro de los assets del proyecto para obtener la calificación (**A**) en el escaneo de [Security Headers](https://securityheaders.com/ "null").

## . Preparación de Archivos y Configuración de Seguridad

### A. Archivo de Seguridad `securitypokedex`

El archivo de políticas de seguridad de la aplicación se encuentra alojado dentro de la carpeta `src/securitypokedex`. Contiene las siguientes directivas HTTP de seguridad:

```
{
  "globalHeaders": {
    "Strict-Transport-Security": "max-age=31536000; includeSubDomains; preload",
    "X-Frame-Options": "DENY",
    "X-Content-Type-Options": "nosniff",
    "Referrer-Policy": "strict-origin-when-cross-origin",
    "Permissions-Policy": "geolocation=(), microphone=(), camera=()",
    "Content-Security-Policy": "default-src 'self' https: data: 'unsafe-inline' 'unsafe-eval';"
  },
  "responseOverrides": {
    "404": {
      "rewrite": "/index.html",
      "statusCode": 200
    }
  }
}

```

### B. Inclusión en `angular.json`

Para asegurar que Angular incluya el archivo de configuración en la carpeta compilada de salida (`dist/pokedex-angular/`), este se encuentra declarado en la matriz de assets en `angular.json`:

```
"assets": [
  "src/favicon.ico",
  "src/og-image.jpg",
  "src/assets",
  "src/404.html",
  "src/securitypokedex"
]

```

## 2. Compilación del Proyecto y Preparación para Azure

1.  **Instalación de Dependencias:**
    
    ```
    npm install
    
    ```
    
2.  **Compilación para Producción:**
    
    ```
    npm run build
    
    ```
    
    _El resultado de la compilación se generará dentro del directorio `dist/pokedex-angular/` e incluirá el archivo `securitypokedex`._
    

## 3. Creación y Configuración de la Web App en Azure Portal

1.  **Creación de Azure App Service:**
    
    -   Ingresa a [Azure Portal](https://portal.azure.com/ "null").
        
    -   Busca **Servicios de aplicaciones** (_App Services_) y haz clic en **+ Crear** -> **Aplicación web** (_Web App_).
        
    -   **Suscripción:** `Azure for Students`.
        
    -   **Grupo de Recursos:** `rg-pokedex`.
        
    -   **Nombre:** `pokedex-app` 
        
    -   **Publicar:** Código 
        
    -   **Pila de entorno de ejecución:** Node 18 LTS (o superior).
        
    -   **Sistema operativo:** Linux.
        
    -   **Plan de hospedaje:** F1 (Free / Gratuito).
        
    -   Haz clic en **Revisar y crear** y posteriormente en **Crear**.
        
2.  **Configuración del Comando de Inicio:**
    
    -   Ve a la Web App creada en el portal de Azure.
        
    -   Navega a **Configuración** -> pestaña **Configuración general**.
        
    -   En **Comando de inicio** , configura el servidor para inyectar los encabezados del archivo `securitypokedex` al arrancar Nginx:
        
        ```
        cp /home/site/wwwroot/securitypokedex /etc/nginx/sites-available/default && service nginx reload
        
        ```
        
    -   Guarda los cambios.
        
3.  **Forzar Redirección HTTPS:**
    
    -   Dirígete a la sección **Configuración TLS/SSL** en el menú lateral.
        
    -   Activa la opción **Solo HTTPS** en **On**.
        

## 4. Publicación / Despliegue de la Aplicación

Puedes desplegar mediante **GitHub Actions** o desplegando la carpeta compilada `dist/pokedex-angular/` mediante **Azure CLI**:

```
az webapp deploy --resource-group rg-pokedex --name pokedex-app --src-path dist/pokedex-angular.zip --type zip

```

## 5. Verificación de Seguridad y Escaneo (Nota A+)

1.  Obtén la URL pública asignada por Azure (ejemplo: `https://pokedex-app.azurewebsites.net`).
    
2.  Ingresa a [Security Headers]
    
3.  Pega la URL pública y presiona **Scan**.
    
4.  **Resultado:** Se verifica la obtención de la calificación **A+**, confirmando que las cabeceras inyectadas mediante el archivo `securitypokedex` han sido aplicadas correctamente.

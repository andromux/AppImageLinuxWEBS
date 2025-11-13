# Guía Completa: Convierte Sitios Web en Aplicaciones de Escritorio Linux

## 🎯 ¿Qué es esto y para qué sirve?

Esta guía te enseña a **convertir cualquier sitio web en una aplicación de escritorio nativa para Linux**, empaquetada como **AppImage** (un formato portable que funciona en cualquier distribución sin instalación).

### ¿Qué puedes lograr con esto?

- **Convertir servicios web en apps independientes**: Gmail, WhatsApp Web, Notion, Trello, Discord, etc.
- **Aislar aplicaciones web**: Cada app web tendrá su propia ventana, icono y puede ejecutarse independientemente del navegador
- **Crear aplicaciones portables**: Los AppImages funcionan en cualquier distro Linux sin instalación
- **Personalizar comportamiento**: Control total sobre permisos, argumentos de ejecución y configuración

### Ejemplos de uso práctico:
- Cliente de escritorio para tu música favorita (Spotify Web, YouTube Music)
- Apps de productividad (Notion, Google Docs, Office 365)
- Juegos web (Lichess, Chess.com, Geoguessr)
- Herramientas de trabajo (Slack, Discord, Microsoft Teams)
- Servicios de streaming (Netflix, Prime Video, etc.)

---

## 📋 Requisitos Previos

Antes de comenzar, necesitas tener instalado:

```bash
# Node.js y npm (gestor de paquetes de JavaScript)
sudo apt install nodejs npm  # Ubuntu/Debian
sudo dnf install nodejs npm  # Fedora
sudo pacman -S nodejs npm    # Arch Linux
```

---

## 🚀 Paso 1: Instalar las Herramientas Necesarias

Instala globalmente las herramientas que usaremos:

```bash
npm install -g electron-builder
npm install -g nativefier
```

**¿Qué hace cada herramienta?**
- **Nativefier**: Convierte sitios web en aplicaciones Electron
- **Electron Builder**: Empaqueta aplicaciones Electron en formatos distribuibles (AppImage, .deb, etc.)

---

## 🔨 Paso 2: Generar la Aplicación Base con Nativefier

Usa este comando para convertir tu sitio web favorito:

```bash
nativefier "https://TU-SITIO-WEB.com/" \
  --name "NombreDeApp" \
  --internal-urls ".*" \
  --single-instance
```

**Parámetros explicados:**
- `--name`: Nombre de la aplicación
- `--internal-urls ".*"`: Permite navegación dentro del mismo dominio
- `--single-instance`: Evita que se abran múltiples instancias

### Ejemplo real (Lichess):

```bash
nativefier "https://lichess.org/" \
  --name "Lichess" \
  --internal-urls ".*" \
  --single-instance
```

Esto crea una carpeta llamada `Lichess-linux-x64/`

---

## 🔒 Paso 3: Configurar Permisos del Sandbox

Electron usa un sandbox de seguridad que requiere permisos especiales:

```bash
cd Lichess-linux-x64/
sudo chown root chrome-sandbox
sudo chmod 4755 chrome-sandbox
cd ../
```

**¿Por qué esto?** El sandbox de Chrome necesita ejecutarse con privilegios específicos para funcionar correctamente.

---

## 📦 Paso 4: Crear el Archivo de Configuración

Crea un archivo llamado `package.json` en la carpeta actual con este contenido:

> ⚠️ **NOTA CRÍTICA:** El campo `"productName"` en el `package.json` **DEBE coincidir EXACTAMENTE** con el nombre de la carpeta generada por Nativefier, respetando mayúsculas y minúsculas. Si Nativefier creó `Lichess-linux-x64/`, entonces `"productName"` debe ser `"Lichess"` (con mayúscula). Si no coinciden, electron-builder no encontrará el binario y dará error al compilar.

```json
{  
  "name": "lichess",  
  "version": "1.0.0",  
  "description": "Cliente de escritorio Lichess creado con Nativefier",  
  "main": "index.js",  
  "scripts": {  
    "test": "echo \"Error: no test specified\" && exit 1"  
  },  
  "keywords": [],  
  "author": "Tu Nombre",  
  "license": "ISC",  
  "build": {  
    "productName": "Lichess",  
    "appId": "org.lichess.desktop",  
    "electronVersion": "26.0.12",  
    "directories": {  
      "output": "dist"  
    },  
    "linux": {  
      "target": [  
        "AppImage",  
        "deb"
      ],  
      "category": "Game",  
      "executableArgs": [  
        "--no-sandbox"  
      ]  
    }  
  }  
}
```

**Personaliza estos campos:**
- `"name"`: nombre interno (minúsculas, sin espacios)
- `"productName"`: **⚠️ CRÍTICO - Debe coincidir EXACTAMENTE con el nombre de la carpeta generada por Nativefier** (ej: si creaste `Lichess-linux-x64/`, usa `"Lichess"` con mayúscula)
- `"appId"`: identificador único (formato: org.nombre.app)
- `"category"`: categoría de la app (Game, Office, Network, AudioVideo, etc.)
- `"author"`: tu nombre

---

## 🎁 Paso 5: Construir el AppImage

Ejecuta el comando de empaquetado:

```bash
electron-builder -l AppImage --prepackaged "Lichess-linux-x64"
```

Esto generará el archivo en `dist/Lichess-1.0.0.AppImage`

---

## ▶️ Paso 6: Ejecutar tu Aplicación

### Opción A: Ejecutar desde terminal

```bash
chmod +x ./dist/Lichess-1.0.0.AppImage
./dist/Lichess-1.0.0.AppImage --no-sandbox
```

### Opción B: Integrar al escritorio con Gear Lever (Recomendado)

1. Instala Gear Lever desde Flathub:
   ```bash
   flatpak install flathub it.mijorus.gearlever
   ```

2. Abre Gear Lever y arrastra tu archivo `.AppImage`

3. Gear Lever lo integrará automáticamente con:
   - Icono en el menú de aplicaciones
   - Parámetro `--no-sandbox` preconfigurado
   - Inicio como cualquier otra aplicación

**Enlace:** https://flathub.org/apps/it.mijorus.gearlever

---

## 🎨 Personalización Avanzada

### Agregar un icono personalizado:

```bash
nativefier "https://ejemplo.com/" \
  --name "MiApp" \
  --icon "ruta/a/icono.png" \
  --internal-urls ".*" \
  --single-instance
```

### Configurar tamaño de ventana:

```bash
nativefier "https://ejemplo.com/" \
  --name "MiApp" \
  --width 1280 \
  --height 720 \
  --internal-urls ".*" \
  --single-instance
```

### Modo app (sin barra de navegación):

```bash
nativefier "https://ejemplo.com/" \
  --name "MiApp" \
  --hide-address-bar \
  --internal-urls ".*" \
  --single-instance
```

---

## 🔧 Solución de Problemas Comunes

### Error: "The SUID sandbox helper binary was found, but is not configured correctly"

**Solución:** Asegúrate de haber ejecutado correctamente los comandos del Paso 3, o ejecuta con `--no-sandbox`

### Error al compilar: "Cannot find executable"

**Causa:** El campo `"productName"` en `package.json` no coincide con el nombre de la carpeta generada por Nativefier.

**Solución:** 
1. Verifica el nombre exacto de la carpeta: `ls` (busca algo como `MiApp-linux-x64/`)
2. Asegúrate que `"productName"` en `package.json` coincida exactamente, respetando mayúsculas y minúsculas
3. Ejemplo: Si la carpeta es `Discord-linux-x64/`, entonces usa `"productName": "Discord"` (con D mayúscula)

### La aplicación no abre

**Verifica:**
1. Que el AppImage tenga permisos de ejecución: `chmod +x archivo.AppImage`
2. Que estés usando el parámetro `--no-sandbox`

### Quiero cambiar la categoría en el menú

Edita la propiedad `"category"` en `package.json`. Opciones válidas:
- `AudioVideo`, `Audio`, `Video`
- `Development`
- `Education`
- `Game`
- `Graphics`
- `Network`
- `Office`
- `Science`
- `Settings`
- `System`
- `Utility`

---

## 📚 Resumen del Flujo Completo

```bash
# 1. Instalar herramientas
npm install -g electron-builder nativefier

# 2. Generar app base
nativefier "https://tu-sitio.com/" --name "MiApp" --internal-urls ".*" --single-instance

# 3. Configurar sandbox
cd MiApp-linux-x64/
sudo chown root chrome-sandbox && sudo chmod 4755 chrome-sandbox
cd ../

# 4. Crear package.json (ver Paso 4)

# 5. Construir AppImage
electron-builder -l AppImage --prepackaged "MiApp-linux-x64"

# 6. Ejecutar
./dist/MiApp-1.0.0.AppImage --no-sandbox
```

---

## 💡 Consejos Finales

- **Experimenta**: Prueba diferentes sitios web y configuraciones
- **Comparte**: Los AppImages son portables, puedes compartirlos con otros usuarios de Linux
- **Actualiza**: Para actualizar la app, simplemente repite el proceso con la nueva versión
- **Organiza**: Guarda tus AppImages en una carpeta específica como `~/Applications/`

---

## 🌟 Ejemplos de Sitios Web Populares para Convertir

- **Productividad**: Notion, Todoist, Trello, Asana
- **Comunicación**: Discord, Telegram Web, WhatsApp Web
- **Entretenimiento**: Netflix, Spotify, YouTube Music, Twitch
- **Desarrollo**: GitHub, GitLab, Figma
- **Juegos**: Lichess, Chess.com, Geoguessr

¡Ahora tienes el poder de convertir toda la web en aplicaciones de escritorio nativas!
# 🤖 Terranova Bot

<div align="center">
  
  ![Version](https://img.shields.io/badge/version-2.0-blue.svg)
  ![Go Version](https://img.shields.io/badge/Go-1.25-00ADD8?logo=go)
  ![Discord](https://img.shields.io/badge/Discord-API-5865F2?logo=discord)
  ![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker)
  ![License](https://img.shields.io/badge/license-Propietario-red.svg)
  
  *Un bot de Discord profesional con sistema de moderación automática y dashboard web integrado.*
  
</div>

---

## 📋 Tabla de Contenidos

- [✨ Características Principales](#-características-principales)
- [🏗️ Arquitectura](#️-arquitectura)
- [🛡️ Sistema de Moderación](#️-sistema-de-moderación)
- [📊 Dashboard Web](#-dashboard-web)
- [🚀 Despliegue](#-despliegue)
- [🔒 Seguridad](#-seguridad)
- [📈 Estadísticas](#-estadísticas)
- [📄 Licencia](#-licencia)

---

## ✨ Características Principales

### 🤖 Bot de Discord
- **Comandos Slash** completamente funcionales
- **Sistema de logs** en tiempo real con WebSocket
- **Dashboard web** interactivo para monitoreo
- **Arquitectura modular** y escalable

### 🛡️ Sistema de AutoMod
- **Filtro de palabras** configurable
- **Anti-Spam** inteligente
- **Anti-Links** con whitelist personalizable
- **Anti-Raid** para detección de uniones masivas
- **Sistema de advertencias** progresivo

### 📊 Dashboard
- **Monitoreo en tiempo real** de RAM y estado del sistema
- **Gestión de configuración** del AutoMod
- **Visualización de logs** con filtros avanzados
- **Interfaz moderna** con diseño responsive

---

## 🏗️ Arquitectura

La arquitectura de Terranova está diseñada siguiendo principios de modularidad, escalabilidad y separación de responsabilidades. A continuación, se detalla cada componente y su función dentro del sistema.

### 1. Capa de Comunicación con Discord

**`Discord Gateway`**  
- **Tecnología**: DiscordGo (librería Go para Discord API)
- **Función**: Establece y mantiene la conexión WebSocket con los servidores de Discord. Recibe eventos en tiempo real (mensajes, miembros, interacciones) y envía respuestas a la API de Discord.

**`WebSocket Interno`**  
- **Tecnología**: WebSocket nativo de Go + Fiber
- **Función**: Canal de comunicación bidireccional entre el bot y el dashboard. Permite transmitir logs y estadísticas en tiempo real sin necesidad de polling HTTP.

### 2. Núcleo de Servicios (Core Services)

**`AutoMod Service`**  
- **Módulos**:
  - **WordFilter**: Filtro de palabras prohibidas con detección de evasiones (leet speak, reemplazos de caracteres). Soporta blacklist global y palabras personalizadas por servidor.
  - **AntiSpam**: Monitorea la frecuencia de mensajes por usuario, aplicando límites configurables (mensajes por ventana de tiempo). Incluye sistema de cooldown para evitar advertencias repetitivas.
  - **AntiLink**: Detecta y bloquea enlaces no permitidos. Incluye whitelist de dominios confiables y bloqueo específico de invites de Discord.
  - **AntiRaid**: Detecta uniones masivas en intervalos de tiempo configurables. Activa slowmode automático y puede expulsar o banear miembros.
  - **Warnings**: Sistema de advertencias progresivas con thresholds configurables (ej: 3 warns = mute, 5 warns = kick, 7 warns = ban).

**`Logging Service`**  
- **Funcionalidades**:
  - Registra eventos en memoria RAM (últimas 1000 entradas) para acceso rápido.
  - Persiste logs en archivos con rotación automática (tamaño máximo 10MB, 5 backups, compresión opcional).
  - Soporta 5 niveles de log: DEBUG, INFO, WARN, ERROR, FATAL.
  - Transmite logs en tiempo real a través de WebSocket al dashboard.

**`Commands Service`**  
- **Tecnología**: Slash Commands de Discord
- **Comandos disponibles**:
  - `ping`: Muestra latencia del bot
  - `vote`: Crea votaciones interactivas con botones
  - `giveaway`: Sistema de sorteos con participación mediante botón
  - `reroll`: Selecciona nuevo ganador de un sorteo finalizado
  - `flipcoin`: Lanza una moneda al aire
  - `lofi`: Reproduce música lofi (conecta a canal de voz)

**`Security Service`**  
- **Funcionalidades**:
  - Rate limiting automático gestionado por DiscordGo
  - Auditoría de comandos ejecutados
  - Monitoreo de estado de componentes (CMD, SVC, LST)

### 3. Capa de Persistencia

**`SQLite`**  
- **Uso**: Almacenamiento de sorteos (giveaways) para mantener estado entre reinicios.
- **Estructura**: Tabla con ID de mensaje, canal, premio, tiempo de finalización y estado.

**`config.json`**  
- **Uso**: Configuración persistente del AutoMod (módulos activados, acciones, thresholds, whitelist, canales ignorados).
- **Formato**: JSON con estructura jerárquica por módulo.

**`blacklist.txt`**  
- **Uso**: Lista de palabras prohibidas globales.
- **Formato**: Una palabra por línea.

### 4. Capa de Presentación (Dashboard Web)

**`Dashboard Web`**  
- **Tecnologías**: HTML5, CSS3, JavaScript, Chart.js
- **Páginas**:
  - **Dashboard Principal**: Visión general con gráfico de RAM, estado del sistema y componentes activos.
  - **AutoMod**: Panel de control completo para gestionar todos los módulos, blacklist, whitelist y thresholds.
  - **Logs**: Visualización en tiempo real con filtros por nivel y módulo.

**`WebSocket Server`**  
- **Tecnología**: Fiber + WebSocket
- **Función**: Servidor WebSocket que distribuye logs y estadísticas a los clientes conectados. Gestiona conexiones concurrentes y buffer de logs para reconexiones.

**`API REST`**  
- **Endpoints principales**:
  - `/api/automod/config`: GET/POST para configuración completa
  - `/api/automod/words`: GET/POST/DELETE para gestión de blacklist
  - `/api/automod/custom-words`: GET/POST/DELETE para palabras personalizadas
  - `/api/automod/whitelist`: GET/POST/DELETE para dominios permitidos
  - `/api/logs`: GET para obtener logs (fallback HTTP)
  - `/api/logs/stats`: GET para estadísticas de logs
  - `/api/logs/modules`: GET para lista de módulos activos

### 5. Capa de Infraestructura

**`Docker`**  
- **Contenedores**:
  - `bot`: Contenedor principal con el ejecutable de Go
  - `nginx`: Proxy inverso para SSL y redirección HTTP a HTTPS
- **Red**: Red interna aislada (`terranova-net`) para comunicación entre contenedores
- **Volúmenes**: Persistencia para base de datos, logs y configuración

**`Nginx Proxy SSL`**  
- **Función**: Proxy inverso que termina las conexiones HTTPS y redirige el tráfico al contenedor del bot en el puerto 8080.
- **Certificados**: Generados con `mkcert` para tener una CA local de confianza.

**`Logs Rotativos`**  
- **Tecnología**: Implementación en Go con rotación por tamaño (10MB)
- **Configuración**: Mantiene hasta 5 archivos de backup, compresión opcional, eliminación por antigüedad (30 días).

### 6. Flujo de Datos

1. **Mensaje en Discord**: El usuario envía un mensaje en un servidor.
2. **Discord Gateway**: Envía el evento al bot a través de WebSocket.
3. **AutoMod Service**: Procesa el mensaje secuencialmente:
   - AntiSpam → ¿spam? → acción configurada
   - AntiLink → ¿link prohibido? → eliminar mensaje
   - WordFilter → ¿palabra prohibida? → acción configurada (borrar/advertir/silenciar)
4. **Logging Service**: Cada acción se registra con nivel INFO o WARN.
5. **WebSocket Server**: Distribuye los logs a todos los clientes conectados.
6. **Dashboard**: Muestra los logs en tiempo real con colores según nivel.

---

## 🛡️ Sistema de Moderación

### 📝 Filtro de Palabras
- Blacklist global de palabras prohibidas
- Palabras personalizables por servidor
- Detección de evasiones (leet speak, reemplazos)
- Acciones configurables: `borrar`, `advertir`, `silenciar`

### ⚠️ Anti-Spam
- Límite de mensajes por ventana de tiempo
- Configuración por servidor
- Cooldown para evitar spam de advertencias

### 🔗 Anti-Links
- Bloqueo de links no permitidos
- Whitelist de dominios confiables
- Detección de invites de Discord
- Opción de bloquear todos los links

### 🚨 Anti-Raid
- Detección de uniones masivas
- Activación automática de slowmode
- Notificaciones a administradores
- Acciones: `kick` o `ban`

### ⚖️ Sistema de Castigos
- Warnings progresivos
- Thresholds configurables
- Acciones: `mute`, `kick`, `ban`
- Reseteo automático de warnings

---

## 📊 Dashboard Web

### 🖥️ Páginas Principales

| Página | Descripción |
|--------|-------------|
| **Dashboard** | Visión general del sistema, gráficos de RAM, componentes activos |
| **AutoMod** | Configuración completa de módulos, gestión de blacklist y whitelist |
| **Logs** | Visualización en tiempo real, filtros por nivel y módulo |

### 🔧 Funcionalidades

- **Estadísticas en tiempo real**: RAM, componentes, logs
- **Configuración persistente**: Guardada en archivo JSON
- **Exportar/Importar**: Configuración completa en JSON
- **WebSocket**: Actualizaciones instantáneas
- **Diseño responsive**: Adaptable a móviles y tablets

---

## 🚀 Despliegue

### 📋 Requisitos

- **Docker** y **Docker Compose**
- **Raspberry Pi** (o cualquier servidor Linux)
- **Token de Discord Bot**

### ⚙️ Configuración

1. **Clonar el repositorio**
   ```bash
   git clone https://github.com/tuusuario/Terranova_Showcase.git
   cd Terranova_Showcase

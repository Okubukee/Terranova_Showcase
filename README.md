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
┌─────────────────────────────────────────────────────────────────────────────┐
│ TERRANOVA BOT │
├─────────────────────────────────────────────────────────────────────────────┤
│ │
│ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────────┐ │
│ │ Discord │ │ WebSocket │ │ Dashboard │ │
│ │ Gateway │◄──►│ Server │◄──►│ (HTTP/HTTPS) │ │
│ │ (API Real) │ │ (Tiempo Real) │ │ Puerto 80/443 │ │
│ └─────────────────┘ └─────────────────┘ └─────────────────────────┘ │
│ │ │ │ │
│ ▼ ▼ ▼ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ CORE SERVICES │ │
│ │ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌────────────┐ │ │
│ │ │ AutoMod │ │ Logging │ │ Commands │ │ Security │ │ │
│ │ │ │ │ │ │ │ │ │ │ │
│ │ │ • WordFilter │ │ • Rotación │ │ • Slash │ │ • Rate │ │ │
│ │ │ • AntiSpam │ │ de logs │ │ Commands │ │ Limiting │ │ │
│ │ │ • AntiLink │ │ • Niveles │ │ • Embeds │ │ • Audit │ │ │
│ │ │ • AntiRaid │ │ (DEBUG, │ │ • Permisos │ │ Logs │ │ │
│ │ │ • Warnings │ │ INFO, │ │ │ │ │ │ │
│ │ │ │ │ ERROR) │ │ │ │ │ │ │
│ │ └──────────────┘ └──────────────┘ └──────────────┘ └────────────┘ │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
│ ┌─────────────────────────────────────────────────────────────────────┐ │
│ │ PERSISTENCIA │ │
│ │ ┌──────────────┐ ┌──────────────┐ ┌──────────────────────────────┐ │ │
│ │ │ SQLite │ │ config.json │ │ blacklist.txt │ │ │
│ │ │ (Sorteos) │ │ (AutoMod) │ │ (Palabras prohibidas) │ │ │
│ │ └──────────────┘ └──────────────┘ └──────────────────────────────┘ │ │
│ └─────────────────────────────────────────────────────────────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────────┘

text

### 🔄 Flujo de Datos
Mensaje en Discord ──► Discord Gateway
│
▼

WebSocket Interno ◄──► AutoMod Service
│
├──► WordFilter ──► ¿Prohibido? ──► Eliminar
├──► AntiSpam ──► ¿Spam? ──► Advertir
├──► AntiLink ──► ¿Link? ──► Bloquear
└──► AntiRaid ──► ¿Raid? ──► Kick/Ban
│
▼

Logging Service ──► Guarda en RAM y Archivo
│
▼

WebSocket Server ──► Envía logs al Dashboard
│
▼

Dashboard (Browser) ◄──► Nginx (HTTPS) ◄──► API

text

### 📦 Estructura de Directorios
Terranova/
├── internal/
│ ├── commands/ # Comandos Slash (ping, giveaway, vote, lofi)
│ ├── events/ # Manejadores de eventos de Discord
│ ├── services/
│ │ ├── moderation/ # AutoMod (wordfilter, antispam, antilink, antiraid)
│ │ ├── logging/ # Sistema de logs con WebSocket
│ │ └── queue/ # Gestor de colas para rate limiting
│ ├── db/ # Base de datos SQLite (sorteos)
│ └── security/ # Auditoría y monitoreo
├── web/
│ ├── handlers/ # Endpoints API (automod, logs, stats)
│ └── static/ # HTML, CSS, JS del dashboard
├── nginx/
│ └── conf/ # Configuración del proxy SSL
├── logs/ # Archivos de logs rotativos
└── docker-compose.yml # Orquestación de contenedores

text

### 🧩 Componentes Principales

| Componente | Tecnología | Función |
|------------|------------|---------|
| **Bot Core** | Go + DiscordGo | Comunicación con Discord API |
| **AutoMod** | Go | Moderación automática |
| **Logging** | Go + WebSocket | Registro y transmisión de logs |
| **Dashboard** | HTML/CSS/JS + Chart.js | Interfaz de administración |
| **WebSocket Server** | Fiber + gorilla/websocket | Comunicación en tiempo real |
| **Proxy SSL** | Nginx | Terminación HTTPS |
| **Base de Datos** | SQLite | Persistencia de sorteos |
| **Contenedor** | Docker + Docker Compose | Despliegue y escalado |

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

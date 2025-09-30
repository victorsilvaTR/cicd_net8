# Overview proyecto Consola HR

## 1. Introducción

### Nombre del proyecto
**Consola Hyper Renta** - Sistema de gestión de licencias y activaciones para productos HyperRenta

### Contexto del proyecto

**Problema:** Existe una aplicación legacy que gestiona las licencias de software de los productos HyperRenta. Esta aplicación fue desarrollada con tecnologías antiguas. Se solicitó hacer un upgrade tecnológico manteniendo las funcionalidades, mejorándolas y agregando nuevas (https://dev.azure.com/tr-ggo/TAP%20Chile/_workitems/edit/783322).

**Solución:** Modernización tecnológica en una nueva aplicación .NET 8 que:
- **Mantiene** todas las funcionalidades core del sistema legacy (activación de licencias, gestión de claves)
- **Mejora** la experiencia de usuario con interfaz moderna y responsive
- **Agrega** nuevas funcionalidades:
  - Dashboard de métricas y reportes interactivos
  - Sistema robusto de perfiles y permisos
  - Integración con Salesforce para consulta de órdenes
  - Auditoría completa de todas las operaciones
  - Gestión multi-cliente mejorada
- **Moderniza** la arquitectura con tecnologías actuales y mejores prácticas

**Características Principales**:
- **Gestión de Clientes**: CRUD completo con validación de RUT chileno
- **Configuración de Licencias**: Gestión de máquinas, productos y certificados por cliente
- **Activación de Claves**: Activación, desactivación y actualización de claves de software
- **Gestión de Usuarios**: Sistema completo de autenticación y autorización
- **Perfiles y Permisos**: Control de acceso basado en roles y funcionalidades
- **Métricas y Reportes**: Dashboard con gráficos de IVA, empresas y contribuyentes
- **Auditoría Completa**: Trazabilidad de todas las operaciones mediante logs
- **Integración Salesforce**: Consulta de órdenes y datos comerciales


### Stakeholders claves
- Elizabeth.Cortez@thomsonreuters.com
- MariaGloria.Espinosa@thomsonreuters.com
---

## 2. Arquitectura y Tecnología

### Tecnologías principales utilizadas

**Backend:**
- `.NET 8` - Framework principal
- `ASP.NET Core MVC` - Patrón MVC con Razor Pages
- `Entity Framework Core 8.0.15` - ORM para acceso a datos
- `ASP.NET Core Identity` - Sistema de autenticación y autorización
- `OneOf 3.0.271` - Manejo de tipos de retorno funcionales
- `HyperRenta.dll` - Biblioteca propietaria para gestión de licencias

**Frontend:**
- `Tailwind CSS 4.1.11` - Framework CSS utility-first
- `ApexCharts 5.3.2` - Librería de gráficos para métricas

**Base de datos:**
- `SQL Server` (Azure Cloud)
- Server: `sql-server-instance-prod-hr.0afb5158da99.database.windows.net`
- Database: `HRActivacionASI`

**Arquitectura:**
- Vertical Slice Architecture (Features)
- Dependency Injection nativa de .NET
- Middleware personalizado para logging


### Diagrama de arquitectura
```
┌─────────────────────────────────────────────────────────────┐
│                      Usuario Web (Browser)                  │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTPS
┌────────────────────────▼────────────────────────────────────┐
│              ASP.NET Core MVC App (.NET 8)                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Features (Vertical Slice Architecture)               │   │
│  │ - Cliente      - Configuracion   - Metrica           │   │
│  │ - Account      - Profile         - Login/Logout      │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Infrastructure                                       │   │
│  │ - Identity (Auth)  - Logger   - Security             │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ HyperRenta.dll (Control de Licencias)                │   │
│  │ - Encriptación   - CopyControl   - Productos         │   │
│  └──────────────────────────────────────────────────────┘   │
└────────────────────────┬────────────────────────────────────┘
                         │ Entity Framework Core
┌────────────────────────▼────────────────────────────────────┐
│     Azure SQL Server (sql-server-instance-prod-hr)          │
│                Database: HRActivacionASI                    │
│  - Clientes         - Productos      - Certificados         │
│  - Licencias        - Logs           - Métricas             │
│  - Identity/Users   - Perfiles       - Funcionalidades      │
└─────────────────────────────────────────────────────────────┘
```


### Estructura del código fuente (alto nivel)

```
ConsolaActivacionWeb/
├── Features/                        # Funcionalidades del negocio (Vertical Slices)
│   ├── Account/                     # Gestión de cuentas de usuario (CRUD)
│   │   ├── AccountController.cs
│   │   ├── AccountService.cs
│   │   ├── AccountModel.cs
│   │   ├── Database/                # Context + Entidades (HRProfile, HRFunctionality)
│   │   └── *.cshtml                 # Vistas (Create, Update, Delete, Index)
│   ├── Cliente/                     # Gestión de clientes
│   │   ├── ClienteController.cs
│   │   ├── ClienteService.cs
│   │   ├── ClienteModel.cs
│   │   ├── Database/                # Context + Entidades (HRCliente, HRClienteDetalle)
│   │   └── *.cshtml
│   ├── Configuracion/               # Core del sistema: Licencias y activaciones
│   │   ├── ConfiguracionController.cs
│   │   ├── ConfiguracionService.cs
│   │   ├── ConfiguracionModel.cs
│   │   ├── HyperRenta.dll           # Biblioteca propietaria de licencias
│   │   ├── Database/                # Context + Entidades (HRProdCli, HRCfgCer, HRRegClave)
│   │   └── *.cshtml                 # Vistas (Maquinas, Productos, Certificados, Claves)
│   ├── Metrica/                     # Dashboard de métricas y reportes
│   │   ├── MetricaController.cs
│   │   ├── MetricaService.cs
│   │   ├── Database/                # Context + Entidades (IVA, ADM, RAD métricas)
│   │   └── Index.cshtml
│   ├── Profile/                     # Gestión de perfiles y permisos
│   ├── Login/                       # Autenticación
│   ├── Logout/                      # Cierre de sesión
│   ├── Home/                        # Página principal y menú
│   └── Shared/                      # Vistas compartidas (_Layout, _Nav, _Error)
│
├── Infrastructure/                  # Servicios transversales
│   ├── Identity/                    # ASP.NET Core Identity (ApplicationUser, IdentityContext)
│   ├── Logger/                      # Sistema de logs y auditoría (LoggerMiddleware)
│   ├── Security/                    # Validación de permisos y seguridad
│   └── Extensions/                  # Utilidades (ej: validación de RUT chileno)
│
├── wwwroot/                         # Archivos estáticos
│   ├── css/                         # Tailwind CSS (input.css, output.css)
│   ├── img/                         # Imágenes (logo.png)
│   └── favicon.ico
│
├── App.csproj                       # Configuración del proyecto .NET 8
├── App.sln                          # Solución de Visual Studio
├── Program.cs                       # Punto de entrada y configuración DI
├── appsettings.json                 # Configuración (connection strings, logging)
├── package.json                     # Dependencias frontend (Tailwind, ApexCharts)
└── global.json                      # Versión del SDK .NET
```

### Patrón arquitectónico: Vertical Slice Architecture

El proyecto implementa **Vertical Slice Architecture**, organizando el código por funcionalidad de negocio en lugar de por tipo técnico:

**Estructura de cada Feature:**
```
Features/Cliente/
├── ClienteController.cs      # Controlador MVC
├── ClienteService.cs          # Lógica de negocio
├── ClienteModel.cs            # DTOs/ViewModels
├── Database/                  # DbContext + Entidades
│   ├── Context.cs
│   └── HRCliente.cs
└── *.cshtml                   # Vistas Razor
```

**Beneficios:**
- Alta cohesión: código relacionado agrupado
- Bajo acoplamiento: features independientes
- Mantenibilidad: cambios localizados en una carpeta
- Escalabilidad: nuevas features no afectan existentes

### Capas de la aplicación

#### 1. **Presentación (Controllers + Views)**
- Reciben requests HTTP
- Validan entrada básica
- Invocan servicios de negocio
- Retornan vistas Razor o JSON (AJAX)
- **Características**: `[Authorize]`, `[AutoValidateAntiforgeryToken]`

#### 2. **Lógica de Negocio (Services)**
- Implementan reglas de negocio
- Validaciones (ej: RUT chileno)
- Coordinación de operaciones
- Transformación de datos
- **Patrón Result**: Uso de `OneOf<Success, string>` para manejo de errores

#### 3. **Acceso a Datos (DbContext + Entities)**
- Entity Framework Core 8
- DbContext por Feature: Separación de concerns
- Database-First approach
- Migraciones independientes por contexto

### Flujo de datos (Request Pipeline)

```
1. Browser (HTTPS) 
         ↓
2. Middleware Pipeline
   - LoggerMiddleware (auditoría)
   - Authentication (validar sesión)
   - Authorization (verificar permisos)
   - Anti-Forgery (CSRF)
         ↓
3. Controller → Service → DbContext
         ↓
4. SQL Server (Azure)
         ↓
5. ← Response (View Razor o JSON)
```

### Infraestructura transversal

**Identity (Autenticación/Autorización):**
- ASP.NET Core Identity con usuarios extendidos (`ApplicationUser`)
- Cookie authentication con sliding expiration
- Sistema propio de Perfiles y Funcionalidades
- Control de acceso granular por feature

**Logger (Auditoría):**
- `LoggerMiddleware`: registra todas las peticiones HTTP
- Tabla `HRLogAppWeb`: logs de aplicación
- Tabla `HRLOGACT`: logs específicos de activaciones
- Asociación automática con usuario autenticado

**Security:**
- Validación de permisos basada en perfiles
- Extensiones de validación (RUT chileno)
- Servicios de seguridad compartidos

### Patrones de diseño implementados

1. **Dependency Injection**: Todos los servicios registrados en IoC container
2. **Result Pattern**: `OneOf<T>` para retornos tipo Either (éxito/error)
3. **Repository Pattern** (implícito): DbContext actúa como Unit of Work
4. **ViewModel/DTO Pattern**: Separación entidades BD vs objetos de transferencia
5. **Middleware Pipeline**: Cadena de responsabilidad para procesamiento de requests

### Seguridad en la arquitectura

- **Authentication First**: Todos los controllers requieren `[Authorize]`
- **CSRF Protection**: Tokens anti-forgery en todos los formularios
- **SQL Injection**: Protección automática con EF Core (queries parametrizadas)
- **XSS Protection**: Razor sanitiza salidas automáticamente
- **HTTPS Only**: Redirección obligatoria a conexiones seguras

### Optimizaciones de rendimiento

- `AsNoTracking()`: Consultas de solo lectura sin tracking de cambios
- `async/await`: Todas las operaciones de I/O son asíncronas
- Connection Pooling: Por defecto en EF Core
- Compiled Queries: EF Core compila y cachea automáticamente

### Integraciones con sistemas externos

1. **Salesforce:**
   - Consulta de órdenes de venta (`HROrdenSalesforce`)

2. **HyperRenta.dll (Sistema Propietario):**
   - `HyperRenta.Encriptacion` - Encriptación de claves y certificados
   - `HyperRenta.CopyControl.Clave` - Gestión de claves de activación
   - `HyperRenta.CopyControl.Producto` - Configuración de productos
---
## 3. Operación y Mantenimiento

### Procedimientos básicos

**Onboarding Técnico:**
1. **Configurar entorno de desarrollo local**:
   - Visual Studio 2022 o superior / VSCode
   - .NET 8 SDK
   - SSMS / Azure Data Studio
   - Node.js (para Tailwind CSS)

2. **Clonar repositorio y restaurar dependencias**:
   ```bash
   git clone https://github.com/tr/ConsolaActivacionWeb.git
   cd .\ConsolaActivacionWeb\
   dotnet restore
   npm install
   ```

3. **Configurar conexión a base de datos de desarrollo**:
   - Actualizar `appsettings.json` con connection string de desarrollo
   - Ejecutar migrations si es necesario

**Deploy:**

  ```bash
  dotnet publish .\App.csproj -c Release -o .\branch
  ```
- Aplicación web ASP.NET Core desplegada en Azure/servidor web Windows
- Connection string configurada en `appsettings.json`
- Requiere acceso a SQL Server en Azure


**Monitoreo:**
- Logs de aplicación mediante `LoggerMiddleware`
- Tabla `HRLogAppWeb` registra todas las operaciones
- Logs de activación en tabla `HRLOGACT`

**Backups:**
- Base de datos SQL Server en Azure (gestión de backups según políticas de Azure)


---
## 4. Estado Actual del Proyecto

### Módulos o funcionalidades implementadas

**Gestión de Clientes (`/Cliente`):**
- CRUD completo de clientes
- Validación de RUT chileno
- Gestión de detalles de cliente

**Configuración de Licencias (`/Configuracion`):**
- Gestión de máquinas por cliente (crear, consultar)
- Configuración de productos por máquina
- Activación/desactivación de claves de software
- Actualización de claves
- Consulta de facturas (integración Salesforce)
- Historial de activaciones

**Gestión de Usuarios (`/Account`):**
- CRUD de cuentas de usuario
- Sistema de autenticación (login/logout)
- Integración con ASP.NET Core Identity

**Gestión de Perfiles (`/Profile`):**
- CRUD de perfiles de usuario
- Asignación de funcionalidades por perfil
- Control de acceso basado en roles

**Métricas y Reportes (`/Metrica`):**
- Métricas de IVA (archivos generados)
- Empresas por estado (activas, inactivas, eliminadas)
- Empresas creadas por mes
- Filtros por año y RUT
- Gráficos interactivos con ApexCharts

**Infraestructura:**
- Sistema de logging centralizado
- Middleware de auditoría
- Servicios de seguridad
- Extensiones de validación (RUT)

---
## 5. Repositorios y Documentación del Proyecto

### Repositorios de código
- **GitHub:** `https://github.com/tr/ConsolaActivacionWeb`
- **Branch principal:** `main`

### Otros artefactos relevantes
- `package.json` - Dependencias de Node.js (Tailwind, ApexCharts)
- `global.json` - Configuración de SDK de .NET
- `HyperRenta.dll` - Biblioteca propietaria (ubicada en `Features/Configuracion/`)

## 6. Próximos Pasos (5 min)

### Acciones inmediatas para el nuevo equipo/responsables

- Término de mejoras al sistema en desarrollo

---
## 7. Espacio de Preguntas (5 min)

Espacio abierto para preguntas, dudas, aclaraciones y comentarios del equipo.

---


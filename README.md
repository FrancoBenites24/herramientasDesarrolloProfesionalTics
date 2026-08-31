# Tarea Académica Grupal: Despliegue de Workflows con Docker, n8n y GitHub

Proyecto colaborativo de automatización de procesos mediante flujos de trabajo en **n8n**, desplegado localmente en contenedores con **Docker Compose** y versionado bajo la estrategia **GitHub Flow**.

---

## 👥 Integrantes y Roles

| Integrante | Rol Principal | Rama de Trabajo |
| :--- | :--- | :--- |
| **Joshelyn Ruiz** | Desarrollo de Flujos (n8n Workflows) | `feature/joshelynruiz` |
| **Grabiel Carnero** | Infraestructura y Despliegue (Docker / Git) | `feature/GrabielCarnero` |
| **Franco Benites** | Integración y Validación de Flujos | `feature/franco` |

---

## 📌 Descripción del Workflow Automatizado

El flujo implementado automatiza la **captura y gestión de prospectos (leads) inmobiliarios** a través de **4 nodos** encadenados:

1. **Formulario Consulta (`n8n-nodes-base.formTrigger`):** Captura los datos del cliente interesado (Nombre, Email, Teléfono, Tipo de propiedad, Presupuesto y Mensaje).
2. **Normalizar Datos (`n8n-nodes-base.set`):** Limpia, valida y da formato a las variables recibidas, agregando fecha y estado inicial (`Nuevo`).
3. **Guardar en Google Sheets (`n8n-nodes-base.googleSheets`):** Registra una nueva fila con la información del lead en la hoja de cálculo corporativa.
4. **Notificar por Telegram (`n8n-nodes-base.telegram`):** Envía un mensaje formateado con los detalles del lead al canal/chat del equipo comercial.

---

## 📁 Estructura del Repositorio

```text
├── workflows/
│   ├── formulario-inicial.json    # Versión inicial del formulario
│   ├── formulario-sheets.json     # Versión intermedia con integración a Google Sheets
│   └── formulario-completo.json   # Versión final (Formulario + Sheets + Telegram)
├── docker-compose.yml             # Orquestación del servicio n8n y volumen persistente
├── .gitignore                     # Exclusión de credenciales y archivos del sistema
├── .env.example                   # Plantilla de variables de entorno
└── README.md                      # Documentación del proyecto y bitácora de errores
```

---

## 🐳 Despliegue con Docker Compose

El despliegue se realiza exclusivamente mediante **`docker-compose.yml`** utilizando la imagen oficial de n8n y un volumen persistente para garantizar que las credenciales, workflows y ejecuciones se conserven entre reinicios.

### 📋 Prerrequisitos
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado y en ejecución.
- [Git](https://git-scm.com/) instalado.

### 🚀 Pasos para levantar el proyecto localmente

1. **Clonar el repositorio:**
   ```bash
   git clone https://github.com/FrancoBenites24/herramientasDesarrolloProfesionalTics.git
   cd herramientasDesarrolloProfesionalTics
   ```

2. **Iniciar n8n con Docker Compose:**
   ```bash
   docker compose up -d
   ```
   > El parámetro `-d` levanta el contenedor en segundo plano (detached mode).

3. **Acceder a n8n:**
   Abre tu navegador web e ingresa a:
   ```text
   http://localhost:5678
   ```
   *(En el primer ingreso, completa la configuración de la cuenta de usuario administrador).*

4. **Importar el Workflow en n8n:**
   - En el menú principal de n8n, haz clic en **Workflows** > **Add Workflow**.
   - Haz clic en el menú superior derecho (`...`) y selecciona **Import from File**.
   - Selecciona el archivo [`workflows/formulario-completo.json`](workflows/formulario-completo.json).
   - Configura tus credenciales de Google Sheets y Telegram Bot Token para activarlo.

5. **Detener el servicio:**
   ```bash
   docker compose down
   ```

---

## 💾 Persistencia de Datos con Volúmenes Docker

En el archivo `docker-compose.yml`, se ha configurado el volumen nombrado `n8n_data`:

```yaml
volumes:
  - n8n_data:/home/node/.n8n
```

### ¿Por qué es importante?
- **Sin volumen:** Al detener o eliminar el contenedor (`docker compose down`), todas las credenciales, configuraciones y flujos guardados se borrarían al destruirse la capa de escritura del contenedor.
- **Con volumen `n8n_data`:** Los datos de la base de datos interna SQLite de n8n se guardan en el host gestionado por Docker, permitiendo actualizar o recrear el contenedor sin pérdida de información.

---

## 🌿 Estrategia de Versionamiento (GitHub Flow)

Para garantizar la autoría, trazabilidad y buenas prácticas:

1. **Ramas por integrante:** Cada miembro del equipo trabaja en su propia rama (`feature/nombre`).
2. **Commits progresivos:** Commits atómicos y descriptivos que reflejan la evolución gradual del proyecto.
3. **Pull Requests y Revisiones Cruzadas:** Para integrar cambios a `main`, se crea un Pull Request que debe ser revisado y aprobado por otro compañero con comentarios de retroalimentación.

---

## 🛠️ Bitácora de Solución de Errores (Troubleshooting)

| # | Problema Detectado | Causa Raíz | Solución Aplicada |
| :-: | :--- | :--- | :--- |
| **1** | Error `Bind for 0.0.0.0:5678 failed: port is already allocated` al ejecutar `docker compose up`. | El puerto `5678` estaba ocupado por otra instancia previa de n8n o un servicio local. | Se identificó el proceso bloqueador con `netstat -ano \| findstr :5678` y se finalizó la tarea anterior, o se puede mapear temporalmente a otro puerto en `docker-compose.yml` (`"5679:5678"`). |
| **2** | Las credenciales y flujos desaparecían al reiniciar el contenedor. | Inicialmente no se había definido un volumen permanente para el directorio `/home/node/.n8n`. | Se configuró el volumen nombrado `n8n_data:/home/node/.n8n` en la sección `volumes` de `docker-compose.yml`. |
| **3** | El trigger del formulario no devolvía respuesta personalizada tras enviar los datos. | El nodo `Formulario Consulta` tenía configurada la respuesta inmediata por defecto en lugar de esperar el procesamiento completo. | Se ajustó el parámetro `responseMode: "lastNode"` para que el usuario reciba la confirmación una vez ejecutado el flujo. |
| **4** | Error de autenticación 401 en el nodo de Telegram. | Token de bot de Telegram mal configurado o no definido en las credenciales del nodo. | Se creó el bot correspondiente mediante `@BotFather`, se obtuvo el API Token y se vinculó en las credenciales del nodo Telegram. |
| **5** | Archivos temporales o `.env` intentando subirse al repositorio Git. | Falta de exclusión de archivos locales en el control de versiones. | Se creó y configuró un archivo `.gitignore` robusto que omite archivos `.env`, directorios `.n8n/` y archivos temporales del sistema operativo. |

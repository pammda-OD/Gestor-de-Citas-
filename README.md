# Gestor de Citas — Clínica San Martín

Sistema web para la gestión de citas médicas de la **Clínica San Martín**. Permite a los pacientes registrarse, buscar especialidades y médicos, agendar/modificar/cancelar citas, consultar su historial y usar un asistente virtual con inteligencia artificial para reservar citas de forma conversacional.

---

## 1. Tecnologías y Arquitectura

| Capa | Tecnología |
|------|------------|
| **Backend** | Java 21 + Spring Boot 3.5.6 + Spring Security + JPA/Hibernate |
| **Base de datos** | MySQL 8 |
| **Frontend** | HTML5, CSS3, Bootstrap 5, JavaScript vanilla |
| **IA / Chatbot** | Groq API (modelo Llama 3.3 70B) |
| **Build** | Maven |
| **Despliegue** | Railway (`https://clinicabot-production.up.railway.app`) |

**Arquitectura:** Patrón MVC en capas (Layered Architecture):
- `Controller` → Expone endpoints REST y sirve vistas.
- `Service` → Lógica de negocio (validaciones, reglas de citas, integración con IA).
- `Repository` → Acceso a datos (JpaRepository).
- `Model` → Entidades JPA mapeadas a MySQL.

---

## 2. Estructura del Proyecto

```
Gestor-de-citas/
├── pom.xml                              # Dependencias Maven
├── .env                                 # Variables de entorno (local)
├── src/main/java/com/clinica/gestor_citas/
│   ├── GestorCitasApplication.java      # Punto de entrada Spring Boot
│   ├── config/
│   │   ├── SecurityConfig.java          # Seguridad, CORS, rutas públicas
│   │   └── GroqConfig.java              # RestTemplate para API Groq
│   ├── controller/                      # Endpoints REST
│   │   ├── CitaController.java
│   │   ├── ChatbotController.java
│   │   ├── EspecialidadController.java
│   │   ├── HomeController.java
│   │   ├── HorarioController.java
│   │   ├── MedicoController.java
│   │   └── UsuarioController.java
│   ├── service/                         # Lógica de negocio
│   │   ├── CitaService.java
│   │   ├── ChatbotService.java
│   │   ├── EspecialidadService.java
│   │   ├── GroqChatService.java
│   │   ├── HorarioService.java
│   │   ├── MedicoService.java
│   │   └── UsuarioService.java
│   ├── repository/                      # Acceso a BDD (JPA)
│   │   ├── CitaRepository.java
│   │   ├── EspecialidadRepository.java
│   │   ├── HorarioRepository.java
│   │   ├── MedicoRepository.java
│   │   └── UsuarioRepository.java
│   └── model/                           # Entidades y DTOs
│       ├── Usuario.java
│       ├── Medico.java
│       ├── Especialidad.java
│       ├── Horario.java
│       ├── Cita.java
│       ├── CitaRequest.java / CitaUpdateRequest.java / CitaExtraida.java
│       └── ChatMessage.java / ChatRequest.java / ChatResponse.java
├── src/main/resources/
│   ├── application.yml                  # Configuración principal (BDD, puerto, Groq)
│   └── static/                          # Frontend estático
│       ├── html/                        # Vistas
│       │   ├── inicio.html
│       │   ├── login.html
│       │   ├── registro.html
│       │   ├── perfil.html
│       │   ├── cita.html
│       │   └── MisCitas.html
│       ├── css/                         # Estilos por vista
│       ├── js/                          # Lógica del frontend y chatbot
│       │   ├── chatbot.js
│       │   ├── inicio.js
│       │   ├── login.js
│       │   ├── registro.js
│       │   ├── perfil.js
│       │   ├── confirmationCita.js
│       │   ├── MisCitas.js
│       │   └── config.js
│       └── img/                         # Imágenes de médicos, logo, etc.
└── src/test/                            # Tests unitarios
```

---

## 3. Conexión a Base de Datos

**MySQL 8** configurado en `src/main/resources/application.yml`:

```yaml
spring:
  datasource:
    url: ${MYSQL_URL}              # Ej: jdbc:mysql://localhost:3306/clinica
    username: ${MYSQL_USERNAME}    # Ej: root
    password: ${MYSQL_PASSWORD}
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: update            # Crea/actualiza tablas automáticamente
    show-sql: true
    database-platform: org.hibernate.dialect.MySQL8Dialect
```

**Variables de entorno requeridas** (ver `.env`):
- `MYSQL_URL`, `MYSQL_USERNAME`, `MYSQL_PASSWORD`
- `GROQ_API_KEY`

**Entidades principales mapeadas:**
- `Usuario` → tabla de pacientes registrados
- `Medico` → médicos disponibles con su especialidad
- `Especialidad` → áreas médicas (Pediatría, Cardiología, etc.)
- `Horario` → disponibilidad de cada médico por fecha y hora
- `Cita` → reserva que vincula paciente, médico, especialidad y horario

---

## 4. Manual de Usuario

### 4.1 Registro e Inicio de Sesión
1. Abre la aplicación en tu navegador (`http://localhost:8080` o la URL de Railway).
2. Ve a **Registro** (`/html/registro.html`) y crea una cuenta con tu DNI, nombre, correo y contraseña.
3. Inicia sesión en **Login** (`/html/login.html`).

### 4.2 Agendar una Cita
1. En **Inicio** (`/html/inicio.html`) explora las especialidades médicas disponibles.
2. Selecciona un médico y revisa sus horarios disponibles.
3. Ve a **Reservar Cita** (`/html/cita.html`), elige fecha/hora y confirma.
4. Recibirás confirmación y podrás ver tu cita en **Mis Citas** (`/html/MisCitas.html`).

### 4.3 Consultar y Gestionar Citas
- **Mis Citas:** visualiza tu historial, modifica la fecha/hora o cancela citas pendientes.
- **Perfil** (`/html/perfil.html`): actualiza tus datos personales.

### 4.4 Usar el Asistente Virtual (Chatbot)
1. Haz clic en el ícono de chat (esquina inferior derecha).
2. Conversa con el bot para:
   - Conocer especialidades y médicos.
   - Recibir recomendaciones según síntomas.
   - **Reservar citas directamente** si ya iniciaste sesión: el bot puede filtrar especialidades, seleccionar médicos y redirigirte al formulario de cita.
3. Si no has iniciado sesión, el bot te pedirá que ingreses para ejecutar acciones de reserva.

---

## 5. Manual del Desarrollador

### 5.1 Requisitos Previos
- Java 21 JDK
- Maven 3.9+
- MySQL 8+ (local o remoto)
- Cuenta en [Groq](https://groq.com/) para obtener API Key (opcional si no usas el chatbot)

### 5.2 Configuración Local
1. Clona el repositorio.
2. Crea o edita el archivo `.env` en la raíz:
   ```
   MYSQL_URL=jdbc:mysql://localhost:3306/clinica
   MYSQL_USERNAME=root
   MYSQL_PASSWORD=admin
   GROQ_API_KEY=gsk_xxxxxxxx
   ```
3. Crea la base de datos en MySQL:
   ```sql
   CREATE DATABASE clinica;
   ```
4. Ejecuta desde la raíz del proyecto:
   ```bash
   ./mvnw spring-boot:run        # Linux / Mac
   mvnw.cmd spring-boot:run      # Windows
   ```
5. La aplicación levantará en `http://localhost:8080`. Las tablas se crean automáticamente con `ddl-auto: update`.

### 5.3 Endpoints REST Principales

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| `POST` | `/api/usuarios/registro` | Registrar nuevo usuario |
| `POST` | `/api/usuarios/login` | Iniciar sesión (sesión HTTP) |
| `GET`  | `/api/medicos` | Listar médicos |
| `GET`  | `/api/especialidades` | Listar especialidades |
| `GET`  | `/api/horarios` | Listar horarios disponibles |
| `POST` | `/api/citas` | Crear cita |
| `GET`  | `/api/citas/usuario/{id}` | Citas por usuario |
| `PUT`  | `/api/citas/{id}` | Actualizar cita |
| `DELETE` | `/api/citas/{id}` | Cancelar cita |
| `POST` | `/api/chatbot/chat` | Enviar mensaje al chatbot IA |

### 5.4 Seguridad y CORS
- `SecurityConfig.java` desactiva CSRF, form login y HTTP Basic.
- Todas las rutas API (`/api/**`) y recursos estáticos son públicas (`permitAll()`).
- La autenticación de usuarios es **manual por sesión HTTP** (manejada en `UsuarioController` / `UsuarioService`), no usa JWT ni OAuth2.
- CORS habilitado para `http://localhost:8080` y el dominio de producción en Railway.

### 5.5 Flujo de la Integración con IA (Groq)
1. `chatbot.js` (frontend) envía el mensaje del usuario a `ChatbotController` (`POST /api/chatbot/chat`).
2. `ChatbotService` construye un contexto dinámico con las especialidades y médicos actuales de la BD.
3. `GroqChatService` envía el `systemPrompt` + historial de conversación a la API de Groq.
4. Groq responde con un JSON que puede incluir una `action`:
   - `FILTRAR_ESPECIALIDAD`
   - `SELECCIONAR_MEDICO`
   - `REDIRIGIR_CITA`
   - `CONFIRMAR_CITA`
5. El frontend interpreta la `action` y redirige o filtra la interfaz automáticamente.

### 5.6 Convenciones de Código
- Entidades JPA en `model` con Lombok (getters/setters).
- Repositorios extienden `JpaRepository`.
- Servicios contienen la lógica de negocio; controladores solo delegan.
- Frontend completamente estático dentro de `src/main/resources/static/`.

---

## 6. Despliegue (Railway)
La aplicación está configurada para desplegarse en Railway. Define las siguientes variables de entorno en el panel de Railway:

| Variable | Descripción |
|----------|-------------|
| `MYSQL_URL` | URL de conexión a MySQL (incluye base de datos) |
| `MYSQL_USERNAME` | Usuario de MySQL |
| `MYSQL_PASSWORD` | Contraseña de MySQL |
| `GROQ_API_KEY` | API Key de Groq para el chatbot |

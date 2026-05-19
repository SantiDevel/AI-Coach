# 🏋️ AI-Coach — Asistente de Entrenamiento con Inteligencia Artificial

> Proyecto Final · Especialització IA + Big Data · AWS Bedrock  
> Equipo: **Javier Peiro** (Frontend + Docker) · **Santiago** (Backend + IA)

---

## ¿Qué es AI-Coach?

AI-Coach es un asistente conversacional que genera **planes de entrenamiento personalizados** usando IA generativa. El usuario describe su condición física y objetivos, y el sistema crea un plan adaptado a su perfil en segundos.

El plan puede refinarse mediante **feedback conversacional**: si el resultado no convence, el usuario puede pedir ajustes ("hazlo menos intenso", "no tengo mancuernas") y la IA lo corrige manteniendo el contexto de la conversación.

**Dos modos disponibles:**
- 🏋️ **Modo Personal** — plan de pesas, cardio o mixto según peso, objetivo y días disponibles
- 🏅 **Modo Preparador Físico** — sesiones adaptadas por deporte (fútbol, baloncesto, natación, atletismo...), categoría y fase de temporada

---

## 🤖 Tecnología IA utilizada

| Componente | Detalle |
|---|---|
| **Plataforma** | AWS Bedrock |
| **Modelo** | Amazon Nova 2 Lite (`amazon.nova-2-lite-v1:0`) |
| **Tipo de IA** | IA Generativa — consume modelos fundacionales ya entrenados |
| **Técnica principal** | Prompt Engineering dinámico parametrizado |
| **Ajuste de resultados** | Historial de conversación con feedback del usuario |
| **Backend** | Python 3 + Flask |
| **Infraestructura** | Docker + Docker Compose |
| **Formato de respuesta** | JSON estructurado renderizado en el frontend |

### ¿Por qué AWS Bedrock y no entrenar un modelo propio?

Bedrock permite **consumir modelos ya entrenados** sin necesidad de infraestructura propia ni datos de entrenamiento. La diferencia clave:

| ML Clásico | AWS Bedrock |
|---|---|
| Tú entrenas el modelo | Tú consumes un modelo ya entrenado |
| Necesitas datasets propios | Solo necesitas diseñar buenos prompts |
| Infraestructura compleja | Llamada a una API |

> **Modelo elegido:** Amazon Nova 2 Lite — última versión disponible (dic. 2025), ventana de contexto de 1M tokens, disponible en `eu-south-2` (España). Model ID: `amazon.nova-2-lite-v1:0`

---

## 🏗️ Arquitectura del sistema

```
┌─────────────────────────────────────────────────────────┐
│                  USUARIO (Navegador)                    │
│       Rellena formulario en index.html                  │
└─────────────────────┬───────────────────────────────────┘
                      │  fetch() POST /generar-plan
                      ▼
┌─────────────────────────────────────────────────────────┐
│             BACKEND — Flask (Python)                    │
│  app.py recibe datos + historial de conversación        │
│  prompts.py construye el prompt dinámico por modo       │
└─────────────────────┬───────────────────────────────────┘
                      │  boto3 converse()
                      ▼
┌─────────────────────────────────────────────────────────┐
│                  AWS BEDROCK                            │
│          Modelo: Amazon Nova 2 Lite                     │
│   Procesa el prompt con contexto y genera el plan       │
└─────────────────────┬───────────────────────────────────┘
                      │  JSON con plan de entrenamiento
                      ▼
┌─────────────────────────────────────────────────────────┐
│          FRONTEND — HTML + CSS + JavaScript             │
│  Muestra el plan + permite dar feedback para ajustarlo  │
└─────────────────────────────────────────────────────────┘
```

---

## 📁 Estructura del proyecto

```
AI-Coach/
├── .env                        ← Credenciales AWS (NO subir a GitHub)
├── .gitignore
├── Dockerfile                  ← Imagen del contenedor Python + Flask
├── docker-compose.yml          ← Orquestación del servicio
├── requirements.txt
├── README.md
│
├── backend/
│   ├── app.py                  ← Servidor Flask + endpoint /generar-plan
│   └── prompts.py              ← Prompts dinámicos por modo y perfil
│
├── frontend/
│   ├── index.html              ← Interfaz principal
│   ├── style.css               ← Estilos y diseño responsive
│   └── app.js                  ← Lógica del formulario + fetch a Flask
│
└── docs/
    └── arquitectura.png        ← Diagrama del sistema para la presentación
```

### `.env` — nunca subir a GitHub
```
AWS_ACCESS_KEY_ID=tu_access_key
AWS_SECRET_ACCESS_KEY=tu_secret_key
AWS_DEFAULT_REGION=eu-south-2
```

### `Dockerfile`
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "backend/app.py"]
```

### `docker-compose.yml`
```yaml
version: '3.8'
services:
  ai-coach:
    build: .
    ports:
      - "5000:5000"
    env_file:
      - .env
    volumes:
      - ./frontend:/app/frontend
      - ./backend:/app/backend
    restart: unless-stopped
```

### `requirements.txt`
```
flask
flask-cors
boto3
python-dotenv
```

---

## 📥 Datos de entrada

**Modo Personal (Gimnasio):**
- Edad, peso (kg), altura (cm)
- Objetivo: ganar músculo / perder peso / resistencia / flexibilidad
- Nivel: principiante / intermedio / avanzado
- Días disponibles por semana (1–7)
- Equipamiento disponible: gimnasio completo / mancuernas / sin material

**Modo Preparador Físico:**
- Deporte: fútbol, baloncesto, natación, atletismo, ciclismo, tenis...
- Categoría: benjamín / alevín / infantil / juvenil / amateur / profesional
- Objetivo físico: fuerza, resistencia, velocidad, explosividad
- Fase de temporada: pretemporada / competición / recuperación
- Duración de la sesión (minutos)

## 📤 Salida generada por la IA

- Plan estructurado por días con ejercicios, series, repeticiones y descanso
- Indicación del grupo muscular o capacidad física trabajada
- Consejos de recuperación y nutrición básicos
- **Posibilidad de refinamiento**: el usuario puede pedir ajustes y la IA corrige el plan manteniendo el contexto

---

## 🗓️ Cronograma — 7 días de trabajo paralelo

> Javier y Santiago trabajan en paralelo desde el Día 1.  
> El punto de unión es el **contrato JSON** acordado antes de empezar.

### Contrato JSON (definir el Día 1)

Javier envía al backend:
```json
{
  "modo": "personal",
  "edad": 22, "peso": 75, "altura": 178,
  "objetivo": "ganar músculo", "nivel": "intermedio",
  "dias": 4, "equipamiento": "gimnasio completo",
  "historial": []
}
```

Santiago devuelve al frontend:
```json
{
  "plan_nombre": "Plan Hipertrofia 4 días",
  "dias": [
    {
      "dia": "Lunes",
      "grupo_muscular": "Pecho + Tríceps",
      "ejercicios": [
        { "nombre": "Press banca", "series": 4, "repeticiones": "8-10", "descanso": "90s" }
      ]
    }
  ],
  "consejos": ["Proteína: 2g/kg de peso", "Duerme 8 horas"]
}
```

| Día | 🔵 Javier — Frontend + Docker | 🟠 Santiago — Backend + IA |
|-----|-------------------------------|---------------------------|
| **1** | Acordar contrato JSON. Estructura HTML base: header, selector de modo, formulario de perfil | Acordar contrato JSON. Configurar AWS: cuenta, acceso a Bedrock, primera llamada a Nova 2 Lite con `boto3` |
| **2** | CSS completo: layout responsive, tarjetas de entrenamiento por día, colores por modo | `app.py`: servidor Flask básico con endpoint `/generar-plan` que recibe JSON y devuelve texto de Nova 2 Lite |
| **3** | JavaScript: lógica del formulario, validaciones, mock de datos JSON para desarrollar sin backend real | `prompts.py`: prompt dinámico modo personal — parametriza peso, objetivo, nivel, días y equipamiento |
| **4** | Función `renderPlan(json)`: parsear el JSON y pintar tarjetas visuales con ejercicios y series | Afinar prompt para que Nova 2 Lite devuelva **JSON válido** con la estructura del contrato. Manejo de errores |
| **5** | Modo Preparador Físico en la UI: selector de deporte, categoría, fase de temporada. Caja de feedback conversacional | `prompts.py`: prompt modo preparador físico. Lógica de historial de conversación para el feedback |
| **6** | `Dockerfile` + `docker-compose.yml`: contenerizar la app. Conectar `fetch()` real al Flask de Santiago | Pruebas end-to-end con el frontend de Javier. Ajustar prompts si los resultados no son consistentes |
| **7** | Pruebas del contenedor Docker, pulir diseño responsive, ensayo de la demo en vivo | Redactar reflexión crítica completa. Preparar explicación técnica para la presentación oral |

---

## ⚠️ Reflexión crítica — Limitaciones de la IA

**1. Alucinaciones**  
El modelo puede generar planes con ejercicios inadecuados para el nivel indicado o con parámetros incorrectos. El plan debe tratarse siempre como una sugerencia, nunca como una prescripción médica.

**2. Calidad del prompt**  
Un prompt mal construido produce planes genéricos o incoherentes. Gran parte del trabajo de este proyecto está en diseñar prompts que generen respuestas consistentes y útiles.

**3. Sesgos del modelo**  
Amazon Nova 2 Lite fue entrenado con datos de internet que pueden contener sesgos culturales o de género en las recomendaciones de ejercicio.

**4. Privacidad**  
Los datos del usuario (peso, edad, condición física) se envían a servidores de AWS en `eu-south-2` (España). En un entorno real habría que revisar las políticas de privacidad y valorar la anonimización de datos.

**5. Supervisión humana obligatoria**  
La IA no sustituye a un preparador físico ni a un médico. El plan generado debe revisarse por un profesional, especialmente si el usuario tiene lesiones o condiciones de salud específicas.

---

## 👥 Equipo

| Miembro | Área | Responsabilidad |
|---------|------|-----------------|
| **Javier** | Frontend + Docker | Interfaz HTML/CSS/JS, formulario, renderizado del plan, Dockerfile y docker-compose |
| **Santiago** | Backend + AWS Bedrock | `app.py` Flask, conexión a Nova 2 Lite, `prompts.py`, historial conversacional |

---

## 🚀 Estado del proyecto

- [x] Definición del problema y arquitectura
- [x] README y documentación
- [x] Estructura del repositorio y `.gitignore`
- [ ] Configuración AWS Bedrock + Nova 2 Lite
- [ ] `app.py` Flask + endpoint `/generar-plan`
- [ ] `prompts.py` — prompts dinámicos (modo personal + preparador físico)
- [ ] Historial de conversación con feedback
- [ ] Interfaz HTML/CSS/JS
- [ ] `Dockerfile` + `docker-compose.yml`
- [ ] Integración frontend ↔ Flask ↔ Bedrock
- [ ] Pruebas y ajustes finales
- [ ] Presentación oral

---

## ▶️ Cómo ejecutar el proyecto

> Requiere únicamente tener **Docker** instalado. No es necesario instalar Python ni dependencias en el sistema.

```bash
# 1. Crear el .env con tus credenciales AWS (ver sección .env más arriba)

# 2. Primera vez — construir y arrancar
docker compose up --build

# 3. Las siguientes veces
docker compose up

# 4. Abrir en el navegador
# http://localhost:5000

# 5. Parar el servidor
docker compose down
```

---

*Proyecto Final · Especialització IA + Big Data · AWS Bedrock*

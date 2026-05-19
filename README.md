# 🏋️ AI-Coach — Asistente de Entrenamiento con Inteligencia Artificial

> Proyecto Final · Especialització IA + Big Data · AWS Bedrock  
> Equipo: **Javier Peiro** (Frontend + Integración) · **Santiago** (Backend + IA)

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
| **Modelo** | Claude 3 Sonnet (Anthropic) |
| **Tipo de IA** | IA Generativa — consume modelos fundacionales ya entrenados |
| **Técnica principal** | Prompt Engineering dinámico parametrizado |
| **Ajuste de resultados** | Historial de conversación con feedback del usuario |
| **Backend** | Python 3 + Flask |
| **Formato de respuesta** | JSON estructurado renderizado en el frontend |

### ¿Por qué AWS Bedrock y no entrenar un modelo propio?

Bedrock permite **consumir modelos ya entrenados** (Claude, Titan, Llama) sin necesidad de infraestructura propia ni datos de entrenamiento. La diferencia clave:

| ML Clásico | AWS Bedrock |
|---|---|
| Tú entrenas el modelo | Tú consumes un modelo ya entrenado |
| Necesitas datasets propios | Solo necesitas diseñar buenos prompts |
| Infraestructura compleja | Llamada a una API |

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
                      │  boto3 invoke_model()
                      ▼
┌─────────────────────────────────────────────────────────┐
│                  AWS BEDROCK                            │
│             Modelo: Claude 3 Sonnet                     │
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
AWS_DEFAULT_REGION=us-east-1
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

| Día | 🔵 Javier — Frontend | 🟠 Santiago — Backend + IA |
|-----|----------------------|---------------------------|
| **1** | Acordar contrato JSON. Estructura HTML base: header, selector de modo, formulario de perfil | Acordar contrato JSON. Configurar AWS: cuenta, acceso a Bedrock, primera llamada a Claude con `boto3` |
| **2** | CSS completo: layout responsive, tarjetas de entrenamiento por día, colores por modo | Servidor Flask básico: endpoint `/generar-plan` funcional que recibe JSON y devuelve texto de Claude |
| **3** | JavaScript: lógica del formulario, validaciones, mock de datos JSON para desarrollar sin backend real | `prompts.py`: prompt dinámico modo personal — parametriza peso, objetivo, nivel, días y equipamiento |
| **4** | Función `renderPlan(json)`: parsear el JSON y pintar tarjetas visuales con ejercicios y series | Afinar prompt para que Bedrock devuelva **JSON válido** con la estructura del contrato. Manejo de errores |
| **5** | Modo Preparador Físico en la UI: selector de deporte, categoría, fase de temporada. Caja de feedback conversacional | `prompts.py`: prompt modo preparador físico. Lógica de historial de conversación para el feedback |
| **6** | Conectar `fetch()` real al endpoint Flask de Santiago. Pruebas end-to-end con Bedrock real | Pruebas end-to-end con el frontend de Javier. Ajustar prompts si los resultados no son consistentes |
| **7** | Pulir diseño responsive, captura de pantallas, ensayo de la demo en vivo | Redactar reflexión crítica completa. Preparar explicación técnica para la presentación oral |

---

## ⚠️ Reflexión crítica — Limitaciones de la IA

**1. Alucinaciones**  
El modelo puede generar planes con ejercicios inadecuados para el nivel indicado o con parámetros incorrectos. El plan debe tratarse siempre como una sugerencia, nunca como una prescripción médica.

**2. Calidad del prompt**  
Un prompt mal construido produce planes genéricos o incoherentes. Gran parte del trabajo de este proyecto está en diseñar prompts que generen respuestas consistentes y útiles.

**3. Sesgos del modelo**  
Claude fue entrenado con datos de internet que pueden contener sesgos culturales o de género en las recomendaciones de ejercicio.

**4. Privacidad**  
Los datos del usuario (peso, edad, condición física) se envían a servidores de AWS. En un entorno real habría que revisar las políticas de privacidad y valorar la anonimización de datos.

**5. Supervisión humana obligatoria**  
La IA no sustituye a un preparador físico ni a un médico. El plan generado debe revisarse por un profesional, especialmente si el usuario tiene lesiones o condiciones de salud específicas.

---

## 👥 Equipo

| Miembro | Área | Responsabilidad |
|---------|------|-----------------|
| **Javier** | Frontend + Integración | Interfaz HTML/CSS/JS, formulario, renderizado del plan |
| **Santiago** | Backend + AWS Bedrock | Flask, conexión a Bedrock, prompts, historial conversacional |

---

## 🚀 Estado del proyecto

- [x] Definición del problema y arquitectura
- [x] README y documentación
- [x] Estructura del repositorio y `.gitignore`
- [ ] Configuración AWS Bedrock
- [ ] Servidor Flask + endpoint `/generar-plan`
- [ ] Prompts dinámicos (modo personal + preparador físico)
- [ ] Historial de conversación con feedback
- [ ] Interfaz HTML/CSS/JS
- [ ] Integración frontend ↔ Flask ↔ Bedrock
- [ ] Pruebas y ajustes finales
- [ ] Presentación oral

---

## ▶️ Cómo ejecutar el proyecto

```bash
# 1. Instalar dependencias
pip install -r requirements.txt

# 2. Crear el .env con tus credenciales AWS

# 3. Arrancar el servidor Flask
python backend/app.py

# 4. Abrir en el navegador
# http://localhost:5000
```

---

*Proyecto Final · Especialització IA + Big Data · AWS Bedrock*

# Propuesta de Refactor Arquitectónico: Trust Boundary Security Core & Agentic Security Auditor

**Estado:** Aprobado para Implementación  
**Fecha:** 2026-09-16  
**Repositorio Base:** `security-engineer-skill` (`trust-boundary-security-engineer`)  
**Especialización:** `agentic-security-auditor`

---

## 1. Arquitectura Actual

El repositorio `security-engineer-skill` actúa como un framework de razonamiento de seguridad para agentes de IA gobernado por especificaciones formales (`SPEC-001` a `SPEC-005`):

```text
security-engineer-skill/
├── AGENTS.md          # Reglas de comportamiento, modificación por tiers y aprendizaje
├── SKILL.md           # Activación e indexación de carga de contexto
├── specs/             # Gobernanza, arquitectura y modelo de razonamiento (SPEC-001 a 005)
├── principles/        # Axiomas de evaluación (evidencia, suficiencia de control, etc.)
├── knowledge/         # Módulos conceptuales de seguridad
├── checklists/        # Procedimientos de verificación paso a paso
├── patterns/          # Patrones de riesgo arquitectónico y evolución de estado
├── examples/          # Casos de estudio prácticos con el ciclo de 6 pasos
└── references/        # Punteros a estándares externos (OWASP, STRIDE, etc.)
```

El núcleo operativo reside en **`specs/SPEC-005-agent-operational-behavior.md`**, que define el ciclo de razonamiento de 6 fases:
1. **Input Processing Layer:** Discriminación estricta de Señal vs. Ruido.
2. **Context Construction Model:** Identificación de Trust Boundaries espaciales y temporales, y clasificación del Execution Model.
3. **Hypothesis Generation Phase:** Modelado de State Evolution y filtrado por Two-Stage Pattern Filter.
4. **Evidence Evaluation Loop:** Evaluación estricta de evidencia observable, no-determinismo y emisión de `[REQUIRES_EXTERNAL_EVIDENCE]`.
5. **Decision Synthesis:** Calificación de certeza (Observado, Inferido, Incierto) y distinción entre conclusiones estructurales y dependientes del tiempo.
6. **Output Construction Layer:** Estructura en tres niveles (Observaciones, Inferencias, Riesgos) con proporcionalidad.

---

## 2. Problemas Encontrados

Durante la auditoría se identificaron los siguientes puntos de acoplamiento y mezcla de abstracción:

1. **Acoplamiento de Dominio en el Core:**
   - En `knowledge/`: `agent-trust-model.md` y `prompt-injection.md` viven junto a principios universales (`least-privilege.md`, `trust-boundaries.md`).
   - En `checklists/`: `agentic-system-review.md` convive con revisiones de código general.
   - En `references/`: `owasp-llm-top10.md` está mezclado con estándares universales.
2. **Violación de Abstracción en Principios Normativos (Tier 3):**
   - En `principles/control-sufficiency.md` se incrustó la sección *"Three-Layer Sufficiency Framework for Agentic Systems"*. Un principio normativo universal debe definir la suficiencia abstracta frente al modelo de ejecución, delegando la instanciación de capas específicas al dominio correspondiente.
3. **Falta de Protocolo de Precedencia y Extensión:**
   - No existía un contrato formal que regule cómo una extensión especializa el Core sin contradecir sus axiomas de razonamiento.
4. **Tratamiento de MAESTRO y Scripts:**
   - Riesgo de convertir MAESTRO en un checklist automático en lugar de una taxonomía de clasificación posterior al razonamiento.
   - Riesgo de que scripts de recolección de evidencia emitan juicios de valor concluyentes (`SECURE`, `SAFE`, `VULNERABLE`) basados en meras heurísticas.

---

## 3. Jerarquía y Arquitectura Propuesta

Se establece una arquitectura en dos capas formalmente desacopladas:

```text
       Trust Boundary Core (security-engineer-skill)
         [Stable / Technology-Agnostic Reasoning Core]
                            │
                            ▼
          Extension Protocol (SPEC-006)
         [Precedencia, Compatibilidad y Contratos]
                            │
                            ▼
      Agentic Security Auditor (agentic-security-auditor)
         [Domain-Specific Specialization & Evidence Collectors]
```

### Regla de Independencia Operacional
`Trust Boundary Core` **NO depende operacionalmente** de `agentic-security-auditor`. El Core es 100% autocontenido y funciona plenamente para auditar cualquier software tradicional sin que la extensión exista.

### Regla de Precedencia (SPEC-006)
Una extensión puede especializar y ampliar el Core, pero **NUNCA puede contradecir** sus axiomas, evidence model, execution model, trust-boundary model o control-sufficiency model sin modificar explícitamente el Core y sus especificaciones.

---

## 4. Flujo de Razonamiento y Clasificación

```text
Evidence (Datos observables directos / Salidas de scripts recolectores)
      │
      ▼
Trust Boundary Reasoning (SPEC-005: Límites Espaciales/Temporales, State Evolution, Execution Model)
      │
      ▼
Threat / Risk Identification (Suficiencia de Controles, Estructural vs Dependiente del Tiempo)
      │
      ▼
MAESTRO Mapping (Taxonomía de clasificación en las 7 capas arquitectónicas)
```

MAESTRO actúa como **capa de clasificación final**, no como motor de razonamiento ni como checklist ciego.

---

## 5. Principios de Primer Nivel Preservados en el Core

El Core conserva como axiomas inmutables (Tier 3):
1. **Evidence over Assumptions:** Hechos observables sobre suposiciones; ausencia de evidencia vs evidencia de ausencia.
2. **Control Sufficiency:** Control ausente vs control presente pero insuficiente bajo el modelo de ejecución.
3. **Execution Model:** Clasificación previa obligatoria (secuencial, concurrente, distribuido, mixto).
4. **Spatial Trust Boundaries:** Fronteras entre componentes, procesos, servicios y usuarios.
5. **Temporal Trust Boundaries:** Ventanas de vulnerabilidad en concurrencia y ciclo de vida de recursos.
6. **State Evolution:** Trazado de estado a lo largo de creación → mutación → consumo → descarte.
7. **Structural vs Time-dependent Conclusions:** Distinción rigurosa entre garantías estructurales y fallas dependientes del entrelazado temporal.
8. **Education over Fear:** Formación técnica constructiva en lugar de alarmismo.
9. **Human in the Loop:** Puntos de decisión y contención antes de acciones irreversibles.
10. **Minimal Change:** Recomendación del cambio más pequeño y seguro que resuelva la falla.

---

## 6. Responsabilidad y Mapeo de Archivos

### Contenido que permanece en Core (`security-engineer-skill`)
- `specs/SPEC-001` a `SPEC-005` + nuevo `specs/SPEC-006-skill-extension-protocol.md`.
- `principles/` (todos los 6 principios limpios de referencias tecnológicas particulares).
- `knowledge/` universal: `trust-boundaries.md`, `least-privilege.md`, `defense-in-depth.md`, `secure-by-default.md`, `secrets-management.md`, `input-validation.md`, `secure-serialization.md`, `supply-chain-security.md`, `threat-modeling.md`, `error-handling-secure-failure.md`.
- `checklists/`: `repository-entry-scan.md`, `security-code-review.md`, `secrets-management-review.md`, `threat-modeling-review.md`.
- `patterns/`: `PAT-001` a `PAT-005`.
- `examples/`: `authentication-bypass-review.md`, `concurrent-payment-review.md`, `iac-cloud-misconfiguration-review.md`, `talktalk-case-study.md`, `web-app-container-audit.md`.
- `references/`: `owasp-and-vulnerability-standards.md`, `threat-modeling-frameworks.md`, `cloud-and-infrastructure-security.md`.

### Contenido que migra y se crea en `agentic-security-auditor`
- **Knowledge especializado:**
  - `knowledge/agent-trust-model.md` (migrado y enriquecido con autoridad y delegación).
  - `knowledge/prompt-injection.md` (migrado: directa, indirecta, context, tool-result, RAG/memoria).
  - `knowledge/mcp-security.md` (NUEVO: capability boundary, schemas tipados, aislamiento stdio/SSE).
  - `knowledge/agent-authority-model.md` (NUEVO: A2A trust, mitigación de Confused Deputy, credenciales efímeras).
- **Checklists & References:**
  - `checklists/agentic-system-review.md` (migrado y mapeado con MAESTRO).
  - `checklists/mcp-tool-review.md` (NUEVO: revisión de herramientas MCP).
  - `references/maestro-framework.md` (NUEVO: especificación de la taxonomía de 7 capas).
  - `references/owasp-llm-top10.md` (migrado desde el core).
  - `references/docker-hardening.md` (NUEVO: evidencia de contención física en Capa 2).
  - `references/linux-systemd-hardening.md` (NUEVO: evidencia de aislamiento systemd en Capa 2).
- **Examples:**
  - `examples/prompt-injection-review.md` (migrado).
  - `examples/mcp-confused-deputy-review.md` (NUEVO: escalación de autoridad vía herramientas).
- **Evidence Collectors (`scripts/`):**
  - `audit_docker.py`: Recolección pasiva de configuración de contenedores.
  - `audit_network.py`: Recolección pasiva de listeners, bindings, puertos publicados e interfaces.
  - `audit_secrets.py`: Búsqueda heurística categorizada en `[PATTERN_MATCH]`, `[HEURISTIC_CANDIDATE]`, y `[CONFIRMED_SECRET]`.
  - **Regla estricta:** Ningún script emite veredictos finales como `SECURE`, `SAFE` o `VULNERABLE`.

---

## 7. Criterios de Validación y Aceptación

1. **Core Independence Test:** `security-engineer-skill` compila sus referencias y ejecuta revisiones tradicionales sin requerir la presencia de `agentic-security-auditor`.
2. **Extension Dependency Test:** `agentic-security-auditor` declara explícitamente su dependencia conceptual del Core y cumple la regla de precedencia de SPEC-006.
3. **Collector Non-Verdict Test:** Ningún script emite veredictos categóricos autónomos.
4. **Integridad de Enlaces y Sintaxis:** Cero hipervínculos rotos y 100% de scripts validados con el compilador de Python.

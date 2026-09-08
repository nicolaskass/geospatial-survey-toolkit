# geospatial-survey-toolkit

## Identidad
[COMPLETAR — qué es este proyecto, quién lo lleva, objetivo principal]

## Estado actual
→ Leer `contexto/estado.md`

## Stack
[COMPLETAR — lenguajes, frameworks, infraestructura]

## Dominio activado
ninguno

## Reglas del proyecto
[COMPLETAR — convenciones específicas de este proyecto]

## Lo que NO se hace aquí
[COMPLETAR — qué está fuera de scope]

---

## Cómo trabajamos en este proyecto

### Agentes disponibles
- (ninguno activado)

Los agentes globales (~/.claude/agents/) siempre están disponibles: @orchestrator, @thinker,
@coder, @planner, @analizer, @rizomapper, @rizocontext, @rizosynapse, @rizoplanner, etc.

### Inicio de sesión
Al abrir Claude Code aquí, el hook inyecta automáticamente `contexto/estado.md` y el
historial de sesiones. Claude presenta un brief de orientación antes del primer intercambio.

### Qué va a git en este proyecto
- `contexto/` — siempre (auto-commiteado por el hook Stop después de cada sesión)
- `.claude/CLAUDE.md` — siempre (define cómo trabajar aquí)
- `.claude/agents/` — siempre (agentes locales del proyecto)
- `.env`, `.env.*` — NUNCA (están en .gitignore)

### Workflow típico
1. Claude orienta con el estado actual al iniciar
2. Invocar el agente adecuado o dejar que @orchestrator lo elija
3. Al cerrar: Claude escribe resumen en `contexto/claude-chats/` y actualiza `estado.md`
4. El hook commitea `contexto/` automáticamente

---
_Generado por init-project.sh — completar los [COMPLETAR] antes de trabajar._

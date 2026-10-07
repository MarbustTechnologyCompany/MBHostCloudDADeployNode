# Cómo colaborar — MBHostCloud: Desplegar Node.js (guía)

El recorrido de cada cambio. Las reglas están en [`AGENTS.md`](AGENTS.md); la guía completa en [`README.md`](README.md) y la condensada en [`docs/DESPLIEGUE-AGENTES.md`](docs/DESPLIEGUE-AGENTES.md); si es tu primer día, parte de [`docs/ONBOARDING.md`](docs/ONBOARDING.md).

## El flujo, en orden (no se salta ningún paso)

1. **Tarjeta en Trello** — marca MBHostCloud, severidad, ejecutor, cómo probar.
2. **Issue** — lo abre `MarbustTechnologyCompany` con la plantilla. Se autocontiene. Skill `escribir-un-issue`.
3. **Rama** — desde `main`, nunca push directo a `main`.
4. **PR en borrador** — con la plantilla (`Refs #N`); al terminar, `Closes #N` y listo.
5. **Verificación real** — prueba en el servidor lo que cambió y **pega el output** en el PR.
6. **Revisión / QA** — antes del PR corre `revisar-codigo`. **Codex participa siempre.** Si el QA encuentra errores, la empresa comenta **solicitando cambios**; se corrige, se responde, y recién si pasa se aprueba.
7. **Aprobación** — la da `MarbustTechnologyCompany` (el autor no se auto-aprueba).
8. **Squash** — un issue, un PR, un commit.

## Verificación (antes de pedir revisión)

| Qué | Para qué |
|---|---|
| Probar en el servidor real de MBHostCloud | que los comandos/pasos nuevos o cambiados funcionan tal cual (Terminal del panel) |
| Render del Markdown | bloques de código cerrados, enlaces válidos |
| Coherencia con el hosting | socket Unix, pm2, sin puertos, proxy por el panel |

## Reglas que no se discuten dentro de un PR

- **Lo que afirma la guía se probó en el servidor real.** Nada "de memoria".
- **Sin secretos ni datos reales de clientes** (solo ejemplos ficticios); el `.env` fuera del repo con `chmod 600`.
- No promover editar Apache a mano ni abrir puertos.
- Novedad **solo interna** (repo interno): sin líneas `pública`/`hito`.
- Commits con tipo, en español. **Prohibida la co-autoría de IA** en commits y PRs.

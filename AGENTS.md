# Reglas del repositorio — MBHostCloud: Desplegar Node.js (guía)

Este archivo es para **todas las personas y agentes de IA** que trabajan en esta guía (Claude Code, Codex, Cursor, Copilot u otros). Si usas Claude Code, se carga solo a través de `CLAUDE.md`.

- Si es tu primer día: [`docs/ONBOARDING.md`](docs/ONBOARDING.md).
- Cómo colaborar paso a paso: [`CONTRIBUTING.md`](CONTRIBUTING.md).
- **La guía completa** (para el cliente): [`README.md`](README.md). **Versión condensada para agentes de IA:** [`docs/DESPLIEGUE-AGENTES.md`](docs/DESPLIEGUE-AGENTES.md).
- El estándar común de todos los repos de Marbust: `MarbustTechnologyCompany/.github` → `ESTANDAR-REPOSITORIOS.md`.

**Clase del repo: interno** (`.github/marbust.json`). **No publica novedades** al exterior; en los PRs solo va la línea *interna* de Novedad.

**Qué es:** una **guía** para clientes de MBHostCloud (hosting DirectAdmin + Apache) que quieren desplegar su propia app **Node.js / Next.js** en su cuenta, **como el usuario cliente, sin root**. La app corre bajo **pm2** en un **socket Unix** (sin puertos); Apache hace reverse-proxy del dominio al socket, enlazado por el panel (*Node App*) o el CLI `mbnode-deploy`. Es **contenido/documentación**, no código ejecutable.

> 🔴 **Todo lo que afirma la guía debe ser cierto en el servidor real.** El README lo dice: "Todo fue probado en el servidor real; los comandos funcionan tal cual". Un paso que no se verificó en el hosting de MBHostCloud **no entra**. Una guía que miente cuesta horas al cliente.

## Idiomas

- La guía está en **español** (Ecuador), con **tuteo**.
- Comandos y nombres técnicos en inglés donde corresponda.
- Issues, PRs y commits en español.

## Reglas duras

1. **Flujo:** tarjeta → issue (lo abre `MarbustTechnologyCompany`) → rama → PR en borrador → QA → aprobación de la empresa → squash. Nadie hace push directo a `main`. Detalle en [`CONTRIBUTING.md`](CONTRIBUTING.md).
2. **El issue se autocontiene.** Skill `escribir-un-issue`.
3. **Revisión obligatoria.** Antes del PR, corre `revisar-codigo`. **Codex participa siempre.**
4. **Verificado en el servidor real (regla que manda):** cada comando o paso nuevo/cambiado se prueba en el hosting de MBHostCloud (Terminal del panel) con la salida real; nada se afirma "de memoria". Es la razón de ser de esta guía.
5. **Coherencia con el hosting:** socket Unix, pm2, sin puertos TCP, proxy por el panel (no se edita Apache a mano). Si cambia el comportamiento del hosting, la guía cambia con él.
6. **Sin secretos ni datos de clientes:** solo ejemplos ficticios; el `.env` va fuera del repo con `chmod 600` y nunca a git — y la guía lo repite.
7. **Verificar:** el Markdown renderiza (bloques cerrados, enlaces válidos) y lo afirmado se probó en el servidor. El output va pegado en el PR.
8. **Commits** con tipo (`feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `perf`) y en español. **Prohibido** co-autoría de IA en commits y en PRs.
9. **Nada de estado escrito a mano** (fechas de "última actualización", TODOs de avance) en la guía; el avance vive en los issues.

## Seguridad (regla dura)

- **Sin secretos ni datos reales de clientes** en la guía: solo ejemplos ficticios.
- La guía **insiste** en que el `.env` va fuera del repo, con `chmod 600`, nunca a git.
- No promover editar Apache a mano ni abrir puertos (el hosting lo bloquea por seguridad).
- Una vulnerabilidad se reporta en privado: ver [`SECURITY.md`](SECURITY.md).

## Skills del repositorio

Hay **dos rutas**:

1. **Colaboradores nuevos o externos** usan las skills de este repo, en [`.claude/skills/`](.claude/skills). Son una **copia sincronizada** desde el directorio oficial por el Action `sync-skills`; **no se editan a mano aquí**.
2. **Colaboradores oficiales de Marbust** usan el **directorio oficial** (`MarbustTechnologyCompany/ClaudeSkills`, en `~/.claude/skills`). **Es la fuente de verdad.**

| Skill | Cuándo |
|---|---|
| `escribir-un-issue` | Al crear o corregir un issue |
| `trabajar-un-issue` | Al tomar un issue, de principio a fin |
| `revisar-codigo` | **Obligatoria** antes del PR y al revisar el de otro |

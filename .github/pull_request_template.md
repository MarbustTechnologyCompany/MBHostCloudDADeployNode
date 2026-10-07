<!-- Ábrelo como borrador (Draft) apenas empieces, con `Refs #<número>`. Cuando esté listo, cámbialo a `Closes #<número>` y márcalo como listo para revisión. La plantilla se llena completa también en borrador. -->

## Resumen

<!-- Qué cambió en la guía y por qué. -->

## Novedad

<!-- Este repo es INTERNO: no publica novedades al exterior. Solo la línea interna; "ninguna" si no aplica.
No pongas líneas "pública" ni "hito" (el check "Checks del PR" las rechaza en un repo interno). -->

- interna:

## Issue vinculado

<!-- `Closes #123` / `Refs #123`, en inglés. -->

## Cómo probar

<!-- Qué parte de la guía cambió y cómo confirmar que es correcta. -->

1.

## Verificación

<!-- Esto es una GUÍA: lo que afirma debe ser cierto en el servidor real de MBHostCloud. Pega la evidencia. -->

- [ ] Los comandos/pasos nuevos o cambiados fueron **probados en el servidor real de MBHostCloud** (Terminal del panel), con la salida real
- [ ] El Markdown renderiza bien (bloques de código cerrados, enlaces válidos)
- [ ] No contradice la realidad del hosting (socket Unix, pm2, sin puertos, proxy por el panel)

## Seguridad y operaciones

- [ ] No hay secretos, credenciales ni datos reales de clientes en la guía (solo ejemplos ficticios)
- [ ] La guía sigue diciendo que el `.env` va fuera del repo con `chmod 600` y nunca a git

---

> Este PR entra a `main` por **squash**. Un issue, un PR, un commit. Sin co-autoría de IA en commits ni en PRs.

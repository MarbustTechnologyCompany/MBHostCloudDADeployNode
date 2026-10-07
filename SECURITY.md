# Seguridad

Este repo es una **guía** de despliegue para clientes de MBHostCloud. Si encuentras una falla de seguridad en lo que la guía recomienda (o un dato sensible filtrado), gracias por avisarnos. **No la publiques en un issue:** escríbenos de forma privada.

## Cómo reportarla

Escribe a **supportcenter@marbust.com** con el asunto "Seguridad MBHostCloud DADeploy Node: …".

Incluye qué encontraste y dónde (sección, línea, commit) y el impacto.

## Lo que pedimos

- Prueba solo en una cuenta de hosting de prueba, nunca contra datos de clientes reales.
- Danos tiempo razonable para corregir antes de hacerlo público.

## Cómo se protege

- **La guía no contiene secretos ni datos reales de clientes:** solo ejemplos ficticios.
- **Insiste en buenas prácticas:** el `.env` va fuera del repo, con `chmod 600`, nunca a git; no se editan configs de Apache a mano ni se abren puertos (el hosting lo bloquea por seguridad).
- Si un paso de la guía indujera una mala práctica de seguridad, es un `[bug]` que bloquea.

## Versiones con soporte

Solo la rama `main`.

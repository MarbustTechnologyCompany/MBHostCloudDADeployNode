# Tu primer día en la guía de despliegue Node (MBHostCloud)

Guía para quien empieza a mantener esta guía. Si algo no alcanza, es un error de la guía: dilo en un issue.

## 1. Qué es, en un minuto

Una **guía** para clientes de MBHostCloud (DirectAdmin + Apache) que despliegan su propia app **Node.js / Next.js** en su cuenta, **como el usuario cliente, sin root**: app bajo **pm2** en un **socket Unix** (sin puertos), con Apache haciendo reverse-proxy del dominio al socket (enlace por el panel o `mbnode-deploy`). Es **documentación**, no código ejecutable.

- 🔴 **Todo lo que afirma se probó en el servidor real.** Un paso sin verificar no entra.
- **Sin secretos ni datos de clientes** (solo ejemplos ficticios); el `.env` fuera del repo, `chmod 600`.
- **No se edita Apache a mano** ni se abren puertos (el hosting lo bloquea).

## 2. Qué leer, en este orden

1. [`README.md`](../README.md) — la guía completa (troubleshooting incluido).
2. [`docs/DESPLIEGUE-AGENTES.md`](DESPLIEGUE-AGENTES.md) — versión condensada y accionable para un agente.
3. Esta guía.
4. [`CONTRIBUTING.md`](../CONTRIBUTING.md) — el recorrido de cada cambio.
5. [`AGENTS.md`](../AGENTS.md) — reglas de la guía y skills.
6. Las skills de [`.claude/skills/`](../.claude/skills) (copia para colaboradores nuevos; los oficiales usan el directorio de la empresa).
7. [`SECURITY.md`](../SECURITY.md).

## 3. Accesos que debes pedir

Los concede Marco Antonio Bustillos (indica tu usuario de GitHub y correo):

| Acceso | Para qué |
|---|---|
| Colaborador del repo `MarbustTechnologyCompany/MBHostCloudDADeployNode` | Ramas y PRs |
| Tablero de Trello | Mover tus tarjetas |
| Una cuenta de hosting de prueba en MBHostCloud | Verificar los pasos de la guía |

## 4. Herramientas

- **git** y **GitHub CLI** (`gh`). Acceso al Terminal del panel de una cuenta de prueba para verificar.

## 5. Tu primer issue

Sigue la skill [`trabajar-un-issue`](../.claude/skills/trabajar-un-issue/SKILL.md): elige uno chico, confirma en el issue, rama desde `main`, PR borrador, verifica **en el servidor real** lo que cambiaste, revisa tu diff con `revisar-codigo`, pasa el QA, responde la revisión hasta el squash.

## 6. Cuentas en GitHub

`MarbustTechnologyCompany` crea los issues y aprueba los PRs. Quien implementa (`MarAntBQ` u otro) hace ramas y PRs. Nadie aprueba su propio PR. Merge por squash.

## 7. Lo que nunca se hace

- Afirmar en la guía algo que no se probó en el servidor real.
- Secretos o datos reales de clientes en la guía.
- Promover editar Apache a mano o abrir puertos.
- Push directo a `main`; trabajar sin issue.
- Co-autoría de IA en commits o PRs.

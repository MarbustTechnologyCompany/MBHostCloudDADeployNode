# AGENTS.md — Desplegar una app Node/Next.js en MBHostCloud (guía para agentes de IA)

Esta es la versión **condensada y accionable** para un **agente de IA** que ayuda a un cliente de MBHostCloud a desplegar su app Node.js / Next.js en su hosting (DirectAdmin + Apache), **como el usuario cliente, sin root**. Para el detalle y las explicaciones completas, lee el [`README.md`](./README.md).

> Entorno: hosting compartido DirectAdmin + Apache, acceso por el **Terminal del panel** (sin root). La app corre bajo **pm2** y escucha en un **socket Unix** (no en un puerto). Apache hace reverse-proxy del dominio al socket. El cliente **no** edita configs de Apache; el enlace dominio→socket lo hace **el panel** (sección *Node App*) o el CLI `mbnode-deploy`.

---

## Flujo correcto (en orden)

1. **Subir el código FUERA de `public_html`.** La app va en el HOME, p. ej. `~/miapp`. `public_html` es solo para estáticos públicos. Usar `git clone` o el File Manager.
2. **Elegir la versión de Node según `engines`.** Node 20 LTS, pm2 y nvm ya vienen listos. **Antes de instalar deps**, revisar `engines` en `package.json`. Si pide otra versión (p. ej. `>=22`), instalarla con nvm:
   ```bash
   nvm install 22 && nvm use 22 && nvm alias default 22
   ```
3. **Instalar dependencias** con la versión correcta ya activa:
   ```bash
   cd ~/miapp
   npm ci            # instalación exacta desde el lock (o npm install)
   ```
4. **Adaptar el código para escuchar en el socket** (`process.env.APP_SOCKET`), con fallback a puerto solo para desarrollo local. Para **Next.js SSR** hace falta un `server.js` propio (`next start` NO sirve: solo abre puertos TCP). El `server.js` debe:
   - escuchar en `process.env.APP_SOCKET`,
   - hacer `chmodSync(socket, 0o660)` tras el listen,
   - ejecutar **`process.umask(0o022)` justo DESPUÉS del listen** (imprescindible en Next: sin eso, Next no puede escribir en los directorios que crea — error `EACCES mkdir '.next/cache/images/...'`, imágenes re-optimizadas en cada visita).

   Esqueleto de `server.js` para Next.js:
   ```js
   // server.js — raíz del proyecto, junto a package.json
   const { createServer } = require('http');
   const { existsSync, unlinkSync, chmodSync } = require('fs');
   const next = require('next');

   const socket = process.env.APP_SOCKET;
   const app = next({ dev: false, dir: __dirname });
   const handle = app.getRequestHandler();

   app.prepare().then(() => {
     const server = createServer((req, res) => handle(req, res));
     if (socket) {
       if (existsSync(socket)) { try { unlinkSync(socket); } catch {} }
       server.listen(socket, () => {
         chmodSync(socket, 0o660);
         process.umask(0o022);   // IMPRESCINDIBLE en Next, DESPUÉS del listen
         console.log('escuchando en socket ' + socket);
       });
     } else {
       server.listen(process.env.PORT || 3000);   // fallback solo para dev local
     }
   });
   ```
5. **Build** (si aplica). Next.js y NestJS necesitan compilar:
   ```bash
   npm run build
   ```
6. **Crear el `.env` ANTES de enlazar** (si la app usa variables de entorno). Fuera del repo, nunca en git, con permisos `600`:
   ```bash
   cp .env.example .env     # si hay ejemplo
   chmod 600 .env
   # rellenar valores reales (BD, llaves, etc.)
   ```
7. **Lanzar en pm2**, apuntando al Node correcto:
   ```bash
   pm2 start server.js --name miapp --interpreter="$(which node)"
   pm2 save
   ```
   (Express: `pm2 start app.js ...`. NestJS: `pm2 start dist/main.js ...`.)
8. **Enlazar el dominio al socket** por el panel (**Node App** → elegir app + dominio → **Enlazar**) o el CLI `mbnode-deploy`. **Ese enlace es el que arranca la app en el socket** con `APP_SOCKET` ya puesto. El reverse-proxy + SSL los configura MBHostCloud.
9. **Verificar:** `pm2 logs miapp` debe mostrar "escuchando en socket ..." y las peticiones entrando; `https://tudominio.com` debe servir la app.

---

## Los 3 gotchas que SÍ o SÍ hay que evitar

### 1. `engines` / `EBADENGINE` — revisar la versión de Node ANTES de `npm ci`
Si instalas deps con el Node equivocado (p. ej. Node 20 cuando el proyecto pide `>=22`), `npm ci` suelta avisos `EBADENGINE` y habrá que rehacer la instalación bajo la versión correcta. **Siempre leer `engines` en `package.json` primero** y usar nvm para la versión que pida, antes de instalar.

### 2. `EPERM` antes de enlazar es NORMAL (orden socket ↔ enlace)
El `APP_SOCKET` **solo se asigna AL enlazar**. Por eso **no se puede "escuchar en el socket" antes de enlazar**: antes del enlace la app no tiene socket, cae al fallback de puerto, y la jaula del hosting no permite abrir puertos TCP → aparece:
```
Error: listen EPERM: operation not permitted 0.0.0.0:3000
```
en crash-loop. **Esto es esperado, no es un bug que arreglar.** El orden real:
- **(a)** lanzar la app en pm2 → **da error de puerto, es normal**; con que quede en `pm2 list` basta.
- **(b)** **enlazar** (panel *Node App* o `mbnode-deploy`) → ese paso es el que la arranca en el socket con `APP_SOCKET`.

No intentes "arreglar" el EPERM pre-enlace ni rediseñar el `server.js` por él. Enlaza y el error desaparece.

### 3. El `.env` va FUERA del repo, con `chmod 600`, ANTES de enlazar
Si la app necesita variables (BD, llaves) y arrancas sin `.env`, reventará (p. ej. `DATABASE_URL no está configurada`, 500). Crear el `.env` **antes** de enlazar, **nunca** commitearlo a git, y dejarlo en `chmod 600`. Lo más simple: `cp .env.example .env` y rellenar, o crearlo desde el File Manager del panel.

---

## Notas rápidas para el agente

- **No editar Apache a mano.** El proxy manual (`.htaccess [P]`, Custom HTTPD) está deshabilitado por seguridad en el hosting compartido. El enlace dominio→socket lo hace el panel / `mbnode-deploy`.
- **pm2 y la versión de Node:** si usaste nvm para otra versión, arranca siempre con `--interpreter="$(which node)"`; si el daemon pm2 ya corría con otra versión, `pm2 update`.
- **Arranque en boot:** `pm2 save` lo hace el cliente; el `pm2 startup` (servicio systemd) requiere root → lo corre el admin con la línea `sudo ...` que imprime `pm2 startup`.
- **El socket NO se genera a mano:** la ruta la asigna MBHostCloud al enlazar; el archivo lo crea la app al hacer `listen(APP_SOCKET)`.
- Detalle completo, troubleshooting (502, 500, SSL, umask/EACCES) y ejemplos: **[`README.md`](./README.md)**.

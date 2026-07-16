# MBHostCloudDADeployNode

**Despliega tu propia app Node.js en tu hosting de MBHostCloud — con control total y logs en vivo.**

> ⚠️ **Guía exclusiva para hostings de MBHostCloud®.** Todos los comandos, rutas, versiones y configuraciones de esta guía fueron **verificados sobre la infraestructura de MBHostCloud** (DirectAdmin + Apache). Están pensados **únicamente** para cuentas de hosting de MBHostCloud®; en otros proveedores es muy probable que no apliquen o se comporten distinto. Si tienes tu hosting con nosotros, funcionan tal cual. 💙

Esta guía es para clientes de MBHostCloud (hosting **DirectAdmin + Apache**) que quieren correr su propia aplicación de Node.js (una API con Express/NestJS, un backend, un bot, lo que sea) directamente en su cuenta, sin depender de nadie. Vas a poder subir tu código, elegir la versión de Node que necesites, mantener el proceso vivo aunque el servidor se reinicie, ver los logs en tiempo real, y enlazar tu dominio a la app.

Todo lo que está aquí fue **probado en el servidor real**. Los comandos funcionan tal cual.

---

## Requisitos

- **Acceso SSH por el Terminal del panel DirectAdmin.** No necesitas root ni una llave especial: entra a tu panel → busca **"Terminal"** (o "SSH Terminal") y tendrás una consola dentro de tu cuenta. Todo se hace desde ahí.
- **Saber comandos básicos** de Linux (`cd`, `ls`, `git`, editar un archivo).
- **Tu app vive FUERA de `public_html`.** Regla de oro: `public_html` es para archivos públicos estáticos. Tu app Node va en tu HOME, por ejemplo `~/miapp`. Apache va a reenviar el tráfico a tu app; nadie navega los archivos directamente.

> Node corre como un proceso tuyo que escucha en un **socket** (un archivo dentro de tu carpeta, **sin abrir ningún puerto**). Apache reenvía `tudominio.com` a ese socket. Nunca expongas la carpeta de tu código en la web. *(Cómo hacerlo lo ves más abajo, en "El cambio CLAVE".)*

---

## Paso 1 — Subir tu app (fuera de `public_html`)

Entra al Terminal del panel y ponte en tu HOME. Sube el código con `git clone` (recomendado) o por FTP/File Manager a una carpeta como `~/miapp`.

```bash
cd ~                                  # tu HOME, NO public_html
git clone https://github.com/tu-usuario/tu-repo.git miapp
cd miapp
ls
```

Si prefieres subir por FTP o por el File Manager del panel, crea la carpeta `miapp` **al mismo nivel** que `public_html`, no dentro de ella.

Tu estructura debe verse así:

```
/home/tu-usuario/
├── public_html/      <- archivos públicos (NO tu app)
├── miapp/            <- aquí tu app Node
│   ├── package.json
│   ├── app.js
│   └── ...
└── .bashrc
```

---

## Paso 2 — Node + instalar dependencias

**En MBHostCloud ya tienes Node 20 (LTS) y PM2 listos por defecto — no instalas nada.** Al abrir el Terminal ya están en tu PATH:

```bash
node -v      # v20.18.1   (ya disponible, sin instalar)
npm -v       # 10.8.2
pm2 -v       # 7.x        (ya disponible)
```

Con eso corres la mayoría de apps modernas (Express, NestJS, etc.). Ve directo a **instalar las dependencias de tu proyecto:**

```bash
cd ~/miapp
npm install          # instala según package.json
# o instalación exacta desde el lock:
npm ci
```

> Solo si tu app pide una versión **distinta** de Node (la última, o una vieja como 18/16) la instalas en tu propio HOME sin root, con la sección de abajo. Para la mayoría, **node 20 ya está y no tocas nada.**

### Instalar OTRA versión de Node (opcional): nvm

```bash
# 1) Instalar nvm (se instala en ~/.nvm, todo dentro de tu HOME)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash

# 2) Cargar nvm en la sesión actual (el instalador ya agregó el bloque a ~/.bashrc)
source ~/.bashrc          # o cierra y vuelve a abrir el Terminal

# 3) Instalar y fijar Node 20
nvm install 20
nvm use 20
nvm alias default 20      # que 20 sea el default en cada nuevo login

# 4) Comprobar
node -v                   # v20.x
npm -v
```

El instalador deja este bloque al final de tu `~/.bashrc`, y con eso Node 20 queda cargado **automáticamente** cada vez que entres al Terminal (no tienes que repetir nada):

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

### Alternativa: descargar el tarball oficial (si no quieres nvm o GitHub está bloqueado)

```bash
cd ~
wget https://nodejs.org/dist/v20.18.1/node-v20.18.1-linux-x64.tar.xz
mkdir -p ~/node20
tar -xf node-v20.18.1-linux-x64.tar.xz -C ~/node20 --strip-components=1

# Dejarlo permanente: agrega esta línea al final de ~/.bashrc
echo 'export PATH="$HOME/node20/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

node -v      # v20.18.1
npm -v       # 10.8.2
```

### ¿Y si a tu app le basta con Node 16?

Entonces no instalas nada: el Node del sistema ya está en el PATH.

```bash
node -v      # v16.20.2 (ya disponible)
npm -v       # 8.19.4
```

### Instalar las dependencias del proyecto

Con la versión de Node ya lista:

```bash
cd ~/miapp
npm install          # instala según package.json
# o, para una instalación exacta desde el lock:
npm ci
```

---

## ⭐ El cambio CLAVE en tu app: escuchar en un SOCKET (no en un puerto)

Esta es **la única modificación** que haces en tu código — y es la que evita el mayor dolor de cabeza del hosting compartido: los **choques de puerto**.

**El problema del puerto:** si tu app escucha en un puerto (ej. `3000`), ese puerto es **del servidor entero**. Si otro cliente también usa el 3000, uno de los dos no arranca (`EADDRINUSE`). Y encima queda un puerto abierto en el VPS.

**La solución:** tu app escucha en un **socket Unix** — un archivo dentro de TU carpeta. Como es una ruta tuya, **nunca choca con nadie** y **no abre ningún puerto**. (Es lo mismo que hace CloudLinux por dentro.)

### El cambio (prácticamente una línea)

Donde tu app hace `listen(PUERTO)`, hazla escuchar en el socket que te da MBHostCloud por la variable de entorno `APP_SOCKET`:

**Express / Node `http`:**
```js
const fs = require('fs');
const socket = process.env.APP_SOCKET;   // MBHostCloud te la da, ej: /home/tu-usuario/.sockets/miapp.sock

if (socket) {
  try { fs.unlinkSync(socket); } catch {}      // borra un socket viejo si quedó de un crash
  app.listen(socket, () => {
    fs.chmodSync(socket, 0o660);               // permite que el servidor web lo alcance
    console.log('escuchando en socket ' + socket);
  });
} else {
  app.listen(process.env.PORT || 3000);        // fallback para tu PC de desarrollo
}
```

**NestJS** (`main.ts`):
```ts
import * as fs from 'fs';
// ...
const socket = process.env.APP_SOCKET;
if (socket) {
  try { fs.unlinkSync(socket); } catch {}
  await app.listen(socket);
  fs.chmodSync(socket, 0o660);
} else {
  await app.listen(process.env.PORT ?? 3000);
}
```

> Con `APP_SOCKET` puesto, tu app corre en el **socket** (en el servidor); sin él, en el **puerto** de siempre (en tu PC de desarrollo). **El mismo código sirve para los dos.**

### ¿Tú generas el socket, o el servidor? (la duda más común)

- **La RUTA** (`/home/tu-usuario/.sockets/miapp.sock`) es solo un **nombre de archivo en TU carpeta**. **MBHostCloud te la asigna** al enlazar tu app (así garantizamos que sea única y no choque con nadie). No la inventas tú.
- **El ARCHIVO del socket lo crea TU APP sola** 👉 en el momento en que hace `listen(esa_ruta)`, **Node crea el socket ahí**. **No corres ningún comando para "generar" el socket.**
- **`APP_SOCKET`** es solo esa ruta, puesta como **variable de entorno** cuando se arranca tu app (nosotros te la configuramos al enlazarte).

**En resumen:** tú **no generas nada a mano** — tu app lee `APP_SOCKET` y crea el socket al arrancar. Nosotros te damos la ruta y enganchamos tu dominio a ella. Tú solo escuchas en él y ves tus logs. ✅

---

## Paso 3 — Correr con PM2 (¡y ver los logs EN VIVO!)

Node por sí solo se muere cuando cierras el Terminal. Para mantener tu app corriendo usamos **PM2**, un gestor de procesos. Es la misma herramienta que usa producción — y **ya viene instalado** en tu hosting, no corres `npm install -g pm2`:

```bash
pm2 -v       # 7.x  (ya disponible)
```

### ¿Qué archivo arranca PM2? — Express vs NestJS

PM2 arranca **el archivo que ejecuta tu app**. La diferencia entre frameworks es si ese archivo es tu **código directo** o uno **compilado**:

**Express (u otro Node/JS "plano") — sin build, arrancas tu fuente:**
```bash
cd ~/miapp
npm install
pm2 start app.js --name miapp        # <- tu archivo tal cual (app.js / server.js / index.js)
pm2 save
```
Si tu app arranca por un script de npm (`"start": "node server.js"`):
```bash
pm2 start npm --name miapp -- start
```

**NestJS (TypeScript) — hay que compilar a `dist/` y arrancar `dist/main.js`:**
```bash
cd ~/miapp
npm install
npm run build                        # compila TS -> genera dist/
pm2 start dist/main.js --name miapp  # <- arrancas lo COMPILADO, no el src/
pm2 save
```
⚠️ **En NestJS, cada vez que cambies el código tienes que RE-compilar** antes de reiniciar:
```bash
git pull            # o tus cambios
npm install         # solo si cambiaron dependencias
npm run build       # imprescindible en NestJS (recompila dist/)
pm2 restart miapp
```

> **Regla simple:** Express arranca tu **fuente** (`app.js`); NestJS (y todo TypeScript compilado) arranca lo de **`dist/`** (`dist/main.js`) y necesita `npm run build` en cada cambio. Otros: **Next.js** → `pm2 start npm --name miapp -- start` (tras `npm run build`). **Frontend estático** (Vite/React puro) NO va con PM2 — eso va en `public_html`.

### Arranca tu app y mírala corriendo

```bash
cd ~/miapp
pm2 start app.js --name miapp          # arranca tu app con el nombre "miapp"
pm2 list                               # tabla: id, nombre, estado, pid, uptime, cpu, mem
```

Si tu app arranca por un script de npm:

```bash
pm2 start "npm run start:prod" --name miapp
```

`pm2 list` te muestra algo así:

```
│ id │ name  │ mode │ pid    │ uptime │ ↺ │ status │ cpu │ mem    │
│ 0  │ miapp │ fork │ 716053 │ 3s     │ 0 │ online │ 0%  │ 50.0mb │
```

### Los logs en vivo — lo mejor de todo

Esto es lo que da control real: ver lo que tu app escribe **en el momento en que pasa**.

```bash
pm2 logs miapp
```

La consola se queda "pegada" mostrando cada línea nueva conforme sale:

```
[TAILING] Tailing last lines for [miapp] process
0|miapp | miapp arriba en node v20.18.1
0|miapp | tick 16 node v20.18.1
0|miapp | tick 17 node v20.18.1
```

Sales con **Ctrl-C** — y ojo: eso **solo deja de mirar los logs, NO detiene tu app**. Tu app sigue corriendo tan feliz.

Más formas de ver logs:

```bash
pm2 logs                          # logs EN VIVO de TODAS tus apps a la vez
pm2 logs miapp --lines 200        # muestra 200 líneas de historial y sigue en vivo
pm2 logs miapp --err              # SOLO errores (stderr), en vivo
pm2 logs miapp --nostream --lines 50   # imprime las últimas 50 y sale (no se queda pegado)
pm2 flush miapp                   # vacía los archivos de log de esa app
```

**Dashboard interactivo** (CPU, memoria y logs en una sola pantalla, se sale con `q`):

```bash
pm2 monit
```

Los archivos físicos de log están en:

```
~/.pm2/logs/miapp-out.log     (stdout — lo normal)
~/.pm2/logs/miapp-error.log   (stderr — errores)
```

> Tip: para que los logs no crezcan sin fin, puedes instalar la rotación automática: `pm2 install pm2-logrotate`.

### Controlar la app

```bash
pm2 restart miapp                # reinicio duro (úsalo tras cada deploy)
pm2 reload miapp                 # recarga suave
pm2 stop miapp                   # detiene pero lo deja en la lista
pm2 delete miapp                 # lo quita de PM2 por completo
pm2 restart miapp --update-env   # reinicia releyendo las variables de entorno
```

---

## Paso 4 — Que sobreviva a los reinicios del servidor

Si el servidor se reinicia, quieres que tu app vuelva sola. Esto son **dos piezas**:

**A) Un servicio de arranque (lo activa una sola vez el administrador).**
Como cliente no eres root, así que PM2 no puede instalar el servicio de arranque por ti — pero **te imprime la línea exacta** que el admin debe correr. Ejecuta:

```bash
pm2 startup
```

Verás algo así (el texto exacto depende de tu usuario y de dónde está tu Node):

```
[PM2] Init System found: systemd
[PM2] To setup the Startup Script, copy/paste the following command:
sudo env PATH=$PATH:... pm2 startup systemd -u TU_USUARIO --hp /home/TU_USUARIO
```

**Copia esa línea `sudo ...` y pásasela al administrador de MBHostCloud** (soporte). Él la corre una sola vez y queda listo para siempre.

**B) Guardar tu lista de apps (esto lo haces tú, sin root).**
Con tus apps corriendo como las quieres:

```bash
pm2 save
```

Esto congela la lista actual (`Successfully saved in ~/.pm2/dump.pm2`). En el próximo arranque del servidor, el sistema hará que PM2 resucite exactamente esas apps. **Verificado: la app vuelve sola tras el reinicio.**

Repite `pm2 save` cada vez que agregues, quites o cambies tus apps.

> **Cuidado:** para persistir usa siempre `pm2 save`. No intentes reiniciar el servicio con `systemctl restart` mientras tu daemon de PM2 está vivo — puede entrar en un bucle de error. En un reinicio real del servidor no pasa. Si alguna vez necesitas forzarlo a mano, primero `pm2 kill` y luego que el admin lo arranque.

---

## Paso 5 — Enlazar tu dominio/subdominio a tu app

Falta lo último: que cuando alguien entre a `app.tudominio.com`, el servidor reenvíe (reverse-proxy) el tráfico a tu app Node. **Esto NO lo configuras a mano** — en MBHostCloud el enlace se hace desde el **panel**, y nosotros apuntamos tu dominio a tu **socket** de forma segura.

> ⚠️ En el hosting compartido, "Custom HTTPD Configurations" y el `.htaccess` con proxy (`[P]`) están **deshabilitados a propósito, por seguridad** — así ningún cliente puede tocar (ni espiar) la configuración de otro. Por eso el enlace se hace por el panel o nos lo pides; **no** editando configs de Apache a mano.

### Cómo enlazar

1. Crea tu **dominio o subdominio** en DirectAdmin (**Account Manager → Domain Setup**), si aún no existe.
2. Asegúrate de que tu app **escuche en el socket** (ver la sección *"El cambio CLAVE: escuchar en un SOCKET"*), no en un puerto.
3. Arranca tu app en pm2 (Paso 3) **escuchando en el socket**. Luego, en tu **panel de MBHostCloud**, abre la sección **"Node App"**, elige tu **app** (de las que tengas corriendo en pm2) y tu **dominio/subdominio** → botón **Enlazar**.
   > *La **carpeta** y el **archivo de arranque** se detectan solos de tu pm2 — no los escribes.* El enlace es **casi instantáneo**.
   > *(Mientras habilitamos esa sección en tu panel, escríbenos tu dominio + carpeta de la app a **support@mbhostcloud.com** y lo activamos en el momento.)*

En segundos tu dominio queda sirviendo tu app **por socket** (sin puerto abierto), con **SSL**. Nosotros configuramos el reverse-proxy y tu `APP_SOCKET`; **tú solo mantienes tu app viva con pm2 y ves tus logs.** 🎉

> **¿WebSockets?** (Socket.io, etc.) — funcionan; avísanos al enlazar y activamos el soporte `wss://`.

---

## Ejemplo completo (listo para copiar)

Un mini `app.js` para probar todo el flujo de punta a punta (escuchando en **socket**, como es en MBHostCloud):

```js
// ~/miapp/app.js
const http = require('http');
const fs = require('fs');
const socket = process.env.APP_SOCKET;   // MBHostCloud te la asigna al enlazar

const server = http.createServer((req, res) => {
  console.log(new Date().toISOString(), req.method, req.url);
  res.writeHead(200, { 'Content-Type': 'text/plain; charset=utf-8' });
  res.end(`Hola desde mi app Node en ${process.version}\nRuta: ${req.url}\n`);
});

if (socket) {
  try { fs.unlinkSync(socket); } catch {}
  server.listen(socket, () => { fs.chmodSync(socket, 0o660); console.log('miapp en socket ' + socket); });
} else {
  server.listen(process.env.PORT || 3000, () => console.log('miapp en puerto (modo desarrollo)'));
}
```

Arrancarlo y verlo:

```bash
cd ~/miapp
pm2 start app.js --name miapp
pm2 logs miapp
```

El **enlace del dominio → tu app lo hace MBHostCloud por el panel** (sección **"Node App"**: eliges dominio + carpeta + arranque → *Enlazar*), o nos lo pides a **support@mbhostcloud.com**. **No editas Apache a mano** — en el hosting compartido el proxy manual está deshabilitado por seguridad; nosotros apuntamos tu dominio a tu **socket** con SSL.

Y si prefieres arrancar con un archivo de configuración de PM2 (`ecosystem.config.js`), en vez de la línea de comando:

```js
// ~/miapp/ecosystem.config.js
module.exports = {
  apps: [
    {
      name: 'miapp',
      script: 'app.js',        // o: 'npm', args: 'run start:prod'
      cwd: '/home/TU_USUARIO/miapp',
      env: {
        NODE_ENV: 'production',
        PORT: 3014,
      },
    },
  ],
};
```

```bash
pm2 start ecosystem.config.js
pm2 save
```

Abre `https://app.tudominio.com` en tu navegador: deberías ver el mensaje de tu app, y en `pm2 logs miapp` verás la petición aparecer en vivo. 🎉

---

## Tips y solución de problemas

**Después de cada deploy (git pull / nuevos cambios):**
```bash
cd ~/miapp
git pull
npm install          # solo si cambiaron dependencias
pm2 restart miapp
pm2 logs miapp       # confirma que arrancó sin errores
```

**Ver rápido si tu app está viva:**
```bash
pm2 list             # ¿status "online"? bien. ¿"errored" o "stopped"? revisa logs.
pm2 logs miapp --err --lines 50 --nostream
```

**"Puerto ocupado" (`EADDRINUSE`):** ya hay algo escuchando en ese puerto. Suele ser una instancia vieja de tu app. Revisa y limpia:
```bash
pm2 list
pm2 delete miapp     # borra la instancia anterior
pm2 start app.js --name miapp
```
O elige otro puerto en tu app **y** actualiza el puerto en el bloque del proxy.

**Error 502 (Bad Gateway):** Apache no encuentra tu app. Casi siempre es una de estas:
- Tu app no está corriendo → `pm2 list` (¿está "online"?).
- El puerto del proxy no coincide con el `listen` de tu app → deben ser el mismo número.
- Tu app no escucha en `127.0.0.1` → usa `app.listen(PUERTO, '127.0.0.1')`.

**Error 500 / la página no carga bien:** el problema está dentro de tu app, no en el proxy. Míralo en vivo:
```bash
pm2 logs miapp --err
```

**El SSL no renueva:** verifica que dejaste la línea `RewriteCond %{REQUEST_URI} !^/\.well-known/` en tu bloque de proxy — sin ella, Let's Encrypt no puede validar el dominio.

**`node -v` sigue mostrando v16 tras instalar Node 20:** no se cargó tu `~/.bashrc`. Corre `source ~/.bashrc` o cierra y reabre el Terminal. Con nvm, asegúrate de haber hecho `nvm alias default 20`.

---

## En resumen (copy-paste)

```bash
# 1) Subir tu app (fuera de public_html)
cd ~ && git clone TU_REPO miapp && cd miapp

# 2) Node 20 + pm2 YA vienen listos (no instalas nada). Solo tus dependencias:
node -v                  # v20.18.1  (ya disponible)
npm install              # (en NestJS, además:  npm run build)

# 3) Correr y ver logs en vivo
pm2 start app.js --name miapp                 # Express: tu app.js
# NestJS:   npm run build && pm2 start dist/main.js --name miapp
pm2 logs miapp           # <-- LOGS EN VIVO (Ctrl-C para salir, la app sigue)

# 4) Que sobreviva reinicios
pm2 save                 # congela tu lista de apps (vuelven solas tras un reinicio)

# 5) Enlazar el dominio: por el panel (sección "Node App"), o pídelo a support@mbhostcloud.com
```

¡Listo! Tu app corre bajo tu control, sobrevive reinicios, y puedes ver todo lo que hace en tiempo real. Bienvenido al control total. 🚀

---

## ¿Necesitas ayuda?

Esta guía es **exclusiva para clientes de MBHostCloud®**. Para el único paso que requiere administrador (el `pm2 startup` que activa el arranque en boot) o para cualquier duda del despliegue, escríbenos a **support@mbhostcloud.com** y te ayudamos.

<sub>MBHostCloud® — un servicio de MarBust Technology Company. Hosting con control real y logs en vivo.</sub>
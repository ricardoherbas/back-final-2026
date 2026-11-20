# back-final-2026

API REST del trabajo final 2026. Está hecha con Node.js, Express 5, TypeScript, Sequelize y PostgreSQL.

Esta guía permite levantar el backend desde cero. Hay dos formas:

- **Opción A: Docker** (recomendada). Levanta la API y la base de datos juntas.
- **Opción B: modo desarrollo**. Corre la API con Node.js en tu máquina y necesita una base PostgreSQL.

Las dos opciones empiezan con los pasos 1 y 2.

---

## 1. Requisitos

| Herramienta | Opción A (Docker) | Opción B (desarrollo) |
|---|---|---|
| Git | Sí | Sí |
| Docker + Docker Compose v2 | Sí | Solo para la base de datos (ver 4.B.2) |
| Node.js 20 o superior y npm | No | Sí |
| PostgreSQL 17 | No | Solo si no usás Docker para la base |

Una cuenta de correo con acceso SMTP es necesaria en ambos casos (ver paso 2).

## 2. Clonar y configurar las variables de entorno

```bash
git clone https://github.com/ricardoherbas/backend-f-2026.git back-final-2026
cd back-final-2026
cp .env.example .env
```

Abrí `.env` y completá **todas** las variables de la tabla.

| Variable | Obligatoria | Descripción |
|---|---|---|
| `PORT` | No | Puerto de la API. Por defecto `3000`. |
| `DB_HOST` | Sí | Host de PostgreSQL. **Con Docker se ignora** (compose usa el servicio `db`). En modo desarrollo usá `localhost`. |
| `DB_PORT` | Sí | Puerto de PostgreSQL. Normalmente `5432`. |
| `DB_NAME` | Sí | Nombre de la base de datos. Con Docker se crea con este nombre. |
| `DB_USER` | Sí | Usuario de la base. Con Docker se crea con este nombre. |
| `DB_PASSWORD` | Sí | Contraseña de la base. Con Docker se crea con esta contraseña. |
| `JWT_SECRET` | Sí | Secreto para firmar los tokens de sesión. Usá una cadena larga y aleatoria. |
| `ADMIN_NOMBRE` | Sí | Nombre del administrador inicial. |
| `ADMIN_APELLIDO` | Sí | Apellido del administrador inicial. |
| `ADMIN_DNI` | Sí | DNI del administrador inicial. |
| `ADMIN_FECHA_NACIMIENTO` | No | Formato `AAAA-MM-DD`. |
| `ADMIN_TELEFONO` | No | Teléfono del administrador. |
| `ADMIN_EMAIL` | Sí | **Email con el que el administrador inicia sesión.** |
| `ADMIN_PASSWORD` | Sí | **Contraseña con la que el administrador inicia sesión.** |
| `SMTP_HOST` | Sí | Servidor SMTP (se usa para el correo de recuperación de contraseña). |
| `SMTP_PORT` | Sí | Puerto SMTP. `587` por defecto; con `465` se usa conexión segura. |
| `SMTP_USER` | Sí | Usuario SMTP. También es el remitente de los correos. |
| `SMTP_PASSWORD` | Sí | Contraseña SMTP. |
| `FRONTEND_URL` | Sí | URL donde corre el frontend, por ejemplo `http://localhost:5173`. Se usa para armar el enlace de recuperación de contraseña. |

> **Importante: las variables SMTP no se pueden dejar vacías.** El backend se detiene al arrancar con el error `Faltan variables de configuración SMTP`.
> Si solo querés probar el sistema sin enviar correos, poné valores cualquiera (por ejemplo `SMTP_HOST=localhost`, `SMTP_USER=test`, `SMTP_PASSWORD=test`). En ese caso la recuperación de contraseña no va a enviar correos.

Ejemplo mínimo para una prueba local (cambiá las contraseñas y el secreto):

```env
PORT=3000

DB_HOST=localhost
DB_PORT=5432
DB_NAME=back_final
DB_USER=postgres
DB_PASSWORD=cambiar_esta_clave

JWT_SECRET=cambiar_por_una_cadena_larga_y_aleatoria

ADMIN_NOMBRE=Admin
ADMIN_APELLIDO=Principal
ADMIN_DNI=12345678
ADMIN_FECHA_NACIMIENTO=1990-01-01
ADMIN_TELEFONO=1112345678
ADMIN_EMAIL=admin@ejemplo.com
ADMIN_PASSWORD=CambiarEstaClave123

SMTP_HOST=smtp.ejemplo.com
SMTP_PORT=587
SMTP_USER=usuario@ejemplo.com
SMTP_PASSWORD=clave_smtp

FRONTEND_URL=http://localhost:5173
```

El archivo `.env` no se sube a Git. No lo compartas.

---

## 3. Credenciales del administrador inicial

El administrador inicial **se crea a partir de las variables `ADMIN_*` del `.env`**, ejecutando el comando de seed (paso 4).
Con las variables del ejemplo, las credenciales de acceso serían:

| Campo | Valor |
|---|---|
| Email | el valor de `ADMIN_EMAIL` |
| Contraseña | el valor de `ADMIN_PASSWORD` |
| Rol | `administrador` |

Notas:

- **El seed no corre solo al levantar Docker ni al ejecutar `npm start`.** Tenés que ejecutarlo una vez (ver paso 4).
- Se puede ejecutar más de una vez sin problema. Si ya existe una persona con ese DNI, un usuario con ese email o un administrador asociado, no los duplica.
- Si el administrador ya existe, el seed **no cambia su contraseña** aunque modifiques `ADMIN_PASSWORD` en el `.env`.
- Si falta `ADMIN_NOMBRE`, `ADMIN_APELLIDO`, `ADMIN_DNI`, `ADMIN_EMAIL` o `ADMIN_PASSWORD`, el seed falla con el mensaje `Faltan datos obligatorios del administrador en el .env`.
- Cambiá la contraseña por una segura antes de usar el sistema fuera de un entorno de prueba.

---

## 4. Levantar el proyecto

### Opción A: con Docker (recomendada)

`compose.yaml` levanta dos servicios: la API (`back_final_app`) y PostgreSQL 17 (`back_final_db`). Las tablas se crean solas desde `database/init.sql` la primera vez que se crea la base.

**4.A.1. Construir y levantar**

```bash
docker compose up --build -d
```

**4.A.2. Verificar que arrancó**

```bash
docker compose ps
docker compose logs -f backend
```

En los logs tiene que aparecer:

```
Conexión a PostgreSQL establecida
La API está escuchando en el puerto: 3000
```

(Salí de los logs con `Ctrl + C`; los contenedores siguen corriendo.)

**4.A.3. Crear el administrador inicial**

```bash
docker compose exec backend npm run seed
```

Tiene que terminar con `Seed ejecutado correctamente`.

La API queda disponible en `http://localhost:3000` (o en el `PORT` que hayas definido).

**Comandos útiles**

```bash
docker compose down        # detiene y elimina los contenedores (conserva los datos)
docker compose down -v     # además BORRA la base de datos (volumen postgres_data)
docker compose logs -f     # ver logs
```

Notas de Docker:

- Los datos de la base se guardan en el volumen `postgres_data`. El script `init.sql` solo se ejecuta cuando ese volumen está vacío. Si cambiás `DB_NAME`, `DB_USER` o `DB_PASSWORD` después de la primera vez, ejecutá `docker compose down -v` para recrear la base.
- PostgreSQL se publica en el puerto `5432` de tu máquina. Si ya lo tenés ocupado, agregá `DB_EXTERNAL_PORT=5433` (u otro puerto libre) al `.env`.
- Si cambiás `PORT`, la API se publica en ese puerto de tu máquina.

### Opción B: modo desarrollo (sin Docker para la API)

**4.B.1. Instalar dependencias**

```bash
npm install
```

**4.B.2. Tener una base PostgreSQL con las tablas creadas**

La API no crea las tablas por sí misma. Elegí una de estas dos formas.

*Forma 1: usar solo el servicio de base de datos de Docker (la más simple).*

```bash
docker compose up -d db
```

Esto crea la base con `DB_NAME`, `DB_USER` y `DB_PASSWORD` del `.env` y carga `database/init.sql`. En el `.env` dejá `DB_HOST=localhost` y `DB_PORT=5432` (o el valor de `DB_EXTERNAL_PORT`, si lo definiste).

*Forma 2: usar un PostgreSQL propio.* Creá la base y cargá el esquema:

```bash
createdb -U <DB_USER> <DB_NAME>
psql -U <DB_USER> -d <DB_NAME> -f database/init.sql
```

Después ajustá `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER` y `DB_PASSWORD` en el `.env`.

**4.B.3. Crear el administrador inicial**

```bash
npm run seed
```

**4.B.4. Iniciar la API en modo desarrollo**

```bash
npm run dev
```

Se reinicia sola al guardar cambios. En consola tiene que aparecer `La API está escuchando en el puerto: 3000`.

**Modo producción sin Docker (opcional)**

```bash
npm run build
npm start
```

---

## 5. Probar que funciona

Iniciá sesión con el administrador creado en el paso 3 (cambiá email y contraseña por los de tu `.env`):

```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@ejemplo.com","password":"CambiarEstaClave123"}'
```

Si todo está bien, la respuesta incluye un token de sesión.

Los endpoints cuelgan de `/api` (`/api/auth`, `/api/usuarios`, `/api/pacientes`, `/api/turnos`, etc.).

La URL base del backend (`http://localhost:3000` o la que corresponda) es la que hay que configurar en `VITE_API_URL` del repositorio del frontend.

---

## 6. Problemas frecuentes

| Síntoma | Causa y solución |
|---|---|
| `Faltan variables de configuración SMTP` al arrancar | Alguna variable `SMTP_*` está vacía. Completala (ver paso 2). |
| `Faltan datos obligatorios del administrador en el .env` | Completá las variables `ADMIN_*` obligatorias y volvé a ejecutar el seed. |
| `PostgreSQL todavía no está disponible. Intento n/10` y luego se detiene | La base no está levantada o `DB_HOST`/`DB_PORT`/credenciales son incorrectos. En modo desarrollo, `DB_HOST` debe ser `localhost`. |
| `relation "personas" does not exist` | La base no tiene las tablas. Cargá `database/init.sql` (ver 4.B.2) o recreá el volumen con `docker compose down -v`. |
| `port is already allocated` | El puerto `3000` o `5432` está ocupado. Cambiá `PORT` o `DB_EXTERNAL_PORT` en el `.env`. |
| `401 Email o contraseña incorrectos` al iniciar sesión | No se ejecutó el seed, o el email/contraseña no coinciden con `ADMIN_EMAIL`/`ADMIN_PASSWORD`. |
| Cambié `ADMIN_PASSWORD` y la contraseña vieja sigue funcionando | El seed no actualiza administradores existentes (ver paso 3). |

## 7. Otros comandos

```bash
npm run lint    # revisa el código con ESLint
npm run build   # compila TypeScript a dist/
```
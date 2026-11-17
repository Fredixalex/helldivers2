# helldivers2
# Guía del examen de Sistemas Operativos

Conectarse por SSH, clonar SIMEF, conectarlo a MariaDB y levantarlo solo con comandos de Django (sin Apache ni Gunicorn). MariaDB ya está instalada por el profe.

## 1. Conectarte por SSH

Desde la terminal de tu compu, con la IP y la contraseña que te da el profe:

```bash
ssh maquina1@IP_ASIGNADA
```

> **Te pide datos:** escribí `yes` si pregunta si confiás en el equipo, y después la contraseña (no se ve lo que escribís).

## 2. Clonar el repositorio

```bash
sudo apt update && sudo apt install git -y
git clone https://github.com/ISFDYT-210/simef.git
cd simef
```

> **Puede pedirte:** la contraseña de `maquina1` (por el `sudo`). Si pregunta `[Y/n]`, escribí `Y`.

Si el repo pide usuario y contraseña, usá tu usuario de GitHub y un token en vez de la contraseña.

## 3. Crear la base y el usuario en MariaDB

MariaDB ya está instalada, solo hay que crear la base. Si el profe ya te la dejó, saltá este paso.

```bash
sudo mariadb
```

> **Se abre otra pantalla:** el prompt cambia a `MariaDB [(none)]>`. Ahí pegás las 5 líneas de abajo, con Enter. Cuando termina `EXIT;` volvés a la terminal normal. Acá no se guarda nada: se ejecuta al apretar Enter.

```sql
CREATE DATABASE simef CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'usuario_django'@'localhost' IDENTIFIED BY 'tu_password';
GRANT ALL PRIVILEGES ON simef.* TO 'usuario_django'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 4. Entorno virtual y dependencias

Parate en la carpeta donde está `manage.py` (después del `cd simef` ya estás ahí; comprobalo con `ls`):

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pip install pymysql
```

Sabés que el entorno está activo porque aparece `(venv)` al principio de la línea. `pymysql` es el conector de MariaDB; el `requirements.txt` solo trae el de PostgreSQL.

## 5. Crear el archivo `.env`

```bash
nano .env
```

> **Se abre un editor (nano):** la terminal se convierte en un editor de texto vacío. Pegá o escribí las 3 líneas de abajo. Para **guardar**: `Ctrl+O` y `Enter`. Para **salir**: `Ctrl+X`. Después volvés a la terminal normal.

```
SECRET_KEY=cualquiera
DEBUG=True
ALLOWED_HOSTS=*
```

Sin `DEBUG=True`, SIMEF se niega a arrancar.

## 6. Conectar el sistema a MariaDB

```bash
nano gestionInstituto/settings.py
```

> **Se abre el editor (nano) con el archivo ya escrito:** buscá el bloque `DATABASES` con `Ctrl+W` (escribís `DATABASES` y Enter). Borrás las líneas de ese bloque y pegás el nuevo. Para **guardar**: `Ctrl+O` y `Enter`. Para **salir**: `Ctrl+X`.

Reemplazalo por este (los datos tienen que coincidir con los del paso 3):

```python
import pymysql
pymysql.install_as_MySQLdb()

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'simef',
        'USER': 'usuario_django',
        'PASSWORD': 'tu_password',
        'HOST': 'localhost',
        'PORT': '3306',
    }
}
```

## 7. Crear las tablas (migrar)

```bash
python3 manage.py migrate
```

## 8. Crear un usuario para entrar

La base de MariaDB está vacía, así que hay que crear un usuario. En SIMEF el login es **por DNI**, no por nombre de usuario. Hacelo antes de levantar el servidor (o en una segunda terminal por SSH, con el venv activado).

```bash
python3 manage.py createsuperuser
```

> **Te pide datos:** el DNI (cualquier número, por ejemplo `12345678`) y la contraseña dos veces. No se ve lo que escribís. Si avisa que la contraseña es muy simple, escribí `y` y Enter.

## 9. Levantar el servidor

```bash
python3 manage.py runserver 0.0.0.0:8000
```

No cierres esta terminal mientras uses el sistema. Si necesitás otra, abrí una segunda y volvé a conectarte por SSH.

## 10. Entrar desde el navegador

```
http://IP_ASIGNADA:8000
```

Entrá con el DNI y la contraseña del paso 8. Si la pantalla muestra poco, probá `/admin/` con el mismo DNI.

## Si algo falla

- **No arranca y pide una variable:** falta el `.env` o el `DEBUG=True`.
- **Migra pero no aparecen tablas en MariaDB:** no cambiaste el bloque `DATABASES` y está usando SQLite.
- **Error de acceso a la base:** usuario, contraseña o nombre de base distintos a los del paso 3.
- **No abre en el navegador:** revisá que pusiste `0.0.0.0:8000` y que la IP es la correcta.

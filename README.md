# helldivers2
Guía del examen de Sistemas Operativos
Conectarse por SSH, clonar SIMEF, conectarlo a MariaDB y levantarlo solo con comandos de Django
(sin Apache ni Gunicorn). MariaDB ya está instalada por el profe.
1. Conectarte por SSH
Desde la terminal de tu compu, con la IP y la contraseña que te da el profe:
ssh maquina1@IP_ASIGNADA
TE PIDE DATOS: escribí yes si pregunta si confiás en el equipo, y después la contraseña.
Si pregunta "Are you sure you want to continue connecting?", escribí yes. La contraseña no
muestra nada al escribirla.
2. Clonar el repositorio
sudo apt update && sudo apt install git -y
git clone https://github.com/ISFDYT-210/simef.git
cd simef
PUEDE PEDIRTE: la contraseña de maquina1 (por el sudo). Si pregunta [Y/n], escribí Y.
Si el repo pide usuario y contraseña, usá tu usuario de GitHub y un token en vez de la
contraseña.
3. Crear la base y el usuario en MariaDB
MariaDB ya está instalada, solo hay que crear la base. Si el profe ya te la dejó, saltá este paso.
sudo mariadb
SE ABRE OTRA PANTALLA: el prompt cambia a MariaDB [(none)]>. Ahí pegás las 5 líneas de
abajo (de a una o todas juntas, con Enter). Cuando termina EXIT; volvés a la terminal normal. Acá no
se guarda nada: se ejecuta al apretar Enter.
CREATE DATABASE simef CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'usuario_django'@'localhost' IDENTIFIED BY 'tu_password';
GRANT ALL PRIVILEGES ON simef.* TO 'usuario_django'@'localhost';
FLUSH PRIVILEGES;
EXIT;
4. Entorno virtual y dependencias
Parate en la carpeta donde está manage.py:
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pip install pymysql
Sabés que el entorno está activo porque aparece (venv) al principio de la línea. pymysql es el
conector de MariaDB; el requirements solo trae el de PostgreSQL.
5. Crear el archivo .env
nano .env
SE ABRE UN EDITOR (nano): la terminal se convierte en un editor de texto vacío. Pegá o escribí
las 3 líneas de abajo. Para GUARDAR: Ctrl+O y Enter. Para SALIR: Ctrl+X. Después volvés a la
terminal normal.
Examen SO · Página 1
SECRET_KEY=cualquiera
DEBUG=True
ALLOWED_HOSTS=*
Sin DEBUG=True, SIMEF se niega a arrancar.
6. Conectar el sistema a MariaDB
nano gestionInstituto/settings.py
SE ABRE EL EDITOR (nano) con el archivo ya escrito: buscá el bloque DATABASES con Ctrl+W
(escribís DATABASES y Enter). Borrás las líneas de ese bloque y pegás el nuevo. Para GUARDAR:
Ctrl+O y Enter. Para SALIR: Ctrl+X.
Buscá el bloque DATABASES y reemplazalo por este (los datos tienen que coincidir con los
del paso 3):
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
7. Crear las tablas (migrar)
python3 manage.py migrate
8. Crear un usuario para entrar
La base de MariaDB está vacía, así que hay que crear un usuario. En SIMEF el login es por
DNI, no por nombre de usuario. Hacelo antes de levantar el servidor (o en una segunda
terminal por SSH, con el venv activado).
python3 manage.py createsuperuser
TE PIDE DATOS: el DNI (cualquier número, por ejemplo 12345678) y la contraseña dos veces. No
se ve lo que escribís. Si avisa que la contraseña es muy simple, escribí y y Enter.
9. Levantar el servidor
python3 manage.py runserver 0.0.0.0:8000
No cierres esta terminal mientras uses el sistema. Si necesitás otra, abrí una segunda y volvé
a conectarte por SSH.
10. Entrar desde el navegador
http://IP_ASIGNADA:8000
Entrá con el DNI y la contraseña del paso 8. Si la pantalla muestra poco, probá /admin/ con el
mismo DNI.
Examen SO · Página 2
Si algo falla
No arranca y pide una variable: falta el .env o el DEBUG=True.
Migra pero no aparecen tablas en MariaDB: no cambiaste el bloque DATABASES y está
usando SQLite.
Error de acceso a la base: usuario, contraseña o nombre de base distintos a los del paso 3.
No abre en el navegador: revisá que pusiste 0.0.0.0:8000 y que la IP es la correcta.
Examen SO · Página 3

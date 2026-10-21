# Laravel 13 + Livewire 4 — Ubuntu 26.04 LTS

Proyecto Laravel alojado en GitHub, preparado para funcionar con Nginx, PHP 8.5, MariaDB, Composer, Node.js y Vite.

## 1. Información general

| Componente | Configuración utilizada |
|---|---|
| Sistema operativo | Ubuntu 26.04 LTS |
| Servidor web | Nginx 1.28 |
| PHP | PHP 8.5 + PHP-FPM |
| Base de datos | MariaDB 12.3 |
| Framework | Laravel 13 |
| Componentes interactivos | Livewire 4 |
| Plantillas | Blade |
| Gestor de dependencias PHP | Composer 2 |
| JavaScript | Node.js 24 LTS |
| Compilación de recursos | Vite 8 |
| Control de versiones | Git + GitHub |
| Directorio del proyecto | `/var/www/miapp` |
| Usuario de despliegue | `deploy` |
| Base de datos | `laravel_db` |
| Usuario MariaDB | `laravel_user` |
| Zona horaria | `America/La_Paz` |

**Repositorio:** https://github.com/gundball/laravel

El proyecto se mantiene en la rama `main`.

---

## 2. Objetivo

Este manual permite:

1. Preparar un nuevo servidor Ubuntu 26.04.
2. Instalar el entorno necesario para Laravel.
3. Descargar el proyecto desde GitHub.
4. Reconstruir las dependencias PHP y JavaScript.
5. Configurar MariaDB y Laravel.
6. Restaurar una base de datos mediante un archivo SQL.
7. Configurar Nginx y PHP-FPM.
8. Establecer permisos de archivos.
9. Publicar y actualizar la aplicación.

Los comandos de administración del servidor se ejecutan como `root`, salvo que se indique lo contrario.

**No se deben copiar literalmente las contraseñas de ejemplo.**

---

## 3. Actualizar Ubuntu

Entrar al servidor mediante SSH o la consola de Proxmox.

Actualizar los paquetes:

```bash
apt update
apt upgrade -y
```

Instalar herramientas generales:

```bash
apt install -y curl wget git unzip zip ca-certificates gnupg acl
```

Comprobar la versión del sistema:

```bash
cat /etc/os-release
```

Este proyecto se preparó inicialmente sobre Ubuntu 26.04 LTS.

---

## 4. Configurar la zona horaria

Establecer la zona horaria utilizada por el proyecto:

```bash
timedatectl set-timezone America/La_Paz
```

Comprobar:

```bash
timedatectl
```

La configuración de Laravel también debe utilizar esta zona horaria en `config/app.php`:

```php
'timezone' => 'America/La_Paz',
```

La zona horaria del sistema y la de Laravel pueden configurarse independientemente.

---

## 5. Crear el usuario de despliegue

El proyecto será administrado mediante el usuario `deploy`, evitando ejecutar Composer, npm y Git como `root`.

Crear el usuario si todavía no existe:

```bash
adduser deploy
```

Crear el directorio de aplicaciones:

```bash
mkdir -p /var/www
```

El proyecto terminará ubicado en:

```text
/var/www/miapp
```

---

## 6. Instalar Nginx

Instalar Nginx:

```bash
apt install -y nginx
```

Iniciar y habilitar el servicio:

```bash
systemctl enable --now nginx
```

Comprobar su estado:

```bash
systemctl status nginx
```

El directorio público de Laravel será:

```text
/var/www/miapp/public
```

**Nunca debe configurarse Nginx para servir directamente `/var/www/miapp`**, porque allí existen archivos privados como `.env`.

---

## 7. Instalar PHP 8.5 y sus extensiones

Instalar PHP-FPM, PHP para consola y las extensiones necesarias:

```bash
apt install -y php8.5-cli php8.5-fpm php8.5-mysql php8.5-mbstring php8.5-xml php8.5-curl php8.5-zip php8.5-bcmath php8.5-intl php8.5-gd
```

Según los paquetes disponibles en el servidor, algunas extensiones pueden estar incorporadas en PHP o instalarse como dependencias.

Comprobar la versión:

```bash
php -v
```

Comprobar PHP-FPM:

```bash
systemctl status php8.5-fpm
```

La instalación utilizada por este proyecto se comunica mediante el socket:

```text
/run/php/php8.5-fpm.sock
```

Si se utiliza otra versión de PHP, será necesario ajustar ese socket y comprobar la compatibilidad con Laravel.

---

## 8. Instalar Composer

Composer administra las dependencias PHP, incluyendo Laravel y Livewire.

Instalar Composer 2 siguiendo el procedimiento oficial:

https://getcomposer.org/download/

Una vez instalado, debe estar disponible como:

```text
/usr/local/bin/composer
```

Comprobar:

```bash
composer --version
```

Para reproducir las versiones del proyecto se utilizará `composer install`, no `composer update`.

---

## 9. Instalar MariaDB

El servidor original utiliza MariaDB 12.3.

Para instalar esa misma rama, configurar el repositorio oficial de MariaDB correspondiente a Ubuntu 26.04, siguiendo la documentación:

https://mariadb.org/download/

Después de configurar el repositorio, instalar:

```bash
apt install -y mariadb-server mariadb-client
```

Iniciar MariaDB:

```bash
systemctl enable --now mariadb
```

Comprobar:

```bash
mariadb --version
```

Acceder a MariaDB localmente:

```bash
mariadb
```

En la instalación utilizada originalmente, la cuenta administrativa `root` de MariaDB emplea autenticación local mediante `unix_socket`.

### Seguridad de MariaDB

La base de datos del proyecto debe ser accesible localmente, sin exponer innecesariamente el puerto 3306 a Internet.

En nuestra configuración se utiliza:

```text
127.0.0.1:3306
```

---

## 10. Crear la base de datos y el usuario

Entrar a MariaDB:

```bash
mariadb
```

Ejecutar:

```sql
CREATE DATABASE laravel_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
```

Crear un usuario exclusivo para Laravel:

```sql
CREATE USER 'laravel_user'@'localhost'
IDENTIFIED BY 'REEMPLAZAR_POR_UNA_CONTRASENA_SEGURA';
```

Asignar permisos únicamente a la base del proyecto:

```sql
GRANT ALL PRIVILEGES ON laravel_db.*
TO 'laravel_user'@'localhost';
```

Salir:

```sql
EXIT;
```

**Importante:** reemplazar la contraseña de ejemplo por una contraseña nueva y segura.

Esa contraseña también deberá configurarse en el archivo privado `.env` de Laravel.

---

## 11. Instalar Git y clonar el proyecto

Git permite descargar el código fuente desde GitHub.

Comprobar que Git está disponible:

```bash
git --version
```

Clonar el repositorio como usuario `deploy`:

```bash
runuser -u deploy -- git clone https://github.com/gundball/laravel.git /var/www/miapp
```

Comprobar:

```bash
ls -la /var/www/miapp
```

El repositorio incluye el código de Laravel, las configuraciones y los archivos necesarios para reconstruir las dependencias.

**No incluye la base de datos, el archivo `.env`, `vendor/` ni `node_modules/`.**

---

## 12. Instalar las dependencias PHP

Ingresar al directorio del proyecto como `deploy`:

```bash
runuser -u deploy -- sh -c 'cd /var/www/miapp && composer install --no-dev --prefer-dist --optimize-autoloader'
```

Este comando utiliza `composer.lock` para instalar las versiones registradas de las dependencias.

Reconstruirá, entre otras cosas:

- Laravel 13.
- Livewire 4.
- Las librerías PHP necesarias.

**No es necesario copiar la carpeta `vendor/` desde el servidor anterior.**

Para un entorno de desarrollo donde se necesiten herramientas de pruebas, puede ejecutarse `composer install` sin `--no-dev`.

---

## 13. Instalar Node.js y npm

Este proyecto utiliza Node.js 24 LTS.

Consultar las instrucciones oficiales de instalación para Ubuntu:

https://nodejs.org/

También puede utilizarse el repositorio de NodeSource para Node.js 24:

https://github.com/nodesource/distributions

Comprobar las versiones:

```bash
node --version
```

```bash
npm --version
```

En el servidor original se utilizó Node.js 24 y npm 11.

---

## 14. Reconstruir los recursos de Vite

Instalar las dependencias JavaScript registradas en `package-lock.json`:

```bash
runuser -u deploy -- sh -c 'cd /var/www/miapp && npm ci'
```

Compilar los recursos para producción:

```bash
runuser -u deploy -- sh -c 'cd /var/www/miapp && npm run build'
```

Comprobar la compilación:

```bash
ls -la /var/www/miapp/public/build
```

Los archivos generados en `public/build/` se reconstruyen cuando es necesario.

**No es necesario copiar `node_modules/` ni `public/build/` desde el servidor original.**

---

## 15. Crear el archivo privado `.env`

Al descargar el repositorio, el archivo `.env` no estará presente porque contiene configuración privada.

Crear una copia de la plantilla:

```bash
runuser -u deploy -- cp /var/www/miapp/.env.example /var/www/miapp/.env
```

Editar:

```bash
runuser -u deploy -- nano /var/www/miapp/.env
```

Configurar los valores para el nuevo servidor:

```dotenv
APP_NAME=Laravel
APP_ENV=production
APP_DEBUG=false
APP_URL=http://IP_O_DOMINIO_DEL_SERVIDOR

DB_CONNECTION=mariadb
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=CONTRASENA_REAL_DE_MARIADB
```

Conservar las demás variables necesarias que proporciona la plantilla.

### Clave de Laravel

Si se trata de una instalación nueva, sin datos cifrados que se deban recuperar, generar una clave:

```bash
runuser -u deploy -- sh -c 'cd /var/www/miapp && php artisan key:generate'
```

**Atención:** si se está restaurando una aplicación real con información cifrada previamente por Laravel, se debe recuperar de forma segura la clave `APP_KEY` original. Generar una clave diferente impediría descifrar esos datos.

Nunca publicar `.env` ni `APP_KEY` en GitHub.

---

## 16. Restaurar una base de datos existente

El servidor original dispone de un respaldo SQL privado:

```text
/var/www/miapp/dbinicial/laravel_db.sql
```

Ese respaldo se generó mediante `mariadb-dump` y se probó importándolo en una base de datos temporal.

### Importante

La carpeta `dbinicial/` está excluida de Git.

Por lo tanto, **el archivo SQL no se descarga al clonar GitHub**.

Es necesario copiar el respaldo desde una ubicación privada y segura al nuevo servidor.

Crear la carpeta:

```bash
install -d -o deploy -g deploy -m 700 /var/www/miapp/dbinicial
```

Colocar allí el archivo de respaldo:

```text
/var/www/miapp/dbinicial/laravel_db.sql
```

Una vez transferido, establecer permisos privados:

```bash
chown deploy:deploy /var/www/miapp/dbinicial/laravel_db.sql
```

```bash
chmod 600 /var/www/miapp/dbinicial/laravel_db.sql
```

### Importar el respaldo

**Ejecutar esta operación únicamente sobre una base de datos nueva o previamente respaldada.**

```bash
mariadb laravel_db < /var/www/miapp/dbinicial/laravel_db.sql
```

Comprobar las tablas:

```bash
mariadb -e "SHOW TABLES FROM laravel_db;"
```

El respaldo inicial del proyecto contiene las siguientes nueve tablas:

- `cache`
- `cache_locks`
- `failed_jobs`
- `job_batches`
- `jobs`
- `migrations`
- `password_reset_tokens`
- `sessions`
- `users`

Si se restauró correctamente el respaldo, **no es necesario ejecutar migraciones iniciales que ya figuran como aplicadas**.

Para futuras versiones del proyecto se podrá utilizar:

```bash
runuser -u deploy -- sh -c 'cd /var/www/miapp && php artisan migrate --force'
```

Si no se cuenta con un respaldo y se desea comenzar con una base vacía, las migraciones permitirán crear las tablas del proyecto.

**Nunca utilizar `migrate:fresh` sobre una base de datos con información que se desea conservar.**

---

## 17. Configurar directorios y permisos

El código fuente pertenece al usuario `deploy`.

Establecer propietario del proyecto:

```bash
chown -R deploy:deploy /var/www/miapp
```

Laravel necesita escribir principalmente en:

```text
storage/
bootstrap/cache/
```

Dar acceso al usuario de Nginx y PHP-FPM (`www-data`):

```bash
setfacl -R -m u:deploy:rwX,u:www-data:rwX /var/www/miapp/storage /var/www/miapp/bootstrap/cache
```

Configurar permisos predeterminados heredables:

```bash
setfacl -R -d -m u:deploy:rwX,u:www-data:rwX /var/www/miapp/storage /var/www/miapp/bootstrap/cache
```

Proteger el archivo `.env`:

```bash
chmod 640 /var/www/miapp/.env
```

Permitir su lectura al proceso PHP-FPM:

```bash
setfacl -m u:www-data:r-- /var/www/miapp/.env
```

El archivo `.env` debe permanecer fuera del repositorio y no debe ser accesible directamente desde Internet.

---

## 18. Crear el enlace de almacenamiento público

Laravel utiliza un enlace simbólico para exponer los archivos públicos almacenados en `storage/app/public`.

Ejecutar:

```bash
runuser -u deploy -- sh -c 'cd /var/www/miapp && php artisan storage:link'
```

El resultado esperado es:

```text
public/storage -> storage/app/public
```

Los archivos privados almacenados en `storage/app/private` no deben exponerse mediante ese enlace.

---

## 19. Configurar Nginx para Laravel

Crear el archivo:

```text
/etc/nginx/sites-available/miapp
```

Editar como `root`:

```bash
nano /etc/nginx/sites-available/miapp
```

Ejemplo de configuración:

```nginx
server {
    listen 80;
    server_name _;

    root /var/www/miapp/public;
    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ ^/index\.php(/|$) {
        fastcgi_pass unix:/run/php/php8.5-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
        internal;
    }

    location ~ \.php$ {
        return 404;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

Esta configuración dirige las peticiones a `public/index.php`, el punto de entrada de Laravel, y evita ejecutar directamente otros archivos PHP.

**Si se utiliza un dominio, reemplazar `server_name _;` por el dominio correspondiente y configurar HTTPS antes de publicar la aplicación.**

Habilitar el sitio:

```bash
ln -s /etc/nginx/sites-available/miapp /etc/nginx/sites-enabled/miapp
```

Si el enlace ya existe, no es necesario crearlo otra vez.

Deshabilitar el sitio predeterminado si corresponde:

```bash
rm -f /etc/nginx/sites-enabled/default
```

Comprobar la configuración:

```bash
nginx -t
```

Recargar Nginx:

```bash
systemctl reload nginx
```

---

## 20. Verificar la conexión de Laravel con MariaDB

Desde el proyecto:

```bash
runuser -u deploy -- sh -c 'cd /var/www/miapp && php artisan db:show'
```

Se debe mostrar información sobre la base `laravel_db`.

Si aparece un error, revisar:

- Que MariaDB esté funcionando.
- Que `laravel_db` exista.
- Que el usuario tenga permisos.
- Que las credenciales de `.env` sean correctas.
- Que esté instalada la extensión PHP para MariaDB/MySQL.

---

## 21. Preparar Laravel para producción

La aplicación debe tener:

```dotenv
APP_ENV=production
APP_DEBUG=false
```

Nunca activar `APP_DEBUG=true` en una aplicación accesible públicamente.

Optimizar Laravel:

```bash
runuser -u deploy -- sh -c 'cd /var/www/miapp && php artisan optimize'
```

Si se modifica `.env` o algún archivo de configuración, limpiar la configuración anterior cuando sea necesario:

```bash
runuser -u deploy -- sh -c 'cd /var/www/miapp && php artisan optimize:clear'
```

Posteriormente puede ejecutarse `php artisan optimize` otra vez.

Para un despliegue público también se recomienda:

- Configurar un dominio.
- Instalar un certificado HTTPS.
- Restringir el acceso SSH.
- Configurar el firewall.
- Mantener Ubuntu y las dependencias actualizados.
- Establecer copias de seguridad automáticas.
- Configurar los trabajadores de cola y el programador de tareas si la aplicación los utiliza.

---

## 22. Comprobar el funcionamiento

Verificar los servicios:

```bash
systemctl status nginx
```

```bash
systemctl status php8.5-fpm
```

```bash
systemctl status mariadb
```

Consultar las rutas de Laravel:

```bash
runuser -u deploy -- sh -c 'cd /var/www/miapp && php artisan route:list'
```

Abrir desde un navegador:

```text
http://IP_DEL_SERVIDOR
```

En nuestro servidor inicial la aplicación se utilizó en la dirección:

```text
http://192.168.10.112
```

Esa dirección es propia de la red local original y deberá adaptarse en cada nueva instalación.

---

## 23. Actualizar el proyecto desde GitHub

Cuando publiquemos cambios nuevos en la rama `main`, podremos descargarlos desde otro servidor.

Entrar al directorio y actualizar:

```bash
runuser -u deploy -- git -C /var/www/miapp pull --ff-only origin main
```

Actualizar dependencias PHP según `composer.lock`:

```bash
runuser -u deploy -- sh -c 'cd /var/www/miapp && composer install --no-dev --prefer-dist --optimize-autoloader'
```

Actualizar dependencias JavaScript:

```bash
runuser -u deploy -- sh -c 'cd /var/www/miapp && npm ci'
```

Compilar los recursos:

```bash
runuser -u deploy -- sh -c 'cd /var/www/miapp && npm run build'
```

Aplicar migraciones nuevas, si existen:

```bash
runuser -u deploy -- sh -c 'cd /var/www/miapp && php artisan migrate --force'
```

Actualizar las cachés de Laravel:

```bash
runuser -u deploy -- sh -c 'cd /var/www/miapp && php artisan optimize'
```

En aplicaciones en producción, realizar estos pasos mediante un procedimiento de despliegue controlado y con respaldo previo de la base de datos.

---

## 24. Guardar cambios y subirlos a GitHub

El trabajo con Git debe realizarse como el usuario `deploy`.

Consultar el estado:

```bash
runuser -u deploy -- git -C /var/www/miapp status
```

Preparar los cambios:

```bash
runuser -u deploy -- git -C /var/www/miapp add .
```

Crear un commit:

```bash
runuser -u deploy -- git -C /var/www/miapp commit -m "Descripcion de los cambios"
```

Subir los cambios:

```bash
runuser -u deploy -- git -C /var/www/miapp push origin main
```

Para utilizar GitHub mediante HTTPS es necesario autenticarse mediante un método admitido, como un token de acceso personal con los permisos correspondientes.

**Nunca pegar un token dentro de los archivos del proyecto ni compartirlo públicamente.**

---

## 25. Archivos que no deben subirse a GitHub

El repositorio debe contener el código fuente, las configuraciones compartibles y los archivos de bloqueo de dependencias.

Los siguientes archivos o carpetas no deben versionarse:

| Archivo o carpeta | Motivo |
|---|---|
| `.env` | Contiene credenciales y secretos |
| `vendor/` | Se reconstruye con Composer |
| `node_modules/` | Se reconstruye con npm |
| `public/build/` | Se reconstruye con Vite |
| `dbinicial/` | Contiene respaldos privados de MariaDB |
| `public/storage` | Es un enlace simbólico generado localmente |
| Logs, sesiones y cachés | Son archivos temporales del servidor |

Los archivos `composer.lock` y `package-lock.json` **sí deben conservarse en GitHub** para facilitar instalaciones reproducibles.

---

## 26. Copias de seguridad

GitHub es una copia del código fuente, **no es una copia de seguridad completa del servidor**.

Para recuperar una instalación completa se deben conservar, por separado:

1. El repositorio Git.
2. La base de datos MariaDB.
3. El archivo `.env` y los secretos necesarios, protegidos adecuadamente.
4. Los archivos privados o subidos por usuarios dentro de `storage/`.
5. La configuración de Nginx y otros servicios que no esté documentada en Git.
6. Los certificados y demás configuraciones operativas necesarias.

### Generar un respaldo SQL

Desde el servidor original, como administrador local de MariaDB:

```bash
mariadb-dump --user=root --single-transaction --quick --routines --triggers --events laravel_db > /var/www/miapp/dbinicial/laravel_db.sql
```

Después deben comprobarse el archivo y sus permisos.

**Muy importante:** conservar también una copia cifrada o protegida fuera del servidor. Un respaldo ubicado en el mismo disco no protege frente a la pérdida total del servidor o del contenedor.

El respaldo SQL inicial fue probado restaurándolo en una base temporal, verificando las nueve tablas y eliminando después esa base de prueba.

---

## 27. Estructura principal del proyecto

```text
/var/www/miapp/
├── app/
├── bootstrap/
├── config/
├── database/
│   └── migrations/
├── dbinicial/              # Privado, excluido de Git
│   └── laravel_db.sql
├── public/
│   ├── index.php
│   └── build/              # Generado por Vite
├── resources/
│   ├── css/
│   ├── js/
│   └── views/
├── routes/
├── storage/
├── tests/
├── vendor/                 # Generado por Composer
├── node_modules/           # Generado por npm
├── .env                    # Privado, no versionado
├── .env.example
├── .gitignore
├── artisan
├── composer.json
├── composer.lock
├── package.json
├── package-lock.json
└── README.md
```

---

## 28. Recomendaciones finales

- Administrar el código mediante el usuario `deploy`.
- Reservar `root` para tareas del sistema.
- Ejecutar Composer y npm como `deploy`.
- Mantener MariaDB accesible únicamente desde los servicios autorizados.
- No publicar credenciales en GitHub.
- Mantener `APP_DEBUG=false` en producción.
- Configurar HTTPS antes de exponer el servidor a Internet.
- Realizar respaldos frecuentes y comprobar periódicamente su restauración.
- Registrar los cambios del código con Git.
- Probar las actualizaciones antes de aplicarlas en producción.

---

## 29. Referencias oficiales

- Laravel: https://laravel.com/docs
- Livewire: https://livewire.laravel.com/docs
- PHP: https://www.php.net/docs.php
- Composer: https://getcomposer.org/
- Nginx: https://nginx.org/en/docs/
- MariaDB: https://mariadb.com/docs/
- Node.js: https://nodejs.org/
- Git: https://git-scm.com/doc
- GitHub: https://docs.github.com/

---

**Repositorio:** https://github.com/gundball/laravel

**Proyecto:** Laravel 13 + Livewire 4

**Sistema base:** Ubuntu 26.04 LTS

**Servidor web:** Nginx + PHP-FPM

**Base de datos:** MariaDB

**Despliegue:** GitHub + Composer + npm + Vite

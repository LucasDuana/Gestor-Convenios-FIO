# Puesta en marcha - Gestor de Convenios FIO

Esta guía configura el proyecto en Windows con XAMPP. La instalación inicial crea el esquema y carga solo los datos estructurales y la cuenta administradora; los registros ficticios se cargan únicamente cuando se quiere probar el sistema en local. Está escrita para la carpeta `C:\xampp\htdocs\Gestor-Convenios-FIO`.

## Requisitos

- XAMPP con Apache, MySQL y PHP 8.1 o posterior.
- Composer 2.
- Node.js 18 o posterior (incluye npm).
- Extensiones PHP habilitadas: `fileinfo`, `mbstring`, `openssl`, `pdo_mysql`, `tokenizer`, `xml`, `ctype`, `curl` y `zip`.

Si los comandos `php`, `composer` o `npm` no se reconocen, agregá sus carpetas al `PATH` o ejecutalos con sus rutas completas. En XAMPP, PHP suele estar en `C:\xampp\php\php.exe`.

## Instalación desde una copia limpia

1. Copiá/cloná el repositorio en `C:\xampp\htdocs\Gestor-Convenios-FIO`.
2. En el panel de XAMPP, iniciá **Apache** y **MySQL**.
3. Abrí una terminal en la carpeta del proyecto:

   ```powershell
   cd C:\xampp\htdocs\Gestor-Convenios-FIO
   ```

4. Instalá las dependencias PHP y JavaScript:

   ```powershell
   composer install
   npm install
   ```

5. Creá el archivo de entorno. El repositorio incluye `.env.example`; no incluye `.env.production`:

   ```powershell
   Copy-Item .env.example .env
   ```

   Editá `.env` y verificá la configuración de MySQL. Para la configuración predeterminada de XAMPP:

   ```dotenv
   APP_NAME="Gestor de Convenios FIO"
   APP_ENV=local
   APP_DEBUG=true
   APP_URL=http://localhost

   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=gestor_convenios_fio
   DB_USERNAME=root
   DB_PASSWORD=
   ```

   Si tu usuario de MySQL tiene contraseña, asignala en `DB_PASSWORD`.

6. En phpMyAdmin (`http://localhost/phpmyadmin`), creá la base `gestor_convenios_fio` con cotejamiento `utf8mb4_unicode_ci`. También podés crearla desde la consola de MySQL:

   ```sql
   CREATE DATABASE gestor_convenios_fio CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   ```

7. Generá la clave de Laravel y creá el esquema con los datos estructurales, sin registros ficticios:

   ```powershell
   php artisan key:generate
   php artisan migrate --seed --seeder=ProductionSeeder
   ```

   Este comando ejecuta las migraciones y `ProductionSeeder`. Carga roles, permisos, estados, tipos de convenio, carreras y la cuenta administradora inicial, pero no genera usuarios, empresas, estudiantes ni convenios ficticios.

8. Compilá los recursos del frontend:

   ```powershell
   npm run build
   ```

9. Abrí el sistema en `http://localhost/Gestor-Convenios-FIO/public`.

   Para desarrollo también podés iniciar Vite (`npm run dev`) y, en otra terminal, Laravel (`php artisan serve`). En ese caso usá `http://127.0.0.1:8000`.

## Carga de datos de prueba para desarrollo local

Solo cuando quieras probar el sistema con registros ficticios, configurá `.env` para una base local descartable y ejecutá:

```powershell
php artisan migrate:fresh --seed
```

Este comando **borra todas las tablas y los datos existentes** de la base indicada en `.env`, vuelve a ejecutar las migraciones y carga `DatabaseSeeder`. No lo ejecutes en una base compartida o con información que quieras conservar. No uses este seeder en producción.

`DatabaseSeeder` crea estas cuentas con la contraseña `password`:

- `admin@test.com` (Admin)
- `director@test.com` (Director)
- `coordinador@test.com` (Coordinador)
- `secretary@test.com` (Secretaria)
- `docente@test.com` (Docente)

Estas credenciales son solo para desarrollo/demo; no las uses en un entorno público.

## Reiniciar la base local

Para borrar y reconstruir **toda** la base local configurada en `.env` sin datos ficticios, repetí las migraciones y el sembrado estructural:

```powershell
php artisan migrate:fresh --seed --seeder=ProductionSeeder
```

También borra todos los datos existentes. Usalo únicamente en una base local descartable; no lo ejecutes en una base con información que quieras conservar.

Las pruebas automatizadas usan la configuración declarada en `phpunit.xml`. Ejecutalas con `php artisan test`; no requieren poblar MySQL.

## Problemas frecuentes

- **No conecta a MySQL:** confirmá que MySQL esté iniciado en XAMPP y que `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME` y `DB_PASSWORD` coincidan con tu instalación.
- **Cambiaste `.env` y Laravel conserva valores anteriores:** ejecutá `php artisan config:clear` y volvé a intentar.
- **Falta una extensión PHP:** habilitala en el `php.ini` usado por la terminal y reiniciá Apache si también se sirve desde XAMPP. Verificá la configuración de CLI con `php --ini`.
- **No aparecen estilos o scripts:** ejecutá `npm install` y `npm run build`; para desarrollo, mantené activo `npm run dev`.

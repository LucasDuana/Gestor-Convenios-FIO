# Puesta en marcha - Gestor de Convenios FIO

Esta guía configura el proyecto en Windows con XAMPP, crea el esquema de la base y carga los datos iniciales del sistema. Está escrita para la carpeta `C:\xampp\htdocs\Gestor-Convenios-FIO`.

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

7. Generá la clave de Laravel, migrá el esquema e insertá los datos estructurales y la cuenta administradora:

   ```powershell
   php artisan key:generate
   php artisan migrate --seed --seeder=ProductionSeeder
   ```

   Este comando ejecuta las migraciones y `ProductionSeeder`. Crea roles y permisos, estados y tipos de convenio, carreras y el administrador inicial. No carga registros aleatorios de demostración.

8. Compilá los recursos del frontend:

   ```powershell
   npm run build
   ```

9. Abrí el sistema en `http://localhost/Gestor-Convenios-FIO/public`.

   Para desarrollo también podés iniciar Vite (`npm run dev`) y, en otra terminal, Laravel (`php artisan serve`). En ese caso usá `http://127.0.0.1:8000`.

## Cuenta inicial

El `ProductionSeeder` crea esta cuenta si todavía no existe:

- Email: `admin@fio.uner.edu.ar`
- Contraseña: `Admin1234!`

Cambiala después del primer acceso. Esta cuenta es de instalación; no la uses como credencial compartida para un entorno público.

## Datos de demostración y pruebas

`DatabaseSeeder` contiene usuarios de demostración (`admin@test.com`, `director@test.com`, `coordinador@test.com`, `secretary@test.com` y `docente@test.com`, contraseña `password`) y genera datos aleatorios con factories. **No ejecutes** `php artisan db:seed` después de `ProductionSeeder` esperando obtener el entorno de prueba: los seeders actuales no están preparados para ejecutarse juntos sobre esa misma base y el sembrado puede fallar o producir datos inconsistentes.

En particular, `DatabaseSeeder` no invoca `TestContractsSeeder`; ese seeder crea convenios suponiendo IDs existentes (por ejemplo, empresa, secretaria, empleado, docente y estudiante con ID 1), por lo que no garantiza datos válidos en una base nueva. Hasta corregir y verificar el orden y las dependencias de esos seeders, la instalación reproducible de esta guía incluye solo los datos estructurales y la cuenta administradora.

Si necesitás una base aislada para ejecutar pruebas automatizadas, el proyecto configura PHPUnit para usar SQLite en memoria (`php artisan test`); eso no modifica la base MySQL de desarrollo.

## Reiniciar la base local

Para borrar y reconstruir **toda** la base configurada en `.env`, incluidos sus datos, ejecutá:

```powershell
php artisan migrate:fresh --seed --seeder=ProductionSeeder
```

Usalo únicamente en una base local descartable. No lo ejecutes en una base con información que quieras conservar.

## Problemas frecuentes

- **No conecta a MySQL:** confirmá que MySQL esté iniciado en XAMPP y que `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME` y `DB_PASSWORD` coincidan con tu instalación.
- **Cambiaste `.env` y Laravel conserva valores anteriores:** ejecutá `php artisan config:clear` y volvé a intentar.
- **Falta una extensión PHP:** habilitala en el `php.ini` usado por la terminal y reiniciá Apache si también se sirve desde XAMPP. Verificá la configuración de CLI con `php --ini`.
- **No aparecen estilos o scripts:** ejecutá `npm install` y `npm run build`; para desarrollo, mantené activo `npm run dev`.
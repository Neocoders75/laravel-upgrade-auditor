# Upgrade en hosting compartido o sin acceso SSH

Guía complementaria de `SKILL.md`. Aplícala cuando producción sea un hosting compartido (DirectAdmin, cPanel, Plesk u otro) o cuando no exista terminal en el servidor. Mantiene todos los principios de la skill principal: copia de trabajo, respaldo verificado, sin acciones destructivas y autorización antes de cambiar nada.

## Índice
1. Qué cambia respecto a un servidor con SSH
2. Diagnóstico previo del servidor
3. Construir las dependencias en local
4. Probar en un entorno de pruebas del mismo hosting
5. Ejecutar comandos de Artisan sin terminal
6. Promoción y reversión
7. Errores comunes y limpieza final

## 1. Qué cambia respecto a un servidor con SSH

- Normalmente **no se puede ejecutar `composer`** en el servidor: `vendor/` se construye en local y se sube.
- La versión de PHP se cambia **desde el panel**, y suele afectar a todo el dominio, no solo a una carpeta.
- Los comandos de Artisan requieren alternativas (tareas cron o rutas temporales protegidas), y solo con autorización expresa.
- El soporte del proveedor puede tardar o no resolver cambios de servidor. No planificar el upgrade dependiendo de que el proveedor modifique algo: si algo exige acceso de administrador del servidor, marcarlo como **BLOQUEADOR** y proponer alternativas.

## 2. Diagnóstico previo del servidor

Registrar, sin exponer credenciales:
- Versiones de PHP que ofrece el panel (por ejemplo el selector de PHP) y cuál usa el dominio ahora. Es el dato que decide la versión objetivo (sección 8.1 de `SKILL.md`).
- Extensiones PHP necesarias para el proyecto y los paquetes. Pueden verse en el panel o mediante un archivo temporal con `phpinfo()` que se **elimina de inmediato** después de consultarlo.
- Límites relevantes: `memory_limit`, `max_execution_time`, `upload_max_filesize`.
- Si hay tareas cron, FTP o SFTP, gestor de archivos, bases de datos adicionales y subdominios.
- Cuál es la carpeta pública del dominio y dónde queda el resto de la aplicación (idealmente fuera de la carpeta pública).
- Cómo se hacen y se restauran los respaldos del hosting, y si se probó alguna restauración.

## 3. Construir las dependencias en local

1. Trabajar en la copia de trabajo, nunca sobre el original.
2. Usar en local una versión de PHP de la misma línea (por ejemplo 8.3) que la que usará producción. Laragon permite tener varias.
3. Si la versión local difiere, fijar la plataforma para reflejar el servidor real: `composer config platform.php <versión>`. Esto no sustituye cumplir los requisitos; solo evita resolver paquetes contra un PHP que producción no tiene.
4. Instalar para producción:
   ```bash
   composer install --no-dev --optimize-autoloader
   composer check-platform-reqs
   ```
5. Si el proyecto compila frontend, ejecutar el build en local y subir el resultado (por ejemplo `public/build`). No es necesario subir `node_modules`.
6. `vendor/` tiene muchos archivos: comprimirlo en un `.zip`, subirlo y descomprimirlo desde el gestor de archivos del panel suele ser más fiable que subirlo por FTP archivo por archivo. Verificar luego que el número de archivos coincide.

## 4. Probar en un entorno de pruebas del mismo hosting

Antes de tocar producción, desplegar en un subdominio o carpeta aparte:
- Con su **propia base de datos**, restaurada desde un respaldo. Nunca apuntar la prueba a la base de datos real.
- Con su propio `.env`: `APP_ENV` distinto de producción, `APP_DEBUG=false` si es accesible desde internet, y correo desviado a un registro (`MAIL_MAILER=log`) para no enviar mensajes reales a clientes.
- Sin credenciales reales de pasarelas de pago ni de mensajería, o en modo sandbox.
- Restringir el acceso a la prueba (por ejemplo con protección por contraseña del panel) para que no la indexen ni la usen usuarios reales.
- Ejecutar la lista de salida de la sección 11 de `SKILL.md`: arranque, autenticación, Livewire, consultas sobre vistas SQL, correo, colas y flujos críticos.

## 5. Ejecutar comandos de Artisan sin terminal

Solo con autorización expresa del usuario y después de un respaldo verificado. Opciones, de menos a más riesgo:

1. **Tarea cron de una sola ejecución** creada desde el panel, que ejecute el comando y se elimine después de comprobar el resultado.
2. **Limpieza manual de cachés**: vaciar los archivos de caché compilados en `bootstrap/cache` (excepto `.gitignore`) y en `storage/framework/cache`, `views` y `sessions` si procede, sin tocar `storage/app`.
3. **Ruta temporal protegida** (último recurso): con un token largo, que ejecute un único comando y que se **elimine** en cuanto termine. Una ruta de este tipo expuesta es un riesgo de seguridad grave.

Comandos habituales: `php artisan optimize:clear` y, solo si hay migraciones pendientes revisadas, `php artisan migrate --force`. No ejecutar nunca `migrate:fresh`, `db:wipe` ni equivalentes. Algunos hostings no permiten enlaces simbólicos, por lo que `storage:link` puede fallar: documentar la alternativa usada.

## 6. Promoción y reversión

1. Crear un respaldo final y **verificado** de: código, base de datos y archivos persistentes (`storage/app`, subidas de usuarios, medios).
2. Elegir una ventana de bajo tráfico y avisar si hay usuarios activos.
3. Conservar la versión anterior renombrada (por ejemplo `app_anterior`) en lugar de sobrescribirla. No pisar el `.env` ni `storage/app` de producción al subir archivos.
4. Cambiar la versión de PHP en el panel solo cuando el código nuevo ya esté subido. Recordar que el cambio puede afectar a otros sitios del mismo dominio.
5. Comprobar inmediatamente: página principal, inicio de sesión, un flujo crítico, envío de un correo de prueba, y `storage/logs` en busca de errores.
6. **Plan de reversión:** volver a la versión anterior de PHP y a la carpeta conservada. Copiar archivos no deshace cambios de base de datos ya aplicados, correos enviados ni otros efectos externos, por eso el respaldo de la base de datos es imprescindible.

## 7. Errores comunes y limpieza final

- Dejar archivos temporales accesibles (`phpinfo()`, rutas de prueba, respaldos `.zip` o `.sql` en la carpeta pública).
- Dejar `APP_DEBUG=true` en un sitio público.
- Subir el `.env` o respaldos a repositorios o carpetas públicas.
- Olvidar los permisos de escritura de `storage` y `bootstrap/cache` (los valores exactos dependen del proveedor).
- Cambiar PHP en el panel sin comprobar que `vendor/` se construyó para esa versión.
- Declarar éxito sin revisar logs ni probar correo y flujos críticos.

Al terminar, registrar en el informe de auditoría (sección 15 de `SKILL.md`): versión de PHP antes y después, archivos subidos, comandos ejecutados por la vía alternativa, respaldos y el resultado de las pruebas.

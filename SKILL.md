---
name: laravel-upgrade-auditor
description: Audita un proyecto Laravel existente y guía, paso a paso y de forma segura, la actualización entre versiones mayores (por ejemplo Laravel 10 → 11 → 12 → 13). Úsala siempre que el usuario mencione actualizar, subir de versión, migrar o hacer upgrade de Laravel, evaluar si su proyecto puede pasar a una versión más nueva, revisar la compatibilidad de PHP, Composer, Livewire u otros paquetes antes de actualizar, o preparar un respaldo antes de tocar dependencias, aunque no pida explícitamente una "auditoría". También aplica a proyectos alojados en hosting compartido sin acceso SSH (DirectAdmin, cPanel).
---

# Auditor Global y Asistente de Upgrade Seguro de Laravel

## 1. Propósito

Actúa como ingeniero senior especializado en Laravel, PHP, Composer, bases de datos, pruebas automatizadas, seguridad de dependencias y flujos de desarrollo.

Tu función es **descubrir el estado real del proyecto, explicar las opciones y guiar una actualización segura**, minimizando las preguntas que el usuario debe responder manualmente. La skill debe funcionar con proyectos Laravel existentes, estructuras antiguas o modernas, con o sin Git, con o sin GitHub, y en distintos entornos locales.

No presupongas la versión de Laravel, el sistema operativo, el gestor de dependencias, el entorno, el método de respaldo ni la experiencia del usuario. Detecta lo que pueda inferirse de los archivos y herramientas disponibles; pregunta solo por lo que no pueda determinarse de forma fiable.

**Límite importante:** una skill por sí sola no puede inspeccionar el disco ni ejecutar comandos si el agente no tiene acceso a terminal, archivos o herramientas. Primero identifica las capacidades disponibles. Si no puede acceder al proyecto, guía al usuario con comandos de diagnóstico de solo lectura y analiza sus resultados. Nunca afirmes que detectaste o ejecutaste algo sin evidencia.

## 2. Principios operativos

1. **Descubrir antes de preguntar.** Examinar archivos del proyecto y resultados de comandos disponibles antes de pedir al usuario que indique versiones o configuración.
2. **Preguntas mínimas y contextuales.** No solicitar información que ya esté confirmada por archivos o comandos. Preguntar solo por decisiones, datos inaccesibles o ambiguos.
3. **Inspección antes de modificación.** Empezar con auditoría de solo lectura.
4. **Copia de trabajo separada.** No modificar la única copia del proyecto.
5. **Respaldo recuperable.** El código, la base de datos y los archivos persistentes pueden estar en ubicaciones distintas; respaldar cada componente aplicable.
6. **Un major por vez y versión objetivo explícita.** Ruta habitual: 10 → 11 → 12 → 13, deteniéndose en la versión objetivo acordada con el usuario (ver sección 8). La versión objetivo depende del PHP realmente disponible en producción; no asumir que debe ser la última. Si el proyecto está en una versión diferente, primero investigar la ruta compatible; no inventar saltos.
7. **No asumir Git.** Detectar Git local, remoto configurado y proveedor remoto por separado. GitHub no es requisito para usar Git.
8. **Opciones de gestión de cambios.** Si no existe control de versiones, explicar las alternativas y preguntar si el usuario desea seguir con copias manuales o implementar Git local y, opcionalmente, un remoto como GitHub.
9. **Bloqueo ante riesgos críticos.** No avanzar con incompatibilidades conocidas, errores críticos o falta de una vía de recuperación.
10. **Trazabilidad.** Registrar evidencia, comandos, cambios, resultados y excepciones sin secretos.
11. **No prometer riesgo cero.** Reducir y contener el riesgo no equivale a eliminarlo.
12. **No realizar acciones destructivas sin autorización expresa.** Nunca ejecutar ni recomendar como arreglo automático `migrate:fresh`, `db:wipe`, `git reset --hard`, `git clean -fd`, borrados masivos ni restauraciones sobre el único ejemplar.
13. **Privacidad.** No pedir ni imprimir valores sensibles de `.env`, contraseñas, tokens, claves privadas o credenciales.
14. **Separar hechos de suposiciones.** Etiquetar los datos como detectados, inferidos, desconocidos o confirmados por el usuario.

## 3. Inicio adaptativo

### 3.1 Si el agente puede acceder al proyecto

No comenzar saludando con una lista de preguntas sobre versiones. Primero:
1. Identificar la raíz del proyecto.
2. Comprobar si es un proyecto Laravel y ubicar los archivos de configuración.
3. Detectar las capacidades disponibles: lectura de archivos, terminal, ejecución de comandos y creación de archivos.
4. Ejecutar únicamente inspecciones de solo lectura.
5. Presentar un resumen de lo detectado y pedir confirmación solo de los puntos que no puedan verificarse automáticamente.

### 3.2 Si el agente no puede acceder al proyecto

Explicar brevemente que no puede inspeccionar el equipo directamente desde el contexto actual. Solicitar que el usuario abra la terminal en la raíz del proyecto y ejecute el bloque de diagnóstico adecuado. No pedirle que adivine la versión.

### 3.3 Mensaje de inicio cuando se necesita interacción

> Voy a identificar primero la estructura, las versiones y el método de trabajo de este proyecto. Empezaré con comprobaciones de solo lectura; no modificaré archivos ni instalaré dependencias durante el diagnóstico. Después te presentaré lo detectado y te pediré autorización antes de preparar la actualización.

Este mensaje no sustituye la inspección. Si hay acceso real al proyecto, inspeccionar primero y hablar después con resultados concretos.

## 4. Fase A — Descubrimiento automático del proyecto

### 4.1 Identificar la raíz

Buscar indicadores como:
- `artisan`
- `composer.json`
- `composer.lock`
- `bootstrap/app.php`
- `app/`
- `routes/`
- `vendor/`, si existe

No asumir que la carpeta actual es la raíz. Si hay varios proyectos o varios archivos `artisan`, identificar las opciones y pedir al usuario que seleccione el objetivo.

No recorrer carpetas ajenas al proyecto ni leer archivos de secretos sin necesidad. Excluir `vendor/`, `node_modules/`, cachés, logs voluminosos y backups antiguos de la inspección recursiva general.

### 4.2 Detectar Laravel y su versión

Utilizar varias evidencias y priorizar las más fiables:
1. Si es seguro y está disponible, ejecutar `php artisan --version`.
2. Consultar `composer show laravel/framework` si las dependencias están instaladas.
3. Revisar la restricción de `laravel/framework` en `composer.json`.
4. Revisar `composer.lock` para la versión exacta bloqueada.
5. Si no se puede ejecutar Artisan, contrastar los datos de Composer y los archivos.

Distinguir:
- Versión declarada en `composer.json`.
- Versión bloqueada en `composer.lock`.
- Versión realmente instalada en `vendor/`.
- Versión que responde al ejecutar Artisan.

No presentar una restricción como `^12.0` como si fuera la versión exacta instalada. Si las evidencias se contradicen, detener el plan de actualización y resolver la discrepancia antes de modificar dependencias.

Si Laravel no está presente o la carpeta no parece un proyecto Laravel, explicarlo y no ejecutar el flujo de upgrade.

### 4.3 Detectar entorno y herramientas

Cuando haya acceso a terminal, detectar, si es posible:
- Sistema operativo y shell.
- `php -v`, `php --ini` y extensiones necesarias.
- `composer --version`.
- PHP usado por la terminal y, cuando pueda comprobarse, PHP del servidor web o contenedor.
- Existencia de `vendor/`, `node_modules/`, `package.json` y lockfiles de npm, pnpm o Yarn.
- Herramientas de pruebas y scripts definidos por el proyecto.
- Configuración Docker, Sail, DDEV, Laragon, XAMPP, WAMP u otra, sin asumir ninguna.
- Tipo de producción (VPS, hosting compartido con DirectAdmin, cPanel o Plesk, contenedor u otro), si hay acceso SSH, y qué versiones de PHP ofrece y cómo se cambian. Si no hay SSH, consultar `references/hosting-compartido-sin-ssh.md`.
- Motor de base de datos a partir de configuración no secreta. No mostrar credenciales.
- Estado de instalación y posibilidad de ejecutar Artisan.

Si un dato no se puede inferir con seguridad, marcarlo como “no detectado” y formular una pregunta breve solo si es necesario para el siguiente paso.

## 5. Fase B — Detectar Git, GitHub y la estrategia de respaldo

### 5.1 Inspección de Git

Comprobar si existe `.git/` en la raíz o en directorios superiores y, si se dispone de terminal, ejecutar comandos de solo lectura como:

```bash
git --version
git rev-parse --show-toplevel
git status --short
git branch --show-current
git remote -v
```

Interpretación:
- **Git no instalado:** no hay comandos Git disponibles.
- **Git instalado, proyecto sin repositorio:** no existe historial local en esa carpeta.
- **Repositorio Git local:** hay historial local; revisar si existen cambios pendientes.
- **Remoto configurado:** existe una URL remota; identificar proveedor solo por evidencia.
- **GitHub confirmado:** solo si la URL o integración lo demuestra.
- **Sin remoto:** Git local sigue siendo útil y no requiere GitHub.

No imprimir tokens que puedan aparecer embebidos en URLs remotas. Ocultar credenciales de la salida antes de mostrarla.

Si Git no está instalado, no bloquear el upgrade. Explicar la alternativa de copia de trabajo y ofrecer, sin imponerla, instalar/configurar Git local.

### 5.2 Detectar respaldo y persistencia

No asumir que una carpeta duplicada constituye un respaldo completo. Detectar lo que pueda identificarse y preguntar lo que no:
- Copia del código y archivos ocultos relevantes.
- Base de datos y forma de exportarla/restaurarla.
- Archivos subidos, documentos, medios y almacenamiento fuera del proyecto.
- Enlaces simbólicos, directorios externos y servicios locales.
- Espacio disponible para mantener original, copia de trabajo y respaldos.

No leer ni registrar secretos. Para configurar la copia, explicar que `.env` debe apuntar a una base de datos y servicios aislados.

### 5.3 Ofrecer opciones de control de cambios

Si no hay Git, presentar una recomendación breve y contextual:
- **Opción A — Sin Git:** copia manual completa, backup de base de datos y archivos persistentes, registro de cambios y pruebas en copia aislada.
- **Opción B — Git local:** historial y comparación de archivos en el equipo, sin necesidad de GitHub ni internet.
- **Opción C — Git + remoto:** Git local y un remoto privado como GitHub u otro proveedor, con medidas para no subir secretos ni datos reales.

No instalar ni configurar nada sin permiso explícito. GitHub es opcional. Si el usuario desea implementar Git, terminar primero el diagnóstico y luego ofrecer un procedimiento específico según su sistema operativo y estado del proyecto.

## 6. Fase C — Informe de diagnóstico antes de actualizar

Antes de proponer cambios, generar un informe corto con estas secciones:

1. **Proyecto detectado:** ruta o nombre, y evidencias de que es Laravel.
2. **Versiones:** Laravel declarado, bloqueado e instalado; PHP CLI; Composer; incertidumbres.
3. **Entorno:** sistema, servidor local/contenedor y gestor frontend, cuando se detecten.
4. **Control de versiones:** Git disponible, repositorio presente, cambios pendientes, remoto configurado y proveedor confirmado o desconocido.
5. **Respaldo:** código, base de datos, archivos persistentes y recuperación.
6. **Dependencias:** paquetes potencialmente incompatibles, requisitos de plataforma y alertas.
7. **Pruebas:** estado inicial, comandos disponibles y fallos preexistentes.
8. **Riesgos y bloqueadores:** impacto y evidencia.
9. **Plan propuesto:** saltos necesarios, controles y estrategia de recuperación.
10. **Autorización requerida:** indicar con claridad qué acción cambiará archivos o dependencias.

Cada elemento debe marcarse como **Detectado**, **Inferido**, **Pendiente** o **Confirmado por el usuario**. No ocultar contradicciones entre fuentes.

## 7. Fase D — Línea base y copia de trabajo

Antes de modificar dependencias:
1. Ejecutar la suite de pruebas disponible y registrar su resultado.
2. Probar Artisan y flujos críticos identificados por el usuario.
3. Anotar fallos preexistentes.
4. Crear una copia de trabajo independiente.
5. Respaldar la base de datos y archivos persistentes según corresponda.
6. Verificar que la copia no use recursos reales ni la base de datos original.
7. Registrar fecha, rutas y versión inicial en un archivo Markdown fuera del directorio de actualización.

Si Git está disponible, usarlo como apoyo para comparar cambios y crear una rama solo después de revisar el estado del repositorio. Nunca descartar cambios pendientes sin autorización.

Si no hay Git, crear copias de archivos previos antes de editarlos y mantener un registro de modificaciones. No guardar el respaldo dentro de una carpeta que se vaya a copiar recursivamente.

No comenzar el upgrade si no existe una recuperación razonable. Si la suite ya falla, separar los fallos preexistentes de los nuevos y determinar si impiden una validación fiable.

## 8. Fase E — Planificar la ruta de actualización

Ruta habitual (se recorre solo hasta la versión objetivo):
- Laravel 10 → 11.
- Laravel 11 → 12.
- Laravel 12 → 13.

### 8.1 Elegir la versión objetivo

Antes de planificar, determinar la versión de PHP realmente disponible **en producción** (no solo en la máquina local) y proponer la versión objetivo más alta compatible:
- PHP 8.3 o superior en producción: Laravel 13 es viable.
- PHP 8.2 en producción: el objetivo realista es Laravel 12 (o 11), salvo que el proveedor permita subir PHP.
- PHP inferior a 8.2 o desconocido: **BLOQUEADO** hasta confirmar qué versiones ofrece el proveedor; no proponer el upgrade todavía.

Presentar al usuario el objetivo recomendado y las alternativas con su coste y riesgo, y obtener su confirmación. Consultar la política de soporte vigente (https://laravel.com/docs/13.x/releases) para informar qué versiones siguen recibiendo correcciones de seguridad. Un objetivo inferior a la última versión es válido si está soportado y queda documentado.

Si la versión detectada es anterior a Laravel 10, superior a Laravel 13, una versión preliminar o una distribución modificada, investigar la ruta adecuada antes de dar comandos. No forzar la ruta predeterminada.

Fuentes oficiales de referencia:
- https://laravel.com/docs/11.x/upgrade
- https://laravel.com/docs/12.x/upgrade
- https://laravel.com/docs/13.x/upgrade
- https://laravel.com/docs/13.x/releases (política de soporte y versiones de PHP)
- https://laravel.com/docs/13.x/deployment

Revisar la guía completa para cada salto y las guías de los paquetes de terceros. Consultar documentación vigente cuando haya acceso web; si no existe acceso, declarar la limitación y pedir al usuario la información necesaria.

Requisitos mínimos conocidos para los saltos (contrastarlos con la página de releases vigente cuando haya acceso web, porque pueden cambiar):
- Laravel 10 (punto de partida habitual): PHP 8.1 o superior.
- Laravel 10 → 11: PHP 8.2 o superior.
- Laravel 11 → 12: PHP 8.2 o superior.
- Laravel 12 → 13: PHP 8.3 o superior.

Verificar además extensiones, Composer, motor de base de datos y requisitos específicos de las dependencias. No inferir compatibilidad completa a partir de la versión mínima de PHP.

Para cada salto, documentar:
- Cambios incompatibles aplicables.
- Paquetes bloqueadores.
- Archivos que podrían cambiar.
- Comandos que se ejecutarán.
- Pruebas de validación.
- Riesgo de datos y recuperación.
- Criterios de aprobación.

## 9. Fase F — Resolver dependencias con control

No reemplazar `composer.json` por una plantilla genérica. Analizar restricciones existentes, dependencias directas y transitivas, `require-dev`, scripts, plugins y estabilidad.

En la copia de trabajo:
1. Guardar copias de `composer.json` y `composer.lock`.
2. Ajustar las restricciones mínimas necesarias para el salto.
3. Validar:
   ```bash
   composer validate
   composer prohibits laravel/framework ^11.0
   composer update --with-all-dependencies --dry-run
   ```
   Sustituir `^11.0` por el major objetivo. Verificar disponibilidad de los comandos/opciones en la versión instalada de Composer.
4. Leer y explicar los conflictos antes de continuar.
5. Solo con autorización, ejecutar:
   ```bash
   composer update --with-all-dependencies
   ```
6. Revisar los cambios de dependencias y lockfile.
7. Ejecutar cuando estén disponibles:
   ```bash
   composer check-platform-reqs
   composer audit
   ```

No usar `--ignore-platform-reqs`, borrar el lockfile ni quitar paquetes como atajo. No ejecutar `composer update` a ciegas. `composer install` instala las versiones bloqueadas y no realiza por sí solo el upgrade mayor.

Si el PHP local difiere del de producción, resolver las dependencias contra la versión real del servidor con `composer config platform.php <versión>` únicamente para reflejar la realidad de producción, nunca para saltarse un requisito.

**Paquetes críticos.** Para cada salto, comprobar explícitamente la compatibilidad de `livewire/livewire` y de cada paquete de terceros relevante (por ejemplo con `composer prohibits laravel/framework ^X.0` y leyendo la guía de actualización de cada paquete). Un paquete sin versión compatible es un **BLOQUEADOR** hasta decidir, con autorización del usuario, si se actualiza, se reemplaza o se elimina. No asumir que subir de versión mayor un paquete es transparente.

Clasificar alertas de seguridad por paquete, severidad, exposición, mitigación y decisión. No ocultar vulnerabilidades relevantes.

## 10. Fase G — Cambios de código y configuración

Revisar solo los cambios aplicables a la versión y a la estructura real:
- Bootstrap, middleware, providers, excepciones y rutas.
- Autenticación, autorización, sesiones, CSRF y permisos.
- Modelos, casts, contratos, fachadas y APIs modificadas o retiradas.
- Migraciones, consultas SQL (incluidas las crudas con `DB::table`, `DB::select` o `DB::raw`), vistas, procedimientos almacenados y triggers que existan en la base de datos sin estar en las migraciones, índices, tipos de datos y compatibilidad del motor.
- Colas, eventos, scheduler, caché, almacenamiento y sesiones.
- Integraciones externas: correo (SMTP, Mailgun u otros), notificaciones, mensajería, pasarelas de pago y webhooks.
- Blade, componentes Livewire (propiedades, eventos, paginación y directivas `wire:`), Inertia, validación y formularios.
- Paquetes de terceros y configuración publicada.
- Logging, manejo de errores y variables de entorno.
- Vite, dependencias frontend y lockfile JavaScript.

No convertir automáticamente un proyecto a la estructura mínima de una instalación nueva si la guía permite conservar su estructura. Preservar personalizaciones. Antes de editar un archivo, guardar su versión previa o confiar en un mecanismo de comparación validado.

## 11. Fase H — Pruebas y criterios de salida

Ejecutar en un entorno aislado:

```bash
php artisan test
```

Adaptar el comando al framework de pruebas real. Utilizar `--parallel` solo cuando esté soportado y no genere colisiones.

Lista de salida por salto:
- [ ] Resolución de Composer sin conflictos.
- [ ] Requisitos de plataforma satisfechos.
- [ ] Vulnerabilidades relevantes evaluadas.
- [ ] Artisan y aplicación arrancan.
- [ ] Pruebas automatizadas pasan o las limitaciones están explícitamente documentadas.
- [ ] Flujos críticos del negocio pasan pruebas de humo.
- [ ] Migraciones validadas en base de datos de prueba.
- [ ] Autenticación y permisos comprobados.
- [ ] Componentes Livewire y vistas Blade críticas renderizan y responden.
- [ ] Las consultas sobre vistas SQL y las consultas crudas devuelven los mismos resultados que en la línea base.
- [ ] Colas, scheduler, correo, almacenamiento e integraciones comprobados si existen.
- [ ] Frontend compila si aplica.
- [ ] Logs y deprecaciones relevantes revisados.
- [ ] Archivos modificados comparados con la línea base.
- [ ] Copia original y datos originales permanecen intactos.
- [ ] Existe una vía de recuperación documentada.

No exigir que todo log de deprecación desaparezca sin evaluar origen y severidad. No declarar éxito basándose solo en una afirmación del usuario; pedir comandos y salidas relevantes.

Si no hay pruebas automatizadas, no afirmar que existe cobertura. Proponer un plan de pruebas de humo reproducible y documentar el riesgo residual.

## 12. Bloqueo y corrección de errores

Ante cualquier error:
1. Detener el salto siguiente.
2. Pedir el comando exacto y la salida relevante sin secretos.
3. Clasificar la causa.
4. Proponer una corrección mínima y reversible.
5. Guardar el archivo previo o crear un punto de recuperación antes de editar.
6. Repetir las pruebas afectadas.
7. Registrar la incidencia y actualizar el estado de auditoría.

No asegurar que un error está resuelto sin evidencia. No ignorar fallos críticos para continuar. Los fallos preexistentes deben distinguirse de los nuevos. Cualquier excepción debe ser explícita y no puede cubrir riesgos de pérdida de datos, seguridad o funciones críticas sin validar.

## 13. Migraciones y recuperación

Antes de migrar:
- Identificar las migraciones que se ejecutarán.
- Revisar operaciones destructivas, conversiones, índices y locks.
- Probar con datos representativos en una base de datos aislada.
- Evaluar el rollback: `migrate:rollback` no garantiza recuperar datos eliminados ni deshacer efectos externos.
- No migrar la base de datos original durante la fase de upgrade.

Plan de recuperación:
1. Mantener intacta la carpeta original.
2. Conservar el backup del código, la base de datos y archivos persistentes.
3. Definir criterios para descartar la copia actualizada.
4. Probar la restauración en un entorno aislado cuando sea posible.
5. Documentar los pasos para recuperar archivos, base de datos y servicios.
6. No restaurar automáticamente sobre el original.

Copiar archivos no deshace cambios de base de datos, correos enviados, pagos, webhooks ni otros efectos externos.

## 14. Promoción de la copia validada

No reemplazar el original inmediatamente después de que Composer o las pruebas terminen.

Antes de promover:
1. Aprobar los criterios de salida de cada salto.
2. Validar los flujos críticos en local.
3. Revisar `.env`, almacenamiento, enlaces simbólicos y configuración.
4. Crear respaldo final de código, base de datos y archivos persistentes.
5. Definir una ventana de sustitución y un plan de reversión.
6. Evitar que las dos copias escriban en los mismos datos.
7. Verificar rutas críticas y logs tras la sustitución.

Si la recuperación no está garantizada, mantener la copia actualizada separada y no reemplazar el original.

### 14.1 Hosting compartido o sin acceso SSH

Si el destino es un hosting compartido, un panel tipo DirectAdmin o cPanel, o no hay SSH, leer `references/hosting-compartido-sin-ssh.md` antes de planificar la promoción. El flujo cambia: dependencias construidas localmente, subida por FTP o gestor de archivos, selección de PHP desde el panel y despliegue previo en un subdominio o carpeta de prueba.

## 15. Registro de auditoría

Crear un informe Markdown o texto fuera de la carpeta que se está actualizando, con:
- Fecha, proyecto y rutas de original y copia.
- Versión inicial y versiones objetivo.
- Versiones de PHP, Composer y dependencias relevantes.
- Estado de Git, remoto y método de control de cambios.
- Estado del respaldo y verificación.
- Guías oficiales consultadas.
- Comandos y resultados.
- Archivos modificados y ubicaciones de sus copias previas.
- Cambios de dependencias.
- Pruebas ejecutadas.
- Incidencias, excepciones y riesgos residuales.
- Decisión de aprobación y plan de recuperación.

No incluir credenciales ni datos personales. Evitar guardar rutas sensibles si el informe se compartirá externamente.

## 16. Resumen al usuario y decisiones

Después del diagnóstico, presentar:
- Qué detectó el agente y cómo lo verificó.
- Qué no pudo detectar y por qué.
- Si Git está disponible, si hay repositorio y si existe remoto.
- Método de respaldo observado o pendiente.
- Riesgos y bloqueadores.
- Plan sugerido y coste/riesgo de las alternativas.
- Qué acciones requieren permiso.

Si no existe Git, preguntar de forma contextual, no genérica:
> No detecté un repositorio Git en este proyecto. Podemos continuar con una copia de trabajo y respaldos manuales, o puedo ayudarte a configurar Git local para comparar y revertir cambios. GitHub es opcional. ¿Qué opción prefieres?

Si hay Git local sin remoto:
> El proyecto ya usa Git local, pero no detecté un remoto configurado. Puedes seguir sin remoto o añadir uno privado si quieres una copia externa. No es necesario para actualizar Laravel.

Si hay Git y remoto configurado, informar del resultado sin asumir que el remoto es GitHub ni que el usuario desea cambiar su flujo.

No preguntar por preferencias que no afecten al paso actual. Solicitar autorización antes de crear repositorios, instalar herramientas, cambiar configuración, actualizar dependencias, ejecutar migraciones o reemplazar archivos.

## 17. Estados posibles

Al cerrar cada salto, asignar uno:
- **DIAGNÓSTICO INCOMPLETO:** faltan datos fiables.
- **BLOQUEADO:** hay errores o riesgos críticos sin resolver.
- **LISTO PARA ACTUALIZAR:** auditoría terminada, respaldo y aislamiento preparados, pendiente de autorización.
- **EN VALIDACIÓN:** dependencias o código modificados y faltan pruebas.
- **APROBADO:** criterios de salida satisfechos.
- **APROBADO CON EXCEPCIONES DOCUMENTADAS:** solo para excepciones admisibles, explícitas y aceptadas; nunca para ocultar riesgos críticos.
- **RECUPERACIÓN REQUERIDA:** la copia actualizada no pasa las validaciones y debe corregirse o descartarse.

No avanzar al siguiente major mientras el salto actual esté bloqueado o en validación.

## 18. Criterio de éxito

La actualización se considera terminada cuando:
- La versión objetivo acordada (por ejemplo Laravel 12 o 13) está instalada en la copia validada.
- Las versiones declaradas, bloqueadas e instaladas son coherentes.
- Los requisitos de plataforma y las dependencias son compatibles.
- Los flujos críticos han sido probados.
- Las incidencias y riesgos residuales están documentados.
- La aplicación original se conservó hasta la aprobación.
- Existe un procedimiento de recuperación viable.
- El usuario entiende qué se modificó y autoriza la promoción final.

La skill debe adaptarse al estado real del proyecto, automatizar el descubrimiento cuando existan herramientas, minimizar preguntas redundantes y nunca confundir una inferencia con una verificación.

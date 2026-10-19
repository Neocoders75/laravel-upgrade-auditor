# laravel-upgrade-auditor

Skill para agentes de IA (por ejemplo Claude) que **audita un proyecto Laravel existente y guía una actualización segura entre versiones mayores**, por ejemplo Laravel 10 → 11 → 12 → 13.

> **Estado: v0.1.** Probada por el autor en un proyecto Laravel real. Aun así, úsala primero sobre una copia de trabajo, nunca directamente en producción, y revisa cada salto antes de aprobarlo.

**Probada en:** Laravel 10 → 13 · PHP 8.3 · Local con laragon

## Qué hace

- Descubre el estado real del proyecto antes de preguntar: versión de Laravel declarada, bloqueada e instalada, PHP, Composer, entorno local y tipo de producción.
- Detecta Git y respaldos sin asumir que existen ni que se usa GitHub.
- Trabaja en una **copia aislada** y nunca modifica el original ni su base de datos.
- Elige una **versión objetivo según el PHP disponible en producción** (por ejemplo, Laravel 12 si el hosting no ofrece PHP 8.3).
- Revisa dependencias con `composer prohibits`, `--dry-run`, `check-platform-reqs` y `audit`, incluida la compatibilidad de Livewire y otros paquetes.
- Considera consultas crudas, vistas SQL, integraciones externas y componentes Livewire.
- Incluye una guía para **hosting compartido sin SSH** (DirectAdmin, cPanel y similares).
- Define criterios de salida, estados de aprobación, plan de recuperación y un informe de auditoría.

## Qué no hace

- No elimina el riesgo de un upgrade, solo lo reduce y lo contiene.
- No ejecuta acciones destructivas (`migrate:fresh`, `db:wipe`, `git reset --hard`, etc.) sin autorización expresa.
- No sustituye tus respaldos ni las guías oficiales de Laravel.
- No puede inspeccionar tu proyecto si el agente no tiene acceso a archivos o terminal. En ese caso te pedirá que ejecutes comandos de diagnóstico de solo lectura y le compartas el resultado.

## Requisitos

- Un agente que soporte skills y, idealmente, acceso a tu carpeta y terminal (por ejemplo Claude Code).
- Un proyecto Laravel con `composer.json`.
- Un respaldo del código, de la base de datos y de los archivos persistentes antes de empezar.

## Instalación

Copia esta carpeta completa con el nombre `laravel-upgrade-auditor`:

- **Claude Code, uso personal:** `~/.claude/skills/laravel-upgrade-auditor/`
- **Claude Code, solo para un proyecto:** `.claude/skills/laravel-upgrade-auditor/`
- **Otras plataformas:** consulta la documentación de tu agente sobre cómo cargar skills (el archivo principal es `SKILL.md`).

## Uso

Con la skill instalada, pide algo como:

- "Quiero actualizar mi proyecto Laravel 10 a la versión más reciente compatible. Audítalo primero."
- "Mi hosting está en DirectAdmin sin SSH. ¿Puedo subir a Laravel 12 o 13?"
- "Revisa si Livewire y mis paquetes bloquean el upgrade de Laravel."

## Estructura

```
laravel-upgrade-auditor/
├── SKILL.md                                   Flujo principal de la skill
├── references/
│   └── hosting-compartido-sin-ssh.md          Guía para hosting compartido sin SSH
├── README.md
└── LICENSE
```

## Contribuir

Se agradecen issues y pull requests, sobre todo con casos reales: paquetes que bloquean saltos concretos, particularidades de proveedores de hosting o mejoras a los criterios de salida. Evita incluir credenciales, dominios privados o datos personales en cualquier aporte.

## Licencia

MIT. Consulta el archivo `LICENSE`.

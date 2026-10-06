# CitaJusta

Plataforma de cita previa para la administración pública, diseñada para que conseguir cita no dependa de ser el más rápido.

> **Estado: en construcción.** Por ahora existen la estructura del repositorio y las convenciones de trabajo. Todo lo demás que describe este README es el objetivo del proyecto y todavía no está implementado.

## Qué es y qué problema resuelve

Conseguir cita para el DNI, el SEPE o extranjería se ha convertido en una carrera: las citas se liberan a una hora concreta y desaparecen en segundos. Muchas veces se las llevan bots que las acaparan y después las revenden.

CitaJusta está diseñada para atacar ese problema con:

- **Reservas seguras**: un hueco nunca se asigna a dos personas, aunque miles de usuarios reserven a la vez.
- **Lista de espera**: cuando alguien cancela, el hueco se ofrece automáticamente al siguiente.
- **Sala de espera virtual y límite de peticiones**: los picos de tráfico se absorben sin que el sistema caiga.
- **Modo sorteo**: las citas se solicitan durante una ventana de tiempo y se adjudican por sorteo, para que la velocidad deje de dar ventaja.
- **Auditoría completa**: queda registrado quién hizo qué y cuándo.

Hay tres tipos de usuario: ciudadano, funcionario y administrador.

## Estructura del repositorio

| Carpeta | Qué contendrá |
|---|---|
| `backend/` | API REST en Java 21 con Spring Boot: lógica de negocio, reservas, seguridad y acceso a la base de datos Oracle |
| `frontend/` | Aplicación web en Angular: portal del ciudadano, panel del funcionario y panel de administración |
| `infra/` | Todo lo necesario para levantar el entorno en local con Docker Compose (base de datos, Keycloak y, en fases posteriores, Kafka, Redis y MongoDB), junto con su configuración |
| `.github/workflows/` | Integración continua con GitHub Actions: compilará el proyecto y ejecutará los tests en cada push y en cada Pull Request |

## Flujo de ramas

- **`main` es siempre estable**: lo que hay en `main` compila y pasa todos los tests. Nunca se trabaja directamente sobre ella.
  - **Única excepción:** el commit inicial se hizo directamente sobre `main`, porque antes de él no existe ningún commit desde el que crear una rama.
- **Cada cambio se hace en una rama corta**, creada desde `main`, que vive solo lo que dura ese cambio.
- **El prefijo indica el tipo de cambio:**

| Prefijo | Para qué | Ejemplo |
|---|---|---|
| `feature/` | Una funcionalidad nueva (una historia de usuario) | `feature/HU-05-reserva-concurrente` |
| `fix/` | Corregir un error | `fix/huecos-cambio-de-hora` |
| `docs/` | Cambios solo de documentación | `docs/flujo-de-ramas` |

El ciclo de cada rama:

1. Crear la rama desde `main` actualizado.
2. Trabajar en ella con commits pequeños.
3. Abrir un Pull Request hacia `main`. *(Previsto: cuando el repositorio esté publicado en GitHub. Mientras tanto, la rama se revisa e integra en local.)*
4. Integrar en `main` cuando la revisión esté aprobada y los tests pasen. *(Previsto: que los tests se ejecuten en el CI. Hasta que exista, se ejecutan en local antes de integrar.)*
5. Borrar la rama.

## Convención de commits

Los commits siguen **[Conventional Commits](https://www.conventionalcommits.org/es/)**:

```
<tipo>(<ámbito opcional>): <descripción>
```

- La descripción va en minúscula, en imperativo ("añadir", no "añadido") y sin punto final.
- El ámbito indica la parte afectada: `reservas`, `agenda`, `auth`, `frontend`…

| Tipo | Cuándo se usa | Ejemplo |
|---|---|---|
| `feat` | Nueva funcionalidad | `feat(reservas): impedir la doble reserva de un hueco` |
| `fix` | Corrección de un error | `fix(agenda): corregir los huecos en el cambio de hora` |
| `docs` | Solo documentación | `docs: añadir el flujo de ramas al README` |
| `test` | Añadir o corregir tests | `test(reservas): añadir test de reservas concurrentes` |
| `refactor` | Cambio interno que no altera el comportamiento | `refactor(agenda): extraer el cálculo de huecos a su propia clase` |
| `style` | Formato del código, sin cambios de lógica | `style: aplicar el formato estándar al módulo de reservas` |
| `perf` | Mejora de rendimiento | `perf(disponibilidad): añadir índice a la consulta de huecos` |
| `build` | Dependencias o sistema de construcción | `build(backend): actualizar la versión de Spring Boot` |
| `ci` | Configuración de la integración continua | `ci: cachear las dependencias de Maven` |
| `chore` | Tareas de mantenimiento que no encajan en lo anterior | `chore: actualizar el .gitignore` |

Un cambio que rompe la compatibilidad se marca con `!` después del tipo, por ejemplo `feat(api)!: cambiar el formato de las fechas`.

Esta convención permite leer y filtrar la historia del proyecto, y calcular la versión automáticamente: un `fix` sube la versión de parche, un `feat` la versión menor y un cambio incompatible la versión mayor.

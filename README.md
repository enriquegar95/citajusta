# CitaJusta

Plataforma de cita previa para la administración pública, diseñada para que conseguir cita no dependa de ser el más rápido.

> **Estado: en construcción.** Por ahora existen la estructura del repositorio, las convenciones de trabajo y el entorno de desarrollo con Docker. Todo lo demás que describe este README es el objetivo del proyecto y todavía no está implementado.

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

## Entorno de desarrollo

Requisito: Docker Desktop (en Windows, con WSL 2).

Levanta dos servicios, accesibles solo desde tu propio ordenador:

| Servicio | Acceso |
|---|---|
| Oracle AI Database 26ai Free | `localhost:1521`, nombre de servicio `FREEPDB1` |
| Keycloak | Consola de administración en http://localhost:8080/admin |

### Primera vez

1. Clona el repositorio y entra en su carpeta:
```
   git clone https://github.com/enriquegar95/citajusta.git
   cd citajusta
```
2. Copia `infra/.env.example` como `infra/.env` y pon tus contraseñas. `infra/.env` no se sube al repositorio.
3. Levanta el entorno desde la raíz del repositorio:
```
   docker compose -f infra/docker-compose.yml up -d
```
4. Comprueba que los dos servicios aparecen como `healthy`:
```
   docker compose -f infra/docker-compose.yml ps
```
   La primera vez, Oracle tarda varios minutos porque crea la base de datos. Mientras tanto, ps puede mostrarlo como unhealthy: es normal. Estará listo cuando sus logs muestren DATABASE IS READY TO USE!.

### Parar el entorno

| Comando | Qué hace |
|---|---|
| `docker compose -f infra/docker-compose.yml stop` | Para los contenedores sin borrarlos |
| `docker compose -f infra/docker-compose.yml down` | Para y borra los contenedores. Los datos de Oracle se conservan en su volumen |

> **Cuidado:** `docker compose -f infra/docker-compose.yml down -v` borra también el volumen. Se pierden **todos los datos de Oracle** y la base de datos se crea de cero en el siguiente arranque.

### A tener en cuenta

- La contraseña de Oracle (`ORACLE_PWD`) solo se aplica cuando se crea la base de datos. Cambiarla después en `infra/.env` no tiene efecto, salvo que se borre el volumen.
- Keycloak no conserva datos al recrear su contenedor. *(Previsto: el realm del proyecto se importará automáticamente al arrancar.)*

## Flujo de ramas

- **`main` es siempre estable**: lo que hay en `main` compila y pasa todos los tests. Nunca se trabaja directamente sobre ella.
  - **Única excepción:** el commit inicial se hizo directamente sobre `main`, porque antes de él no existe ningún commit desde el que crear una rama.
   - **GitHub lo impone:** `main` solo acepta cambios mediante Pull Request. Antes de publicar el repositorio, las ramas se integraban en local.
- **Cada cambio se hace en una rama corta**, creada desde `main`, que vive solo lo que dura ese cambio.
- **El prefijo indica el tipo de cambio:**

| Prefijo | Para qué | Ejemplo |
|---|---|---|
| `feature/` | Una funcionalidad nueva (una historia de usuario) | `feature/HU-05-reserva-concurrente` |
| `fix/` | Corregir un error | `fix/huecos-cambio-de-hora` |
| `docs/` | Cambios solo de documentación | `docs/flujo-de-ramas` |

El ciclo de cada rama:

1. Actualizar `main` y crear la rama desde ella (`git switch main`, `git pull`, `git switch -c <rama>`).
2. Trabajar en ella con commits pequeños.
3. Subir la rama a GitHub: `git push -u origin <rama>`.
4. Abrir un Pull Request hacia `main`, con el título en Conventional Commits.
5. Integrarlo en GitHub con **Squash and merge**: los commits de la rama se convierten en un único commit en `main`, cuyo mensaje es el título del PR. Antes de integrar, los tests se ejecutan en local. *(Previsto: que se ejecuten automáticamente en el CI.)*
6. Borrar la rama. GitHub la borra automáticamente al integrar; en local, volver a `main`, hacer `git pull` y borrar la rama local.

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

El título de cada Pull Request sigue la misma convención, porque con Squash and merge es el mensaje del commit que queda en `main`. GitHub le añade al final el número del PR, por ejemplo `docs: explicar el flujo de trabajo con pull requests (#1)`.

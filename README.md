# API de Reservas IPVG

API REST para gestionar reservas de recursos tecnológicos del IP Virginio Gómez. El proyecto fue desarrollado para la asignatura Lenguaje de Programación Web II usando NestJS, TypeScript, TypeORM y MySQL.

## Requisitos previos

- Node.js 20 o superior y npm.
- MySQL en ejecución.
- Una base de datos vacía para el proyecto, por ejemplo `reservas_ipvg`.

## Instalación y configuración

1. Instalar las dependencias:

   ```bash
   npm install
   ```

2. Crear la base de datos en MySQL:

   ```sql
   CREATE DATABASE reservas_ipvg;
   ```

3. Copiar el ejemplo de variables de entorno y completar los datos locales. En PowerShell:

   ```powershell
   Copy-Item .env.example .env
   ```

   El archivo `.env` no se versiona. Las variables disponibles son `PORT`, `DB_HOST`, `DB_PORT`, `DB_USERNAME`, `DB_PASSWORD` y `DB_DATABASE`.

4. Ejecutar la aplicación en desarrollo:

   ```bash
   npm run start:dev
   ```

La API queda disponible en `http://localhost:3000` y Swagger en `http://localhost:3000/api`. TypeORM crea la tabla `reservas` automáticamente en desarrollo mediante `synchronize: true`.

## Endpoints

| Método | Ruta               | Descripción                                           |
| ------ | ------------------ | ----------------------------------------------------- |
| GET    | `/reservas`        | Lista todas las reservas.                             |
| GET    | `/reservas/buscar` | Filtra por `estado`, `tipoRecurso` y/o `solicitante`. |
| GET    | `/reservas/:id`    | Obtiene una reserva por identificador.                |
| POST   | `/reservas`        | Crea una reserva.                                     |
| PUT    | `/reservas/:id`    | Actualiza una reserva.                                |
| DELETE | `/reservas/:id`    | Elimina una reserva si no está aprobada.              |

Ejemplo de creación:

```json
{
  "solicitante": "Ana Pérez",
  "recurso": "Notebook 12",
  "tipoRecurso": "Notebook",
  "fechaReserva": "2026-12-01",
  "bloque": "08:30 - 10:00",
  "observacion": "Uso para clase de programación"
}
```

Ejemplo de búsqueda combinada:

```text
GET /reservas/buscar?estado=Pendiente&tipoRecurso=Notebook&solicitante=ana
```

## Reglas de negocio

- RN01: `solicitante` es obligatorio y tiene al menos 3 caracteres.
- RN02: `recurso` es obligatorio y tiene al menos 3 caracteres.
- RN03: los tipos permitidos son Notebook, Proyector, Cámara, Kit Robótica y Tablet.
- RN04: `fechaReserva` es obligatoria y debe ser una fecha válida.
- RN05: `bloque` es obligatorio.
- RN06: toda reserva creada inicia en estado `Pendiente`.
- RN07: los estados permitidos son Pendiente, Aprobada, Rechazada y Finalizada.
- RN08: una reserva Aprobada no puede eliminarse.
- RN09: una reserva Finalizada no puede volver a Pendiente.
- RN10: un identificador inexistente responde HTTP 404.

## Estructura y relación entre archivos

- `app.module.ts` configura variables de entorno y la conexión TypeORM.
- `main.ts` habilita CORS, `ValidationPipe` global y Swagger.
- `reservas.module.ts` registra la entidad, el controlador y el servicio.
- `reservas.controller.ts` recibe las rutas HTTP y delega el trabajo al servicio.
- `reservas.service.ts` utiliza el repositorio TypeORM y contiene las reglas de negocio.
- `entities/reserva.entity.ts` define la tabla `reservas`.
- `dto/` valida los datos de creación, actualización y búsqueda.

## Etapas de desarrollo

1. **Arquitectura inicial:** se creó la base NestJS, el módulo de reservas, controlador, servicio y una prueba inicial.
2. **Persistencia y validaciones:** se integraron TypeORM, MySQL, la entidad Reserva, DTO, variables de entorno y `ValidationPipe`.
3. **CRUD y reglas:** se implementaron las seis rutas, filtros combinables y las diez reglas de negocio.
4. **Documentación y pruebas:** se configuró Swagger en `/api`, se completó este README y se ejecutaron las verificaciones finales.

## Pruebas realizadas

Comandos ejecutados:

```bash
npm run build
npm test -- --runInBand
npx eslint "src/**/*.ts"
```

Las pruebas unitarias cubren:

- Consulta general.
- Consulta de identificador inexistente (404).
- Creación con estado Pendiente.
- Solicitante demasiado corto.
- Tipo de recurso incorrecto.
- Actualización correcta.
- Rechazo de Finalizada a Pendiente.
- Rechazo de eliminación de una reserva Aprobada.
- Eliminación permitida.
- Búsqueda con filtros combinados.

No se ejecutaron pruebas de integración contra MySQL en este equipo porque no se configuró un servidor ni credenciales locales reales. Después de crear `.env` y la base de datos, Swagger permite probar cada ruta de forma manual.

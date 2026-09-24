# HDT 4: suites de Bruno contra un API

Asigna: jueves 24 de septiembre. Entrega: jueves 1 de octubre, por MiU, individual.

## El API

```bash
uv run api.py                      # http://127.0.0.1:8000
DEMO_API_KEY=s3cret uv run api.py  # las escrituras piden X-API-Key
```

El estado vive en memoria. `POST /reset` lo devuelve a los tres libros iniciales.

| Método | Ruta | Éxito | Error |
|---|---|---|---|
| GET | `/health` | 200 | |
| GET | `/books?author=` | 200 | |
| POST | `/books` | 201 + `Location` | 422 |
| GET | `/books/{id}` | 200 | 404 |
| PUT | `/books/{id}` | 200 | 404, 422 |
| PATCH | `/books/{id}` | 200 | 404, 422 |
| DELETE | `/books/{id}` | 204 | 404 |
| HEAD | `/books/{id}` | 200 | 404 |
| OPTIONS | `/books` | 204 + `Allow` | |
| POST | `/reset` | 200 | |

## Especificación

| Campo o regla | Valor |
|---|---|
| `title`, `author` | obligatorios, de 1 a 120 caracteres |
| `year` | obligatorio, de 1450 a 2100 |
| `copies` | de 0 a 999, por defecto 1 |
| `?author=` | subcadena, sin distinguir mayúsculas |
| `PUT` | reemplaza el libro completo |
| `PATCH` | cambia solo los campos enviados |
| Con `DEMO_API_KEY` definida | POST, PUT, PATCH y DELETE sin la llave correcta dan 401 |
| Sin `DEMO_API_KEY` | no hay autenticación |

## Qué se entrega

Una carpeta `bruno/` con tres colecciones, cada una con su `bruno.json` y un entorno `local`:

| Colección | Contenido |
|---|---|
| `smoke/` | 3 a 5 requests. El servicio responde y los caminos principales viven. |
| `regresion/` | Entre 10 y 15 requests diseñados con técnicas de ISTQB. Al menos tres técnicas distintas entre partición de equivalencia, valores frontera, tabla de decisión y transición de estados. |

Reglas para las tres:

- Cada colección abre con `POST /reset` y pasa corrida dos veces seguidas.
- Cada request lleva al menos una aserción, en `assert` o en `tests`.
- Cada request de `regresion/` lleva un bloque `docs` con la técnica, la partición o el valor que cubre, y el resultado esperado según la tabla de arriba.
- La llave de API, si se usa, sale de `process.env` y no queda en el repositorio.

Evidencia, dentro del mismo ZIP:

- `junit-smoke.xml`, `junit-integracion.xml` y `junit-regresion.xml`, generados con `bru run --env local --reporter-junit junit-<suite>.xml` desde la raíz de cada colección.
- `tecnicas.md`: una tabla con una fila por request de `regresion/` (técnica, valor probado, resultado esperado) y, si la regresión encontró algo que no coincide con la especificación, qué fue.

`bru` escribe el reporte relativo al directorio de trabajo y no crea carpetas. Apuntarlo a una carpeta que no existe termina con código 2 aunque todo haya pasado.

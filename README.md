# Recepción Diaria

Checklist operativo del local para los turnos **Mañana · Pase · Tarde · Madrugada**: recepción de proveedores, tareas diarias, pase de turno, cierre y madrugada. Cada líder entra con su nombre y contraseña, y cada tarea tildada queda con quién la hizo y a qué hora.

## Qué hace

- **Detecta el día y el turno.** El día operativo cambia a las 5:30. La pestaña se elige sola según la hora (Mañana 5:30–12, Pase 12–14, Tarde 14–23, Madrugada 23–5:30).
- **Proveedores por día.**
  - `F`: llega fijo ese día.
  - `V1`/`V2`: llega una vez por semana entre los días del mismo grupo. Si Coca vino el lunes, el martes no aparece.
  - `O`: ocasional, cualquier día.
  - Cada proveedor puede ser de la mañana o de la tarde (por ejemplo, EXO).
- **Tareas diarias con hora.** Tienen estados *Próximo*, *Ahora* y *Atrasado*.
- **N/A** en cada tarea. Las que llevan `[N/A]` en la configuración arrancan como "No aplica hoy".
- **Solo hoy:** tareas por excepción para un día y un turno.
- **Pase de turno:** junta lo que quedó pendiente de la mañana.
- **Madrugada:** controla que el cierre de la tarde del mismo día esté completo.
- **Usuarios:** nombre y contraseña (hash SHA-256 con sal). Sirve para registrar quién hizo cada tarea, no es seguridad real.

## Cómo corre

`index.html` es una sola página, sin dependencias ni build.

- **Publicada como artifact de Claude:** usa la base compartida en tiempo real (capacidades `db` y `user`). Lo que tilda uno lo ven todos.
- **Abierta como archivo suelto o en cualquier hosting estático:** funciona igual, pero guarda los datos solo en ese navegador (`localStorage`).

## Modelo de datos

| Colección / doc | Contenido |
|---|---|
| `config/main` | `proveedores[]` (`id`, `nombre`, `tareas[]`, `dias{0-6: F/V1/V2/O}`, `turno?`), `controles[]` (`texto`, `hora?`, `na?`), `pase[]`, `cierre[]`, `madrugada[]` |
| `dias/AAAA-MM-DD` | `hechas{clave: true \| "na"}`, `quien{clave: {n, t}}`, `llegados{}`, `noVino{}`, `extras{id: {tab, texto, hora?}}`, `franco` |
| `usuarios/{id}` | `nombre`, `salt`, `hash` |

Claves de tareas: `proveedor__slug`, `ctl__slug`, `pase__slug`, `cierre__slug`, `mad__slug`, `x__id`.

## Recordatorios

Los avisos al celular (5:52, 7:52, 9:52 y 11:52, más 2:55 y 3:55 para madrugada) son tareas programadas de Claude que leen esta base. No forman parte del código de la página.

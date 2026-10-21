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
- **Usuarios:** nombre y contraseña. Con Firebase, las contraseñas las maneja Firebase Authentication.

## Cómo corre

`index.html` es una sola página, sin build. Elige dónde guardar según dónde se abra:

1. **Firebase**, cuando se abre desde la web (por ejemplo GitHub Pages). Usuarios con Firebase Authentication y datos en Firestore, en tiempo real para todos los líderes.
2. **Base de Claude**, cuando se publica como artifact de Claude.
3. **`localStorage`**, si no hay ninguna de las dos (solo ese navegador).

### Puesta en marcha de Firebase

1. **Authentication → Método de acceso:** activar *Correo electrónico/contraseña*.
2. **Authentication → Configuración → Dominios autorizados:** agregar `tomaspon.github.io`.
3. **Firestore Database → Reglas:** pegar el contenido de [`firestore.rules`](firestore.rules) y publicar.
4. **GitHub → Settings → Pages:** *Deploy from a branch*, rama `main`, carpeta `/ (root)`. Pages en repos privados requiere un plan pago de GitHub; si no, hacer el repo público. La config de Firebase no es secreta.

La app queda en `https://tomaspon.github.io/recepcion-diaria/`. La primera vez que entra alguien, se carga sola la configuración de proveedores y tareas.

### Usuarios

- Cada líder entra con **nombre y contraseña** (mínimo 6 caracteres). Por detrás, el nombre se convierte en un mail interno (`tomas@recepcion-diaria.app`) que nadie necesita conocer.
- **Para resetear una contraseña:** en Firebase → Authentication → Usuarios, borrar al usuario. Después, en la app, *Editar → Usuarios → Quitar*, y la persona se crea de nuevo.

## App instalable y sin conexión

- **Instalar:** en Android, Chrome ofrece *Instalar app* (también desde el menú ⋯ de la app). En iPhone: Safari → Compartir → *Agregar a inicio*.
- **Sin señal:** la app abre igual y deja tildar. Los cambios quedan guardados en el teléfono y se suben solos cuando vuelve la conexión. Arriba aparece *"Sin conexión · se guarda al volver"*.
- **Archivos:** `manifest.webmanifest`, `sw.js` (guarda la app en el teléfono) e `icons/`.

## Turno de cada líder

Al crear el usuario se elige el turno (Mañana, Tarde o Madrugada), y se puede cambiar desde el menú ⋯ → *Mi turno*. La app abre en la pestaña de ese turno: Tarde abre en *Tarde* y Madrugada en *Madrugada*. Mañana sigue la hora: *Mañana* hasta las 12 y *Pase* de 12 a 14.

## Modelo de datos

| Colección / doc | Contenido |
|---|---|
| `config/main` | `proveedores[]` (`id`, `nombre`, `tareas[]`, `dias{0-6: F/V1/V2/O}`, `turno?`), `controles[]` (`texto`, `hora?`, `na?`), `pase[]`, `cierre[]`, `madrugada[]` |
| `dias/AAAA-MM-DD` | `hechas{clave: true \| "na"}`, `quien{clave: {n, t}}`, `llegados{}`, `noVino{}`, `extras{id: {tab, texto, hora?}}`, `franco` |
| `usuarios/{uid}` | `nombre`, `email`, `turno` (Firebase). En la versión Claude/local: `nombre`, `salt`, `hash` |

Claves de tareas: `proveedor__slug`, `ctl__slug`, `pase__slug`, `cierre__slug`, `mad__slug`, `x__id`.

## Recordatorios

Los avisos al celular (5:52, 7:52, 9:52 y 11:52, más 2:55 y 3:55 para madrugada) son tareas programadas de Claude que leen esta base. No forman parte del código de la página.

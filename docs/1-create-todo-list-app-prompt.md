CONTEXT
Fresh Vite + React + TypeScript + Tailwind v4 project. Only the
default scaffold exists.

TASK
Build a To-Do app with full CRUD.

Todo: id, title (required), description, dueDate, priority
(low/medium/high), done

1. Add form — title and Add button visible; a "Show all fields"
   toggle reveals description, due date and priority
2. List showing title, description, due date, priority
3. Checkbox to toggle done — completed items get strikethrough and
   go muted
4. Edit — inline form with save and cancel
5. Delete button on each item

DESIGN
Dark, night-toned — deep indigo rather than near-black, with a single
warm accent used only on the primary button. Distinct colours for the
three priority levels. No drop shadows; separate surfaces with
background lift and borders.

IMPLEMENTATION

- Strip the Vite demo scaffold completely — the logos, the counter,
  App.css, and the default styles in index.css. Start from a clean
  slate, keep nothing.
- Tailwind v4: define the named palette tokens in an @theme block in
  index.css. No tailwind.config.js.
- Native <input type="date"> for due date
- crypto.randomUUID() for ids
- Store dates as plain YYYY-MM-DD strings
- Render the stored string as-is, no locale formatting
- No date library, no uuid package. Zero new dependencies.

CONSTRAINTS

- Hooks, all state in App
- Tailwind only, no CSS modules or styled-components
- One component per file under src/components/
- Only one item in edit mode at a time
- Priority values in a shared constant
- System font stack, no web font dependency

OUT OF SCOPE
Filtering, sorting, persistence, routing

DONE WHEN
npm run dev works, and I can add a to-do with all fields, check it
off, edit it inline, and delete it.

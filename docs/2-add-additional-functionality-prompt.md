CONTEXT
The To-Do app from the previous step. Add, edit, toggle and delete
all work. Tailwind v4, palette tokens live in the @theme block in
index.css.

TASK

1. Filters — a row above the list with two selects:
   Completed: All / Active / Completed
   Priority: All / Low / Medium / High
   Both apply together.
2. Persistence — to-dos survive a page reload.

IMPLEMENTATION

- Use the existing @theme tokens. Do not add new ones.
- localStorage, single key, JSON. No storage library.
- Load on mount, save on change.
- Handle a corrupt or missing stored value by starting empty rather
  than crashing.

CONSTRAINTS

- Filter state in App, alongside the to-dos
- Filter options from shared constants, same pattern as priorities
- The list receives already-filtered data, it does not filter itself
- Only the to-dos are persisted, not the filter selection
- Match the existing spacing and component structure
- Empty state that says what to do next

OUT OF SCOPE
A backend, sorting, animations, drag and drop

DONE WHEN
Both filters narrow the list without losing data, and my to-dos are
still there after reload.

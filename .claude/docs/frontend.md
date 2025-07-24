# Front-end Guidelines

## Tech Stack

- React 19 (function components + hooks)
- Vite 6
- Redux Toolkit for global state
- Radix UI + StyleX for styling
- React Flow for diagrams (when needed)

## Folder Layout

|- apps/
| |- frontend/
| | |- public/
| | |- src/
| | | |- routes/ # page routes
| | | |- features/ # feature folders
| | | |- layout/ # shared page layouts
| | | |- store/ # Redux store & slices
| | | |- services/ # API clients
| | | |- hooks/ # custom hooks
| | | |- utils/ # pure helpers

## Coding Conventions

| Topic         | Rule                                                    |
| ------------- | ------------------------------------------------------- |
| Components    | One per file, PascalCase name, co-locate tests & styles |
| State         | Local state for UI; Redux/Context for cross-route data  |
| API calls     | Via `src/services/*`, never directly in components      |
| Styling       | Only StyleX + Radix primitives; no inline style blocks  |
| Accessibility | Use semantic HTML, keyboard nav, ARIA labels            |

## Performance Tips

- Memoize heavy components (`React.memo` / `useMemo`).
- Use list windowing for large lists.
- Lazy-load routes & chunks with Vite’s dynamic `import()`.

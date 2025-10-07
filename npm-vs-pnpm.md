# npm vs pnpm

## Command Equivalents

| npm                 | pnpm             |
| ------------------- | ---------------- |
| `npm install`       | `pnpm install`   |
| `npm install <pkg>` | `pnpm add <pkg>` |
| `npm run <script>`  | `pnpm <script>`  |
| `npm test`          | `pnpm test`      |
| `npx <pkg>`         | `pnpm dlx <pkg>` |

## Gotchas

- If you ever had `pnpm-lockfile` merge conflict, just run `pnpm install` again

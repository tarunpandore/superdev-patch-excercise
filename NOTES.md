# Notes

### Summary of Changes
- **SQL Operator Precedence**: Parenthesized title and description `OR` matching in `TaskRepository.java`, `db/queries/search_tasks.sql`, and `db/oracle/task_search_package.sql`. Previously, default `AND` precedence leaked archived records whenever descriptions matched and bypassed status filtering on title matches.
- **Backend Latency & Robustness**: Removed artificial `Thread.sleep` complexity penalty in `TaskController.java`. Added safe `TaskStatus` enum validation (returning 400 Bad Request on invalid input rather than crashing with 500) and sanitized pagination parameters (`page >= 1`, bounded `pageSize`).
- **Frontend Race Conditions & Debounce**: Implemented `useDebounce` (300ms) to prevent keystroke request spamming. Added request cancellation flag in `useTasks` to prevent out-of-order stale renders, reset `page` to 1 on filter changes, and fixed stuck loading state on errors.

### What Was Not Changed & Why
- **In-Memory Pagination**: While production databases should use SQL `LIMIT`/`OFFSET` (e.g., Spring Data `Pageable`), the current dataset and query volume in H2 did not warrant rewriting the repository interface signature during a focused patch.
- **Task Table UI / Styling**: Left table design and layout untouched to preserve existing visual structure and avoid unnecessary diff bloat.

### Biggest Remaining Risk
- **Native SQL Search Scalability & Wildcard Injection**: Unescaped `%` / `_` characters in query strings can trigger full table scans. Furthermore, loading all matching entities into memory before slicing does not scale for large datasets. Migrating to database-level pagination (`Pageable`) with full-text search indexing is necessary for production scale.

### Tools & AI Used
- Used Gemini to diagnose operator precedence in the SQL/PLSQL layers, draft the `useDebounce` hook, and pinpoint the race condition in `useTasks`. Manually tested and verified queries and endpoints against H2 and Vite.

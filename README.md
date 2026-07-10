# World Cup Match Forecast

Predicts 2026 FIFA World Cup match scores for real, currently-scheduled fixtures, using a Poisson goal model trained on historical international match data plus this tournament's actual results.

## Run & Operate

> Adjust these to match whatever package names Replit's AI actually generated — check `artifacts/*/package.json` for the real script names and ports.

- `pnpm --filter <api-package-name> run dev` — run the backend (fetches/trains the model on startup)
- `pnpm --filter <web-package-name> run dev` — run the frontend
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- No required env vars for the core app — the model is fit in memory at startup from the fetched CSV, no database needed. Ignore/remove the `DATABASE_URL` requirement from the base scaffold unless you've added persistence on top.

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9 (inherited from the base scaffold)
- Backend: Express, fetches historical match data at startup, no persistent DB required
- Frontend: single-page app, simple dark UI
- Model: Poisson attack/defense ratings (Maher/Dixon-Coles style), fit via iterative proportional fitting, blended with FIFA ranking priors

## Where things live

- Backend startup: fetches `results.csv` from the Kaggle-sourced GitHub mirror (`martj42/international_results`), filters to 2022+ matches involving a 2026 World Cup team, fits attack/defense ratings, caches in memory (or a JSON file) so it only refits once per boot.
- Fixture list + FIFA ranking points + already-completed 2026 World Cup results are hardcoded constants (see the original build prompt) — update these as the tournament progresses.
- Prediction endpoint: takes two teams, returns expected goals, a scoreline probability grid, and win/draw/loss percentages.

## Architecture decisions

- **No database.** The model is cheap enough to refit on every server boot from the fetched CSV, so there's nothing that needs to persist between runs.
- **Fixture list is hardcoded, not user-selectable teams.** The dropdown only ever shows real, currently-scheduled or already-confirmed matchups (e.g. once both Round of 16 legs feeding a quarterfinal are done, that quarterfinal unlocks) — you should never be able to construct a hypothetical matchup between two arbitrary teams, including eliminated ones.
- **Ratings blend fitted data with a ranking-based prior.** Teams with few matches in the filtered dataset get shrunk toward a strength estimate derived from FIFA ranking points, so early-tournament predictions aren't noisy for teams with a thin sample.

## Product

- Pick a real fixture from a dropdown (Round of 16, or later rounds once confirmed).
- See the predicted scoreline, win/draw/loss odds, and a full scoreline probability grid (heatmap).

## User preferences

- Keep it simple — one backend file, one frontend file is fine; no need to over-engineer this into the full monorepo structure unless it grows.

## Gotchas

- The fixture list and "already completed" results need manual updates as real matches finish — there's no live results feed wired in.
- The historical CSV fetch needs outbound internet access at startup; if Replit's network sandboxing ever blocks it, the app has nothing to train on.
- 49,000+ rows in the source CSV — parse/filter once at startup and cache, don't refetch per request.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details.
- Data source: [International football results from 1872 to present](https://www.kaggle.com/datasets/martj42/international-football-results-from-1872-to-2017) (Mart Jürisoo), via its [GitHub mirror](https://github.com/martj42/international_results).

Hi @Kushaal-k — this is my first contribution to Tessera.io, opened for #39. I've taken it out of draft; it's ready for review whenever you have a chance.

Summary: the `/health` route in `apps/ai-service` was a scaffold placeholder returning a static `{"status":"ok"}` even with MongoDB down. This PR makes it ping MongoDB and report a `database` block (connectivity + latency) and a `models` block, returning `200` when healthy and `503` when the database is unreachable — backed by a 5-test pytest suite. The CI Pipeline (Lint, Typecheck, Test, Build) check is green.

One design question whenever you get to it: do you prefer `503` on an unhealthy dependency, or always-`200` with a status field in the body? Happy to adjust either way. Thanks for taking a look!

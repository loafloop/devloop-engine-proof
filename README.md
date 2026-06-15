# devloop-engine portability proof

Throwaway static page used to PROVE the generic config-driven devloop engine (v8)
works on a target that is NOT our dashboard.

`index.html` deliberately includes a 2000px-wide element so the engine's web
adapter screenshots it, the detector finds the horizontal overflow, and the
PR-based fix path opens a PR with an idempotent overflow guard — routed through
the zero-trust contributor + Tier-0 gate (os-contrib-gate.sh).

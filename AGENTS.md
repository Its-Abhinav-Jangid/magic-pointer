# Magic Pointer

Single-file Windhawk mod. `magic-pointer.wh.cpp` (~19.6k lines) holds the
settings schema, C++ implementation, UI, AI client and Markdown renderer. There
are no manifests, build scripts, tests or CI. The Windhawk entry points are
`Wh_ModInit`, `Wh_ModSettingsChanged` and `Wh_ModUninit` (near the end of the
file).

## Build and verify

- There is no local build. The mod uses Windhawk-provided `Wh_*` symbols and is
  compiled by Windhawk. Load it in Windhawk and use **Advanced -> Debug logging
  -> Show log output** to verify behavior.
- There are no automated tests. Verification is manual, on Windows.
- User-facing docs live in `README.md` for GitHub and in the
  `// ==WindhawkModReadme==` block for the Windhawk listing. Update the block
  too when a change should appear on the mod page.

## Git and file gotchas

- `magic-pointer.wh.cpp` is UTF-8 with CRLF and is marked `-text` in
  `.gitattributes`. Keep CRLF; do not renormalize or run a formatter on it.
- `git diff --check` reports every added line in that file as trailing
  whitespace because of the CRLF setting. That is expected.
- `.gitignore` ignores everything at the root except an allowlist. Add new
  tracked files there or they will not appear in `git status`.
- `main` mirrors upstream `rono-zeroseven/magic-pointer`. Do not commit feature
  work to `main`; use a feature branch.
- Release commits bump `@version` in the mod header and are named
  `Magic Pointer X.Y.Z: ...`. Do not bump the version for a feature branch.

## Architecture

- AI flow: `NewAiJob` -> `AiThreadProc` -> `EndpointFor` / `BuildRequestBody` ->
  `SendToApi` -> `DiagnoseModel`.
- `AiThreadProc` owns retries and model fallback. `SendToApi` is transport only;
  keep provider and model policy out of it.
- `ProviderOfKey` classifies keys by prefix: `sk-ant-` Anthropic, `sk-` OpenAI,
  everything else Gemini.
- Model unavailability is global and keyed by model name. A key-level failure
  must not call `SetModelUnavailable`, or it blocks the model for other keys.
- Never log API keys. Log the key index or the provider name instead.

## Settings

A setting normally touches:

1. A YAML entry in `// ==WindhawkModSettings==`. The block is long, so search
   for the key.
2. A field in `struct Settings` and a read in `LoadSettings()` via
   `Wh_GetIntSetting` or `Wh_GetStringSetting`.
3. The same default in the YAML and the struct.
4. `$name` and `$description` entries for every locale the file already uses.
   If a setting is intentionally English-only, say so in the README.

- Array settings are read as `name[0]`, `name[1]`, ... (see `geminiKeys`).
- Use `GetApiKeySetting` for API keys. It strips non-printable and non-ASCII
  characters; `GetStringSetting` does not.
- New link libraries must be added to `@compilerOptions` in the mod header.

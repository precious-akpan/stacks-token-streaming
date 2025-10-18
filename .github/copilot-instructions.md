<!-- This file is generated/updated by an AI assistant. Keep concise and concrete. -->
# Copilot instructions — stacks-token-streaming

Keep responses short and actionable. This repo is a small Clarinet/Clarity contract + Vitest test harness.

- Big picture
  - Single Clarity contract at `contracts/stream.clar` implementing on-chain streaming state (map `streams`, `latest-stream-id` data-var). Tests run against a local simnet provided by Clarinet SDK.
  - Tests live in `tests/*.ts` and use the Clarinet Vitest environment which exposes a global `simnet` object (see `vitest.config.js`).

- Key files to read first
  - `contracts/stream.clar` — contract storage layout and public APIs (look for `define-map streams`, `define-data-var latest-stream-id` and error constants `ERR_*`).
  - `Clarinet.toml` — manifest used by the test environment (contract paths, clarity version = 3, epoch = latest).
  - `vitest.config.js` — shows the test environment: `environment: "clarinet"`, setup files from `@hirosystems/clarinet-sdk`, and `getClarinetVitestsArgv()` (parses `vitest run --` flags).
  - `package.json` — test scripts: `npm test` (runs `vitest run`), `npm run test:report` (coverage + costs), `test:watch` (chokidar guard used for watch-triggered runs).

- How tests & checks run (developer workflows)
  - Run unit tests: `npm test` (equivalent to `vitest run`).
  - Coverage + cost reports: `npm run test:report`.
  - Contract static check: `clarinet check` (there is a workspace task named "check contracts" that runs this). Use this before tests when changing contract code.
  - Pass extra args to vitest/clarinet via: `npm test -- -- --manifest ./Clarinet.toml` or `vitest run -- --coverage --costs` (see `vitest.config.js` comments).

- Tests: important patterns
  - Tests use the global `simnet` object. Example patterns:

    - Get accounts:

      const accounts = simnet.getAccounts();
      const address1 = accounts.get("wallet_1");

    - Call a read-only function:

      const { result } = simnet.callReadOnlyFn("contract-name", "fn-name", [], address1);

    - Assertions use Clarinet matchers (loaded by the setup file): `expect(result).toBeUint(0)`, `expect(simnet.blockHeight).toBeDefined()`.

- Contract conventions and patterns (concrete)
  - Error codes defined as constants: `ERR_UNAUTHORIZED (err u0)`, `ERR_INVALID_SIGNATURE (err u1)`, etc. Follow that pattern when adding new errors.
  - Storage: `define-data-var latest-stream-id uint u0` and `define-map streams uint { sender: principal, recipient: principal, ... }` — streams are keyed by a uint id.
  - Transfers: use `(stx-transfer? amount contract-caller (as-contract tx-sender))` to move funds into the contract, and `as-contract` to get the contract principal in Clarity.
  - Incrementing IDs: contract uses `(var-set latest-stream-id (+ current-stream-id u1))` pattern — follow similar atomic updates.

- Integrations & dependencies
  - Primary packages: `@hirosystems/clarinet-sdk`, `vitest-environment-clarinet`, and `@stacks/transactions` — tests rely on Clarinet SDK to initialize simnet and matchers.
  - Tests and the Clarinet tool may read `Clarinet.toml` for manifest/path overrides.

- Where to make changes
  - Contract logic: `contracts/stream.clar`.
  - Unit tests: `tests/*.ts` (use `simnet` global and existing matcher patterns).
  - Add vitest setup helpers by editing `vitest.config.js` `setupFiles` array.

- Quick checks for AI coders
  - Prefer modifying tests alongside contract edits and run `clarinet check` then `npm test`.
  - When referencing contract storage, mirror existing map/var names (`streams`, `latest-stream-id`) and error constant naming `ERR_*`.

If anything in this draft is unclear or you'd like more details (example tests, CI steps, or expanded examples), tell me which section to expand.  
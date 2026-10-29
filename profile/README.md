<div align="center">
  <img src="hexadrop.svg" alt="Hexadrop logo" width="120" height="120" />
  <h1>Hexadrop</h1>
  <p>Opinionated building blocks and tooling for TypeScript applications built with hexagonal architecture and DDD.</p>

  [![npm](https://img.shields.io/badge/npm-%40hexadrop-CB3837?logo=npm&logoColor=white)](https://www.npmjs.com/org/hexadrop)
  [![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
  [![ESM](https://img.shields.io/badge/module-ESM-F7DF1E?logo=javascript&logoColor=black)](#principles)
  [![License: MIT](https://img.shields.io/badge/license-MIT-blue)](https://github.com/hexadrop/hexadrop/blob/main/LICENSE)
</div>

---

Hexadrop is a small ecosystem, not a framework. **Domain packages** give you the DDD and CQRS vocabulary — value objects, aggregates, events, buses, explicit errors — and **tooling packages** keep every project linted, typed, released, and updated the same way. Pick only the pieces you need: every package is ESM-only, ships its own types, and solves a single problem.

## Domain building blocks

| Package | What it gives you |
| --- | --- |
| [`@hexadrop/aggregate-root`](https://github.com/hexadrop/hexadrop/tree/main/packages/aggregate-root) | Aggregate root base class that records domain events and serializes to primitives. |
| [`@hexadrop/command`](https://github.com/hexadrop/hexadrop/tree/main/packages/command) | Commands, handlers, and buses for the write side, with sync and mock bus implementations. |
| [`@hexadrop/either`](https://github.com/hexadrop/hexadrop/tree/main/packages/either) | Either monad for explicit, typed error handling instead of exceptions. |
| [`@hexadrop/error`](https://github.com/hexadrop/hexadrop/tree/main/packages/error) | Domain error base class with coded errors, plus invalid-argument and not-found helpers. |
| [`@hexadrop/event`](https://github.com/hexadrop/hexadrop/tree/main/packages/event) | Domain events, handlers, and sync, async, and Google Pub/Sub buses. |
| [`@hexadrop/ioc`](https://github.com/hexadrop/hexadrop/tree/main/packages/ioc) | IoC container interface (`Container`) and a `@Service` decorator; DIOD is a drop-in implementation. |
| [`@hexadrop/ploc`](https://github.com/hexadrop/hexadrop/tree/main/packages/ploc) | Presentation logic containers, framework-agnostic with React bindings and hooks. |
| [`@hexadrop/query`](https://github.com/hexadrop/hexadrop/tree/main/packages/query) | Queries, handlers, and buses for the read side, with sync and mock bus implementations. |
| [`@hexadrop/types`](https://github.com/hexadrop/hexadrop/tree/main/packages/types) | Type-level utilities such as `Primitives`, `Newable`, `Awaitable`, and friends. |
| [`@hexadrop/value-object`](https://github.com/hexadrop/hexadrop/tree/main/packages/value-object) | Value objects for string, number, email, date, UUID, ULID, and Spanish document and phone numbers. |

Buses and handler registries ship with in-memory and test implementations under subpaths like `@hexadrop/command/bus/mock/bun`, `@hexadrop/event/bus/pub-sub`, or `@hexadrop/query/handlers/ioc`, so swapping infrastructure never touches your domain code.

## Tooling

| Project | What it gives you |
| --- | --- |
| [`@hexadrop/eslint-config`](https://github.com/hexadrop/eslint-config) | One-line flat ESLint config: TypeScript, React, JSON, Markdown, and Astro rules; formatting and sorted imports without Prettier. |
| [`@hexadrop/tsconfig`](https://github.com/hexadrop/tsconfig) | Base TypeScript config for Node + ESM + strictest. |
| [`@hexadrop/validate-pr-action`](https://github.com/hexadrop/validate-pr-action) | GitHub Action enforcing issue-first PRs: linked approved issue, `type:*` label, and changeset. |
| [`renovate-config`](https://github.com/hexadrop/renovate-config) | Shared Renovate preset, consumed as `github>hexadrop/renovate-config`. |
| [`github-workflows`](https://github.com/hexadrop/github-workflows) | Reusable workflows for checks, changeset detection, release PRs, publishing, and develop sync. |

The ESLint config is also published as composable slices when you do not want the full preset:

| Package | Focus |
| --- | --- |
| [`@hexadrop/eslint-config-core`](https://www.npmjs.com/package/@hexadrop/eslint-config-core) | Base rules, globals, ignores, `import-x`, and Node plugin. |
| [`@hexadrop/eslint-config-typescript`](https://www.npmjs.com/package/@hexadrop/eslint-config-typescript) | typescript-eslint parser, project service, and type-aware rules. |
| [`@hexadrop/eslint-config-stylistic`](https://www.npmjs.com/package/@hexadrop/eslint-config-stylistic) | Stylistic, Unicorn, Perfectionist, and format/Prettier rules. |
| [`@hexadrop/eslint-config-react`](https://www.npmjs.com/package/@hexadrop/eslint-config-react) | React, React Hooks, and React Refresh rules. |
| [`@hexadrop/eslint-config-json`](https://www.npmjs.com/package/@hexadrop/eslint-config-json) | JSONC parsing and canonical key sorting for `package.json` and `tsconfig.json`. |
| [`@hexadrop/eslint-config-markdown`](https://www.npmjs.com/package/@hexadrop/eslint-config-markdown) | Markdown processors and parser wiring. |
| [`@hexadrop/eslint-config-astro`](https://www.npmjs.com/package/@hexadrop/eslint-config-astro) | Astro plugin, parser, and Prettier plugin rules. |

## Quick start

### 1. Model your domain

```bash
bun add @hexadrop/aggregate-root @hexadrop/event @hexadrop/value-object @hexadrop/types
```

```typescript
import AggregateRoot from '@hexadrop/aggregate-root';
import DomainEvent from '@hexadrop/event';
import type { Primitives } from '@hexadrop/types/primitives';
import EmailValueObject from '@hexadrop/value-object/email';

class UserCreatedEvent extends DomainEvent {
	static override EVENT_NAME = 'user.created';

	constructor(readonly email: string, aggregateId: string) {
		super(UserCreatedEvent.EVENT_NAME, aggregateId);
	}
}

class User extends AggregateRoot {
	constructor(
		readonly id: string,
		readonly email: EmailValueObject
	) {
		super();
	}

	static create(id: string, email: string): User {
		const user = new User(id, new EmailValueObject(email));
		user.record(new UserCreatedEvent(email, id));

		return user;
	}

	override toPrimitives(): Primitives<User> {
		return { id: this.id, email: this.email.value };
	}
}

const user = User.create('01H8Z3M8Y6K8C3B5A9D2E7F4G1', 'dev@hexadrop.dev');
console.log(user.pullDomainEvents()); // [UserCreatedEvent]
```

### 2. Set up the tooling

```bash
bun add -d eslint @hexadrop/eslint-config @hexadrop/tsconfig
```

```js
// eslint.config.js
import hexadrop from '@hexadrop/eslint-config';

export default hexadrop();
```

```json
// tsconfig.json
{
	"extends": "@hexadrop/tsconfig"
}
```

```json
// renovate.json
{
	"extends": ["github>hexadrop/renovate-config"]
}
```

Every package README documents its own install steps, API, and examples; the CI workflows live in [`github-workflows`](https://github.com/hexadrop/github-workflows) and are consumed as `hexadrop/github-workflows/.github/workflows/check.yml@v1` and friends.

## Principles

- **Hexagonal first.** Ports and adapters, domain events, and CQRS primitives are the default vocabulary, not an afterthought.
- **One purpose per package.** No meta-package hiding unrelated utilities; compose what you need.
- **Bare minimum dependencies.** Small transitive surface, no surprises.
- **Errors are values.** Use cases return `Either<DomainError, T>` instead of throwing.
- **ESM-only, strict TypeScript.** Modern Node and Bun, no CommonJS shims.
- **Consistency is automated.** Lint, formatting, dependency updates, and releases are shared across the organization.
- **Tests as documentation.** Every package documents itself through its tests and README.

## Repositories

| Repository | What's inside |
| --- | --- |
| [`hexadrop`](https://github.com/hexadrop/hexadrop) | Monorepo with all the domain packages: aggregates, events, CQRS buses, value objects, and more. |
| [`eslint-config`](https://github.com/hexadrop/eslint-config) | `@hexadrop/eslint-config` plus its composable slices. |
| [`tsconfig`](https://github.com/hexadrop/tsconfig) | `@hexadrop/tsconfig` base TypeScript config. |
| [`renovate-config`](https://github.com/hexadrop/renovate-config) | Shared Renovate preset for every repository. |
| [`github-workflows`](https://github.com/hexadrop/github-workflows) | Reusable GitHub Actions workflows for CI and releases. |
| [`validate-pr-action`](https://github.com/hexadrop/validate-pr-action) | Action that enforces the issue-first PR rules. |

## Contributing

Contributions are welcome, but the workflow is issue-first: **no pull request without an approved issue**. Open an issue with the Bug Report or Feature Request template, wait for the `status:approved` label, and then send your PR.

Read [CONTRIBUTING.md](https://github.com/hexadrop/hexadrop/blob/main/CONTRIBUTING.md) for the full workflow: label system, GitFlow branches, Conventional Commits, changesets, and release process.

## License

MIT — see [LICENSE](https://github.com/hexadrop/hexadrop/blob/main/LICENSE).

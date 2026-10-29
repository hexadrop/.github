<div align="center">
  <img src="hexadrop.svg" alt="Hexadrop logo" width="120" height="120" />
  <h1>Hexadrop</h1>
  <p>Opinionated building blocks for TypeScript applications built with hexagonal architecture and DDD.</p>

  [![npm](https://img.shields.io/badge/npm-%40hexadrop-CB3837?logo=npm&logoColor=white)](https://www.npmjs.com/org/hexadrop)
  [![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
  [![ESM](https://img.shields.io/badge/module-ESM-F7DF1E?logo=javascript&logoColor=black)](#design-principles)
  [![License: MIT](https://img.shields.io/badge/license-MIT-blue)](https://github.com/hexadrop/hexadrop/blob/main/LICENSE)
</div>

---

Hexadrop is a set of small, single-purpose packages. You pick the pieces you need — value objects, aggregates, domain events, CQRS buses, an IoC container — and skip the rest. Every package is **ESM-only**, ships TypeScript types, and keeps its dependency tree as small as possible.

## Packages

| Package | Description |
| --- | --- |
| [`@hexadrop/aggregate-root`](https://github.com/hexadrop/hexadrop/tree/main/packages/aggregate-root) | Aggregate root base class with domain event recording. |
| [`@hexadrop/command`](https://github.com/hexadrop/hexadrop/tree/main/packages/command) | Commands, handlers, and buses for the CQRS write side. |
| [`@hexadrop/either`](https://github.com/hexadrop/hexadrop/tree/main/packages/either) | Either monad for explicit, typed error handling. |
| [`@hexadrop/error`](https://github.com/hexadrop/hexadrop/tree/main/packages/error) | Domain error base class with coded errors, plus invalid-argument and not-found helpers. |
| [`@hexadrop/event`](https://github.com/hexadrop/hexadrop/tree/main/packages/event) | Domain events, handlers, and sync, async, and Google Pub/Sub buses. |
| [`@hexadrop/ioc`](https://github.com/hexadrop/hexadrop/tree/main/packages/ioc) | Minimal IoC container with a `@Service` decorator. |
| [`@hexadrop/ploc`](https://github.com/hexadrop/hexadrop/tree/main/packages/ploc) | Presentation logic containers, framework-agnostic with React bindings. |
| [`@hexadrop/query`](https://github.com/hexadrop/hexadrop/tree/main/packages/query) | Queries, handlers, and buses for the CQRS read side. |
| [`@hexadrop/types`](https://github.com/hexadrop/hexadrop/tree/main/packages/types) | Small type-level utilities such as `Primitives`. |
| [`@hexadrop/value-object`](https://github.com/hexadrop/hexadrop/tree/main/packages/value-object) | Value objects for string, number, email, date, UUID, ULID, and Spanish identifiers. |

## Quick start

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

Each package README has its own install instructions, API reference, and usage examples. Buses also ship in-memory and mock implementations (`@hexadrop/command/bus/mock/bun`, `@hexadrop/query/handlers/ioc`, and friends) so tests stay fast.

## Design principles

- **Hexagonal first.** Ports and adapters, domain events, and CQRS primitives are the default vocabulary, not an afterthought.
- **One purpose per package.** No meta-package hiding a bundle of unrelated utilities.
- **Bare minimum dependencies.** Low transitive surface, no surprises.
- **Functional error handling.** Use cases return `Either<DomainError, T>` instead of throwing.
- **ESM-only, strict TypeScript.** Modern Node and Bun, no CommonJS shims.
- **Tests as documentation.** Every package documents itself through its tests and README.

## Repositories

| Repository | What's inside |
| --- | --- |
| [`hexadrop`](https://github.com/hexadrop/hexadrop) | Monorepo with all `@hexadrop/*` packages. |
| [`eslint-config`](https://github.com/hexadrop/eslint-config) | Shared ESLint configuration (`@hexadrop/eslint-config`). |
| [`tsconfig`](https://github.com/hexadrop/tsconfig) | Base TypeScript configuration for Node + ESM + strictest (`@hexadrop/tsconfig`). |
| [`github-workflows`](https://github.com/hexadrop/github-workflows) | Reusable GitHub Actions workflows used across the organization. |
| [`validate-pr-action`](https://github.com/hexadrop/validate-pr-action) | Action that validates PRs against issue linkage, approval labels, type labels, and changesets. |

## Contributing

Contributions are welcome, but the workflow is issue-first: **no pull request without an approved issue**. Open an issue with the Bug Report or Feature Request template, wait for the `status:approved` label, and then send your PR.

Read [CONTRIBUTING.md](https://github.com/hexadrop/hexadrop/blob/main/CONTRIBUTING.md) for the full workflow, label system, branch naming, and commit conventions.

## License

MIT — see [LICENSE](https://github.com/hexadrop/hexadrop/blob/main/LICENSE).

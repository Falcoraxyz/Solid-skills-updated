# Solid Skills

Professional software engineering skills for AI coding agents. Transforms code into senior-engineer quality software through SOLID principles, TDD, clean code practices, and professional software design.

Skills follow the [Agent Skills](https://github.com/anthropics/skills) format.

## Available Skills

### solid

Transform junior-level code into senior-engineer quality software. Primarily designed for **TypeScript** and **NestJS** projects, but applicable to any object-oriented codebase.

**Use when:**

- Writing any code (features, fixes, utilities)
- Refactoring existing code
- Planning or designing architecture
- Reviewing code quality
- Debugging issues
- Creating tests
- Making design decisions

**Core principles:**

| Principle | Focus |
|-----------|-------|
| TDD | Red-Green-Refactor cycle, tests before code |
| SOLID | Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion |
| Clean Code | Meaningful names, small functions, no comments needed |
| Design Patterns | Creational, Structural, Behavioral patterns |
| Architecture | Vertical slicing, dependency rule, clean architecture |

**Reference documentation included:**

- `solid-principles.md` - SOLID principles with TypeScript examples
- `tdd.md` - Test-Driven Development practices
- `testing.md` - Testing strategies and patterns
- `clean-code.md` - Clean code guidelines
- `code-smells.md` - Code smell detection and fixes
- `design-patterns.md` - GoF patterns with examples
- `architecture.md` - Clean architecture principles
- `nestjs.md` - Applying the principles in NestJS (modules, DI tokens, use cases, testing)
- `object-design.md` - Object stereotypes and responsibilities
- `complexity.md` - Managing essential vs accidental complexity

**Key features:**

- Enforces TDD workflow (write failing test first)
- Detects and fixes code smells automatically
- Applies SOLID principles to every class and function
- Uses value objects for domain primitives (IDs, emails, money)
- Follows Law of Demeter and Tell Don't Ask
- Keeps methods under 10 lines, classes under 50 lines

## Installation

### Using the skills CLI

```bash
npx skills add Falcoraxyz/solid-skills
```

### Manual install (Antigravity and other Agent Skills compatible tools)

Copy the `skills/solid/` folder (it contains `SKILL.md` and `references/`) to one of these locations:

| Scope | Location |
|-------|----------|
| Single project | `<workspace-root>/.agents/skills/solid/` |
| All projects | `~/.gemini/config/skills/solid/` |

```bash
# Project-level example
mkdir -p .agents/skills
cp -r skills/solid .agents/skills/solid
```

In Antigravity you can check that the skill is loaded from the **Customizations** menu in the agent side panel, and invoke it explicitly by name, for example: "Use the solid skill to refactor this service."

Note: `.agent/skills` (singular) is still supported for backward compatibility. Locations may change between versions, so check the [Antigravity skills documentation](https://antigravity.google/docs/skills) if the skill is not detected.

## Usage

Skills are automatically available once installed. The agent will use them when relevant tasks are detected.

**Examples:**

- "Implement a user registration feature"
- "Refactor this service to follow SOLID principles"
- "Review this code for quality issues"
- "Add tests for this module"
- "Design the architecture for a payment system"

## Skill Structure

```
skills/
└── solid/
    ├── SKILL.md           # Main skill instructions
    └── references/        # Supporting documentation
        ├── solid-principles.md
        ├── tdd.md
        ├── testing.md
        ├── clean-code.md
        ├── code-smells.md
        ├── design-patterns.md
        ├── architecture.md
        ├── nestjs.md
        ├── object-design.md
        └── complexity.md
```

## Credits

- Clean code practices in `clean-code.md` include concepts from the **"Clean Code" course summary** by [Academind GmbH / Maximilian Schwarzmuller](https://academind.com) (c) 2020.

## License

MIT. See [LICENSE](LICENSE).

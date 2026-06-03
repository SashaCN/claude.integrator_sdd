# Coding standards — what `review` / `reviewer` judge against (this fork)

> **Reference-only.** Not a skill. The team's coding conventions, folded into the **plugin** (per
> [`./paths.md`](./paths.md) Rule 5 — the writing method / conventions live in the plugin, not in `docs/`). `review` + the
> `reviewer` agent read this as the **documented-standards baseline**; **repository code still wins**.
> Calibrated for **Prime.Integrator** (`KeepgoEU/prime.integrator`) — PHP 8.5 / Laravel 13.
>
> **Live source of truth in the repo.** This file mirrors the repo's own committed standards —
> `docs/standards/php-coding.md` + `docs/standards/coding-standards.md`. When they disagree, the **repo
> files win** (they are the canonical, CTO-propagated copies); flag the drift so this mirror is updated.

## TL;DR (українською)

Конвенції команди живуть **тут, у плагіні** (не у волті — Rule 5), але **дзеркалять** committed-стандарти
репо (`docs/standards/php-coding.md`, `coding-standards.md`). `reviewer` судить diff проти цього
baseline, але **код репозиторію перемагає** будь-яке правило: дивись сусідні файли, і якщо код
послідовно відхиляється — рев'ю за кодом + флаг прогалини. Це **Prime.Integrator** (новий ребілд):
PHP 8.5, `declare(strict_types=1)` у **кожному** файлі, `final`-класи за замовчуванням, інтерфейси з
суфіксом `Contract` **без** `I`-префікса, плюс проєкт-специфічне правило `apiRequest()`.

## How to use these

- **Repository code wins.** Check sibling files for the real local convention; if the code
  consistently diverges from a rule here, review against the code and **flag the gap**.
- **The repo's `docs/standards/` is the canonical normative** — `php-coding.md` (style) +
  `coding-standards.md` (general). This plugin file is the reviewer-facing mirror; on conflict, defer
  to the repo files and flag the mirror as stale.
- These are the **documented baseline**, not an exhaustive linter — prioritise correctness and
  AC-compliance over style (pure style findings are `MINOR`).

## PHP / Laravel — code style

PHP 8.5, Laravel 13. Modern syntax expected: `enum`, `readonly`, `match`, named arguments,
first-class callables, typed constants, property hooks, `never` return type.

### `declare(strict_types=1)` — mandatory

**Every PHP file starts with `declare(strict_types=1);`** (repo `php-coding.md` → "Strict Types";
100% of `app/` enforces it). A file missing it is a finding.

```php
<?php

declare(strict_types=1);

namespace App\Integrations\Mediators;
```

### `final` by default; `readonly` for immutable structures

- **Every concrete class is `final`.** Non-final only when `abstract`.
- **DTOs / Value Objects / Payloads / API responses** are `final readonly class` with
  `public readonly` constructor properties.
- Eloquent Models, Controllers, and classes relying on inheritance are exempt from `readonly` but
  must still be `final`.

### Named arguments — mandatory for 2+ params

```php
// ✅
$mediator->getSimCardDetails(iccid: $simCard->iccid);

dispatch(new ExternalSimCardCheckExpectedStatusJob(callbackUuid: $job->callback_uuid))
    ->onConnection($simCard->getJobConnection())
    ->onQueue('high');

// ❌
$mediator->getSimCardDetails($simCard->iccid);
```

### No boolean parameters in public signatures

A public method must not take a `bool` flag — split into two methods or take an enum.

### Array keys are constants

```php
// ✅
$details[SimCardProperty::STATUS_ID]       = SimCardStatus::ACTIVE_ID;
$details[SimCardProperty::PROVIDER_STATUS] = $providerStatus;

// ❌
$details['status_id'] = 3;
```

### `match`, not `switch`

```php
return match ($statusId) {
    SimCardStatus::ACTIVE_ID    => States::ACTIVE,
    SimCardStatus::SUSPENDED_ID => States::SUSPEND,
    default                     => throw new RuntimeException('Incorrect status ID'),
};
```

Multiple values share an arm (`States::SUSPEND, States::PENDING_SUSPEND => …`).

### Quality rules (repo `php-coding.md` → "Code Quality Rules")

- **No debug functions** in committed code: `dd()`, `dump()`, `ray()`, `var_dump()`, `print_r()`.
- **`env()` forbidden outside `config/`** — read config values, not env, in app code.
- **Typed exceptions only** — never throw base `\Exception` directly.
- **Guard clauses / early return** over deep nesting; methods ~25 lines or less.
- **Prefer closures + Collections** over raw loops and arrays.
- A PHPStan suppression requires an inline explanation.

### Class structure order

```php
<?php

declare(strict_types=1);

namespace App\Integrations\Mediators;

// Imports: Internal → External → Throwable
use App\Integrations\Mediators\_Abstracts\AbstractMediator;
use App\Integrations\Mediators\_Contracts\SimCardChangeStatusContract;
use Carbon\CarbonImmutable;
use Throwable;

final class VerizonMediator extends AbstractMediator implements SimCardChangeStatusContract
{
    // 2. Traits   3. Properties   4. Constructor   5. public   6. protected   7. private
    use SimCardChangeStatusValidationTrait;

    public function __construct(
        private readonly Operator $operator,
        private readonly bool     $isCheck = false,
    ) {}
}
```

### Naming

| Entity | Convention | Example |
|---|---|---|
| Class | PascalCase, `final` | `VerizonMediator` |
| Interface | **`{Name}Contract`** — suffix only, **never** `I`-prefix or `Interface` | `SimCardGetDetailsContract` |
| Abstract class | `Abstract{Name}` | `AbstractMediator` |
| Trait | `{Name}Trait` | `SimCardUsagePersistenceTrait` |
| DTO | `{Name}DTO`, `final readonly` | `ProviderMonthlyUsageDTO` |
| Enum | string-backed, PascalCase cases | `Channel::Email` |
| Variable / non-persisted property | camelCase | `$callbackUuid` |
| Eloquent attribute / DB column | snake_case | `$model->callback_uuid` |
| Method | camelCase | `getSimCardDetails()` |
| Constant | UPPER_SNAKE_CASE | `ACTIVE_ID` |
| Grouping folder | PascalCase, `_`-prefix | `_Contracts/`, `_Abstracts/`, `_Traits/` |

### Imports & PHPDoc

- Always `use`-import at the top; **never** an FQCN inline in the code body.
- In PHPDoc (`@param`/`@return`/`@var`/`@throws`) use FQCN with a leading `\` for IDE resolution.
- Write a docblock only when it adds value beyond native types — descriptions, `@throws`, generics
  (`Collection<int, Model>`), or union types not expressible natively.

```php
/**
 * @param  string                $iccid
 * @param  \Carbon\CarbonImmutable|null $from
 * @return \App\DTOs\ProviderMonthlyUsageDTO
 * @throws \Throwable
 */
```

## Project-specific — the `apiRequest()` rule

⚠️ **Project-wide rule (Integrator mediators):** every provider API call MUST go through
`AbstractMediator::apiRequest()` (`app/Integrations/Mediators/_Abstracts/AbstractMediator.php`).
Calling `$this->client->...->send()` directly is forbidden.

### Why

`apiRequest(callable $request, int $attempts = 3)` provides **retry** (3 attempts default,
configurable per call), **exception unwrapping** (provider exceptions → typed domain exceptions via a
mediator override), **delay** (overridable `getDelay($exceptionMessage)`), and **token refresh** for
OAuth mediators (they wrap `apiRequest()` in an extra layer). A direct `->send()` has none of these.

### Canonical pattern

```php
// ✅
$response = $this->apiRequest(fn() => $this->client->getSimCard($iccid)->send());

// ❌
$response = $this->client->getSimCard($iccid)->send();
```

### Anti-pattern: bypass

```php
// ❌ — ad-hoc recovery outside apiRequest() won't survive a provider rate-limit consistently
try {
    return $this->client->getSubscriberDetails($iccid)->send();
} catch (RateLimitException $e) {
    sleep(60);
    return $this->client->getSubscriberDetails($iccid)->send();
}
```

Instead, override `getDelay()` so `apiRequest()` handles it:

```php
protected function getDelay(string $exceptionMessage): int
{
    return str_contains($exceptionMessage, 'rate limit') ? 60 : 1;
}
```

### Legit exceptions to the rule

- Auth bootstrap in a constructor / authenticator; test fixtures.
- A mediator that **overrides** `apiRequest()` to add a layer (e.g. token refresh on 401, or a
  bespoke retry loop) — legitimate; the anti-pattern is a **bypass without an override**.

### Validation (grep for bypasses)

```bash
grep -rn '->send()' app/Integrations/Mediators/ | grep -v 'apiRequest' | grep -v 'fn() =>'
```

Any hit is a potential bypass.

## Discipline

- **Repository code wins** — these document the intent; the code is the truth.
- **`docs/standards/` is the canonical normative** — this file mirrors it; flag drift.
- **Not exhaustive** — the absence of a rule here is not permission; match the repo's patterns.
- **EN-only**, like every plugin reference file.

## Where this is read

- **`reviewer`** (agent) — the documented-standards baseline for its stage-2 Conventions pass.
- **`review`** / `review-dimensions.md` — the stage-2 Conventions dimension points here.

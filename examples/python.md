# DTTC end to end — Python

One small feature, built under DTTC from nothing to working code.

**Feature:** apply a discount code to an order subtotal.

**Toolchain:** `pytest` for step 3, `mypy --strict` for step 2.

> In Python, type annotations are not enforced at runtime. DTTC says types are
> hard constraints — so in Python the constraint only exists if `mypy --strict`
> runs. Treat a mypy failure exactly like a failing test: the step is not done.

---

## 1. Design (D)

No UI here, so the design step is textual. State what must be satisfied:

- A discount code carries a benefit, an expiry date, and a minimum subtotal.
- A benefit is either a percentage off or a fixed amount off.
- Applying a code to a subtotal either succeeds or is rejected with a reason.
- A code past its expiry is rejected.
- A subtotal under the code's minimum is rejected.
- A fixed amount larger than the subtotal never produces a negative total.

Rejection is a **result**, not an exception: the caller is expected to handle it,
so it belongs in the return type.

Nothing above says *how*. That is the whole point of step 1.

---

## 2. Types (T)

Turn the design into shapes and one signature. No logic yet.

```python
from dataclasses import dataclass
from datetime import date
from decimal import Decimal
from typing import Literal


@dataclass(frozen=True)
class PercentageOff:
    percent: Decimal


@dataclass(frozen=True)
class AmountOff:
    amount: Decimal


Benefit = PercentageOff | AmountOff


@dataclass(frozen=True)
class DiscountCode:
    code: str
    benefit: Benefit
    expires_on: date
    minimum_subtotal: Decimal


@dataclass(frozen=True)
class Applied:
    total: Decimal
    saved: Decimal


@dataclass(frozen=True)
class Rejected:
    reason: Literal["expired", "below_minimum"]


Outcome = Applied | Rejected


def apply_discount(subtotal: Decimal, code: DiscountCode, on: date) -> Outcome:
    raise NotImplementedError
```

Read what the types already enforce:

- `Decimal`, not `float` — money does not round like a float.
- `on: date` is a parameter, not `date.today()` — expiry is testable.
- `Literal[...]` means a typo in a reason is a type error, not a runtime surprise.
- `Outcome` forces the caller to handle rejection. It cannot be ignored.

`mypy --strict` passes. The contract is fixed. Now, and only now, behavior.

---

## 3 + 4. Tests and Code (T, C)

**One test at a time. One implementation at a time.** The active test defines
the work — write it, watch it fail, write the smallest code that satisfies it.

### Iteration 1 — the percentage case

```python
TEN_PERCENT = DiscountCode(
    code="SAVE10",
    benefit=PercentageOff(percent=Decimal(10)),
    expires_on=date(2026, 12, 31),
    minimum_subtotal=Decimal(0),
)


def test_percentage_code_reduces_the_subtotal() -> None:
    outcome = apply_discount(Decimal(200), TEN_PERCENT, on=date(2026, 6, 1))
    assert outcome == Applied(total=Decimal(180), saved=Decimal(20))
```

```python
def apply_discount(subtotal: Decimal, code: DiscountCode, on: date) -> Outcome:
    saved = subtotal * Decimal(10) / Decimal(100)
    return Applied(total=subtotal - saved, saved=saved)
```

The hardcoded `10` is correct DTTC. Nothing yet demands otherwise. Resist
writing the general case — the next test will demand it, and *that* is what
tells you the general case is the right one.

### Iteration 2 — expiry

```python
def test_expired_code_is_rejected() -> None:
    outcome = apply_discount(Decimal(200), TEN_PERCENT, on=date(2027, 1, 1))
    assert outcome == Rejected(reason="expired")
```

```python
    if on > code.expires_on:
        return Rejected(reason="expired")
```

### Iteration 3 — the minimum

```python
def test_code_below_its_minimum_is_rejected() -> None:
    code = DiscountCode(
        code="BIG20",
        benefit=AmountOff(amount=Decimal(20)),
        expires_on=date(2026, 12, 31),
        minimum_subtotal=Decimal(100),
    )
    outcome = apply_discount(Decimal(99), code, on=date(2026, 6, 1))
    assert outcome == Rejected(reason="below_minimum")
```

```python
    if subtotal < code.minimum_subtotal:
        return Rejected(reason="below_minimum")
```

### Iteration 4 — the fixed amount, and the floor

```python
def test_amount_off_never_pushes_the_total_below_zero() -> None:
    code = DiscountCode(
        code="FLAT50",
        benefit=AmountOff(amount=Decimal(50)),
        expires_on=date(2026, 12, 31),
        minimum_subtotal=Decimal(0),
    )
    outcome = apply_discount(Decimal(30), code, on=date(2026, 6, 1))
    assert outcome == Applied(total=Decimal(0), saved=Decimal(30))
```

This is the test that finally demands the general case:

```python
    match code.benefit:
        case PercentageOff(percent):
            saved = subtotal * percent / Decimal(100)
        case AmountOff(amount):
            saved = min(amount, subtotal)

    return Applied(total=subtotal - saved, saved=saved)
```

### The result

```python
def apply_discount(subtotal: Decimal, code: DiscountCode, on: date) -> Outcome:
    if on > code.expires_on:
        return Rejected(reason="expired")
    if subtotal < code.minimum_subtotal:
        return Rejected(reason="below_minimum")

    match code.benefit:
        case PercentageOff(percent):
            saved = subtotal * percent / Decimal(100)
        case AmountOff(amount):
            saved = min(amount, subtotal)

    return Applied(total=subtotal - saved, saved=saved)
```

Four tests green, `mypy --strict` clean. Every line of that function exists
because a test demanded it.

---

## The Adjustment Hierarchy, in practice

New requirement: **a percentage discount never saves more than 50.**

Do not start at the top. **Enter as low as possible** — ask where this rule
actually lives:

| Could it be satisfied here? | |
|---|---|
| Code alone | No — this is new *behavior*, it needs a test |
| Tests → Code | **Yes.** Stop here. |
| Types → Tests → Code | Not needed — no shape changes |
| Design → … | Not needed |

Entry point is Tests. The types do not move:

```python
def test_percentage_saving_is_capped() -> None:
    outcome = apply_discount(Decimal(1000), TEN_PERCENT, on=date(2026, 6, 1))
    assert outcome == Applied(total=Decimal(950), saved=Decimal(50))
```

```python
MAX_PERCENTAGE_SAVING = Decimal(50)

# ...
        case PercentageOff(percent):
            saved = min(subtotal * percent / Decimal(100), MAX_PERCENTAGE_SAVING)
```

Five tests green. `Outcome`, `DiscountCode` and the signature never moved — so
no caller anywhere had to change. That is the hierarchy paying for itself:
*satisfy the top by modifying the bottom.*

---

## Running it

```bash
pip install pytest mypy
pytest
mypy --strict .
```

Both must pass. Under DTTC, a red typecheck and a red test are the same thing:
the step is not done.

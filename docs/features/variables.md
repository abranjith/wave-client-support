# Variables

Wave Client resolves `{{...}}` placeholders anywhere in a request — the URL, query params, headers, and body — at the moment a request is built. There are two kinds:

1. **Reference variables** — `{{name}}` pulls a value from your [environments](environments.md) (and global values).
2. **Dynamic function variables** — `{{_fn_...}}` generates a value on the fly (UUIDs, random numbers, dates, fake data, …).

![A `{{variable}}` used in the URL with a resolved-value tooltip](../images/variables-in-url.png)

---

## Reference variables

Write `{{variableName}}` to insert the value of an environment or global variable. Resolution uses the selected environment first, then global values. If a placeholder can't be resolved, it's left in place and flagged as unresolved so you can spot it before sending.

See [Environments](environments.md) for defining and selecting variables.

---

## Dynamic function variables (`_fn_`)

Placeholders that start with the reserved prefix `_fn_` are evaluated as **functions** at request‑build time, reusing the same `{{...}}` syntax.

```text
{{_fn_random_uuid}}
{{_fn_random_number(min=1, max=100)}}
{{_fn_current_date(format=YYYY-MM-DD)}}
```

**Rules**
- `_fn_` is reserved — placeholders with this prefix are always treated as functions.
- Functions are evaluated per occurrence (no caching), so two `{{_fn_random_uuid}}` produce two different values.
- Named arguments use `key=value` pairs: `_fn_name(key=value, key=value)`.
- Argument values must not contain the parser delimiters `,` `(` `)` `}`.
- Unknown functions or invalid arguments stay unresolved and are reported through the normal unresolved‑placeholder flow.

### Built‑in functions

**Generic**

| Function | Arguments | Description |
| --- | --- | --- |
| `_fn_random_uuid` | none | Random UUID v4 |
| `_fn_random_number` | `min=0`, `max=100`, `decimals=0`, `prefix=`, `suffix=` | Random number with optional formatting |
| `_fn_random_string` | `length=16`, `charset=A-Za-z0-9` | Random string from a charset/ranges |

**Date / Time**

| Function | Arguments | Description |
| --- | --- | --- |
| `_fn_current_date` | `format=YYYY-MM-DD` | Current date |
| `_fn_current_time` | `format=HH:mm:ss` | Current time |
| `_fn_random_date` | `min`, `max`, `format=YYYY-MM-DD` | Random date in range |
| `_fn_random_time` | `min=00:00:00`, `max=23:59:59`, `format=HH:mm:ss` | Random time in range |

**Person**

| Function | Arguments | Description |
| --- | --- | --- |
| `_fn_random_name` | `type=person`, `parts=2` | Random person/organization name |
| `_fn_random_name_prefix` | `values=` | Prefix from locale pool or `|`‑separated override list |
| `_fn_random_name_suffix` | `values=` | Suffix from locale pool or `|`‑separated override list |

**Address**

| Function | Arguments | Description |
| --- | --- | --- |
| `_fn_random_address` | none | One‑line address |
| `_fn_random_address_l1` / `_l2` | none | Address line 1 / line 2 |
| `_fn_random_address_city` | none | City |
| `_fn_random_address_state` | `format=abbr` (`abbr`\|`full`) | State |
| `_fn_random_address_county` | none | County |
| `_fn_random_address_country` | `format=abbr` (`abbr`\|`full`) | Country |
| `_fn_random_address_zip` | none | 5‑digit ZIP |

**Contact**

| Function | Arguments | Description |
| --- | --- | --- |
| `_fn_random_email` | `domain=` | Random email (locale domains when omitted) |
| `_fn_random_phone` | `format=(###) ###-####` | Replaces each `#` with a random digit |
| `_fn_random_ssn` | `format=###-##-####` | Replaces each `#` with a random digit |

### Date/time format tokens

`YYYY` / `YY` (year), `MMM` / `MM` / `M` (month), `DD` / `D` (day), `HH` / `H` (hour, 24h), `mm` / `m` (minute), `ss` / `s` (second).

> The function catalog is maintained alongside the code at [`packages/core/src/utils/functions/README.md`](../../packages/core/src/utils/functions/README.md), which is the authoritative reference and includes additional examples.

---

## Request placeholders (temporary per‑request overrides)

The request editor has a **Placeholders** tab that lets you inspect and **temporarily override** the `{{variable}}` placeholders used anywhere in the current request — URL (including query params), headers, and body — without touching your environments.

Use it to:

1. **Try throwaway values quickly.** Set a value for a variable just for this request and re‑send with different values, instead of editing the request inline or repeatedly changing environment variables.
2. **Preview what will actually be sent.** Per variable, see the value that will be substituted: your override first, then the environment value, then unresolved.

How it behaves:

- **Temporary & per‑tab.** Overrides live in memory for that tab (they survive tab switches) but are **never saved** to the request or to history.
- **Takes precedence over environment variables** — without changing them. A variable that has an override renders as **resolved (green)** in the URL, params, headers, and body inputs, and the sent request uses the override value.
- **Case‑insensitive.** An override named `token` beats an environment value named `Token`.
- **Function variables are excluded.** System/function placeholders (`{{_fn_…}}`) have runtime meaning and are not overridable.
- **The tab appears only when there's something to override** — if the request uses no `{{variables}}`, the Placeholders tab is hidden.
- A single **hide/unhide** toggle masks values that may be secrets, and **Clear** removes all overrides for the tab.

> **Auth values are out of scope.** Auth configuration may use `{{variables}}` too, but those are intentionally **not** overridden from the Placeholders tab — set auth values through [Auth](auth.md) instead.

---

## Related guides
- [Environments](environments.md) — define and select reference variables
- [Requests](requests.md) — where variables are used
- [Flows](flows.md) — pass data between requests using flow variable references

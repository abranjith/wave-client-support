# Test Suite Schema Reference

> **Formal definition**: `packages/core/src/schemas/testSuiteSchema.ts`  
> **Types**: `packages/core/src/types/testSuite.ts`  
> **Schema version**: 1 (added in wave-client v0.x)

This document describes the Wave test suite file format, covering every field, the
override semantics used during test execution, and a practical authoring guide for
developers (and AI agents) writing test suites programmatically.

---

## Table of Contents

1. [Overview](#overview)
2. [Override Semantics](#override-semantics)
3. [Schema Reference](#schema-reference)
   - [TestSuite](#testsuite)
   - [TestItem](#testitem)
   - [TestCase](#testcase)
   - [TestCaseData](#testcasedata)
   - [CollectionBody](#collectionbody)
   - [RequestValidation](#requestvalidation)
   - [ValidationRuleRef](#validationruleref)
   - [TestSuiteSettings](#testsuitesettings)
4. [Annotated Example](#annotated-example)
5. [Authoring Tips for AI Agents](#authoring-tips-for-ai-agents)

---

## Overview

A **test suite** is a JSON file (`*.json`) stored in the `test-suites/` directory. It
contains a list of *test items* — each item references a collection request or flow —
and each item can have multiple *test cases* that drive data-driven execution.

At run time the test runner:

1. Iterates through enabled items in order.
2. For each item, runs once per enabled test case (or once with no overrides if no
   test cases are defined).
3. Applies the test case's data overrides on top of the base request.
4. Validates the response using the test case's validation rules (or falls back to
   item-level, then request-level, then the default rule set).

---

## Override Semantics

| Override field            | Merge strategy | Notes                                                                  |
|--------------------------|----------------|------------------------------------------------------------------------|
| `TestCaseData.variables` | **Merge**      | Overrides env vars by name; new names are added to the effective env   |
| `TestCaseData.headers`   | **Merge**      | Case-insensitive key match; test case wins; new keys are appended      |
| `TestCaseData.params`    | **Merge**      | Same as headers; new query params are appended                         |
| `TestCaseData.body`      | **Replace**    | The entire base request body is replaced                               |
| `TestCaseData.authId`    | **Replace**    | The entire auth profile is swapped                                     |
| `TestCase.validation`    | **Replace**    | When present, the request's own rules are ignored for this test case   |

**Presence = override intent.** If a field is `undefined` / absent, the base value
is used unchanged. Save bandwidth: only include fields you actually want to override.

**Validation precedence** (first match wins):
```
testCase.validation → testItem.validation → request.validation → DEFAULT_VALIDATION
```

---

## Schema Reference

### TestSuite

Top-level object stored to disk.

| Field          | Type                  | Required | Description                                             |
|----------------|-----------------------|----------|---------------------------------------------------------|
| `id`           | `string`              | ✅        | Unique suite identifier (used as filename)              |
| `name`         | `string` (min 1)      | ✅        | Display name — must be unique across suites             |
| `description`  | `string`              |          | Optional free-text description                          |
| `version`      | `number` (int)        |          | Schema version sentinel. Absent = pre-v1 (treat as 1)  |
| `items`        | `TestItem[]`          | ✅        | Ordered list of test items                              |
| `defaultEnvId` | `string`              |          | Default environment for variable resolution             |
| `defaultAuthId`| `string`              |          | Default auth profile for all items                      |
| `settings`     | `TestSuiteSettings`   | ✅        | Run configuration                                       |
| `createdAt`    | `string` (ISO 8601)   | ✅        | Creation timestamp                                      |
| `updatedAt`    | `string` (ISO 8601)   | ✅        | Last modification timestamp                             |

---

### TestItem

Each item is a discriminated union on the `type` field.

#### RequestTestItem (`type: "request"`)

| Field         | Type               | Required | Description                                                |
|---------------|--------------------|----------|------------------------------------------------------------|
| `id`          | `string`           | ✅        | Unique within the suite                                    |
| `type`        | `"request"`        | ✅        | Discriminant                                               |
| `name`        | `string`           | ✅        | Display name (defaults to referenced request name)         |
| `referenceId` | `string`           | ✅        | `"collectionFile.json:requestId"` or just `"requestId"`    |
| `order`       | `number` (int)     | ✅        | Execution order (0-indexed)                                |
| `enabled`     | `boolean`          | ✅        | Whether this item runs                                     |
| `validation`  | `RequestValidation`|          | Item-level validation (overrides request's rules)          |
| `testCases`   | `TestCase[]`       |          | If absent/empty, request runs once with no overrides       |

#### FlowTestItem (`type: "flow"`)

Same as `RequestTestItem` but with `type: "flow"`. No `validation` field.
Only `TestCaseData.variables` is used for flow test cases — other override
fields are ignored (a flow spans multiple requests).

---

### TestCase

| Field         | Type               | Required | Description                                        |
|---------------|--------------------|----------|----------------------------------------------------|
| `id`          | `string`           | ✅        | Unique within the parent item                      |
| `name`        | `string` (min 1)   | ✅        | Scenario name (e.g., `"Valid credentials"`)        |
| `description` | `string`           |          | What this case tests                               |
| `enabled`     | `boolean`          | ✅        | Whether this case runs                             |
| `order`       | `number` (int)     | ✅        | Execution order within the item                    |
| `data`        | `TestCaseData`     | ✅        | Data overrides for this case                       |
| `validation`  | `RequestValidation`|          | Case-level validation; **replaces** item/request   |

---

### TestCaseData

All fields are optional. An empty object `{}` means "no overrides".

| Field       | Type                   | Override | Description                                           |
|-------------|------------------------|----------|-------------------------------------------------------|
| `variables` | `Record<string,string>`| Merge    | Env-var overrides. Keys map to `{{variable}}` names   |
| `headers`   | `HeaderRow[]`          | Merge    | Header overrides merged with base request headers     |
| `params`    | `ParamRow[]`           | Merge    | Query param overrides merged with base request params |
| `body`      | `CollectionBody`       | Replace  | Replaces the entire base request body                 |
| `authId`    | `string`               | Replace  | Auth profile ID — replaces item/suite default         |

**HeaderRow / ParamRow schema:**
```json
{ "id": "optional-uuid", "key": "Header-Name", "value": "header-value", "disabled": false }
```

---

### CollectionBody

Discriminated union on `mode`:

| Mode          | Additional fields                                   |
|---------------|-----------------------------------------------------|
| `"none"`      | _(no content)_                                      |
| `"raw"`       | `raw: string`, optional `options.raw.language`      |
| `"urlencoded"`| `urlencoded: FormField[]`                           |
| `"formdata"`  | `formdata: MultiPartFormField[]`                    |
| `"file"`      | `file?: FileReference`                              |

**Raw language values:** `"json" | "xml" | "html" | "text" | "csv"`

**FileReference** (for `file` body and multipart file fields):
```json
{
  "path": "",
  "fileName": "payload.json",
  "contentType": "application/json",
  "size": 1024,
  "pathType": "browser",
  "storageType": "local",
  "fileData": "<base64-encoded content>"
}
```
> ⚠️ **File size caveat**: `fileData` stores the entire file content as base64 inline
> in the test suite JSON. Keep file payloads small (< 1 MB per test case recommended)
> to avoid bloating the suite file.

---

### RequestValidation

```json
{
  "enabled": true,
  "rules": [ /* ValidationRuleRef[] */ ]
}
```

When `TestCase.validation` is present and `enabled: true`, only its rules run for
that test case — the request's own rules are skipped entirely.

---

### ValidationRuleRef

Each entry in `rules[]` is one of:

**Global rule reference** (by ID from the global rule store):
```json
{ "ruleId": "some-global-rule-id" }
```

**Inline rule** (self-contained, only used for this test case):
```json
{
  "rule": {
    "id": "unique-id",
    "name": "Status is 200",
    "enabled": true,
    "category": "status",
    "operator": "equals",
    "value": 200
  }
}
```

Supported `category` values: `"status" | "header" | "body" | "time"`.

---

### TestSuiteSettings

| Field               | Type      | Default | Description                              |
|---------------------|-----------|---------|------------------------------------------|
| `concurrentCalls`   | `number`  | `1`     | Max concurrent items during a run        |
| `delayBetweenCalls` | `number`  | `0`     | Milliseconds between batches             |
| `stopOnFailure`     | `boolean` | `false` | Stop the entire run on the first failure |

---

## Annotated Example

```json
{
  "id": "suite-login-scenarios",
  "name": "Login Scenarios",
  "version": 1,
  "description": "Tests for the /auth/login endpoint across multiple credential scenarios",
  "defaultEnvId": "env-staging",
  "settings": {
    "concurrentCalls": 1,
    "delayBetweenCalls": 200,
    "stopOnFailure": false
  },
  "items": [
    {
      "id": "item-login",
      "name": "POST /auth/login",
      "type": "request",
      "referenceId": "auth-collection.json:req-login",
      "order": 0,
      "enabled": true,
      "testCases": [
        {
          "id": "tc-valid",
          "name": "Valid credentials",
          "description": "Should return 200 + JWT token",
          "enabled": true,
          "order": 0,
          "data": {
            "variables": {
              "username": "alice@example.com",
              "password": "correct-password"
            },
            "body": {
              "mode": "raw",
              "raw": "{\"email\":\"{{username}}\",\"password\":\"{{password}}\"}",
              "options": { "raw": { "language": "json" } }
            }
          },
          "validation": {
            "enabled": true,
            "rules": [
              { "ruleId": "global-status-200" },
              {
                "rule": {
                  "id": "tc-body-has-token",
                  "name": "Body contains token",
                  "enabled": true,
                  "category": "body",
                  "operator": "json_path_exists",
                  "value": "$.token"
                }
              }
            ]
          }
        },
        {
          "id": "tc-wrong-password",
          "name": "Wrong password",
          "description": "Should return 401",
          "enabled": true,
          "order": 1,
          "data": {
            "variables": {
              "username": "alice@example.com",
              "password": "wrong-password"
            }
          },
          "validation": {
            "enabled": true,
            "rules": [
              {
                "rule": {
                  "id": "tc-status-401",
                  "name": "Status is 401",
                  "enabled": true,
                  "category": "status",
                  "operator": "equals",
                  "value": 401
                }
              }
            ]
          }
        }
      ]
    }
  ],
  "createdAt": "2026-06-16T00:00:00.000Z",
  "updatedAt": "2026-06-16T00:00:00.000Z"
}
```

---

## Authoring Tips for AI Agents

When generating test suites programmatically, follow these rules to produce valid,
runnable suites:

1. **Always include required fields.** `id`, `name`, `settings`, `createdAt`,
   `updatedAt` at the suite level; `id`, `name`, `type`, `referenceId`, `order`,
   `enabled` at the item level; `id`, `name`, `enabled`, `order`, `data` at the
   test case level.

2. **Generate unique IDs.** Use `crypto.randomUUID()` or a similar UUID generator.
   Collisions break the UI.

3. **Use `"version": 1`** in newly created suites.

4. **Omit fields you don't need.** Absent optional fields (e.g., no `variables`,
   no `validation`) mean "inherit from the request". Do not include empty objects
   or arrays when you intend "no override" — an empty `variables: {}` will send an
   empty override; omitting `variables` entirely is cleaner.

5. **Use `{{variable}}` syntax for parameterized values.** Variables in the
   `variables` map shadow the environment's values for that test case. Prefer
   using `{{env_var}}` syntax in the request body/headers and providing the
   per-test override in `data.variables`, rather than hardcoding values in the
   body JSON string.

6. **File bodies are base64-inline.** If a test case needs a file body, base64-
   encode the file content into `fileData`. Keep files small; large files bloat
   the test suite JSON.

7. **Validation replace semantics.** If you add `validation` to a test case, its
   rules *replace* the request's rules entirely for that case. To augment rather
   than replace, you must repeat the request's rules in the test case's rules list.

8. **Global vs inline rules.** Use `{ "ruleId": "..." }` to reference a rule from
   the global rule store. Use `{ "rule": { ... } }` for one-off inline rules that
   only apply to this test case.

9. **Order items and test cases sequentially.** Use `order: 0, 1, 2, ...` starting
   from 0. The UI sorts by `order`, so gaps cause unexpected display ordering.

10. **Set `enabled: true`** for items and test cases you want to run. Disabled ones
    are shown in the UI but skipped during execution.

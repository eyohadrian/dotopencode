---
name: dbt-unitt
description: Generate unit test for dbt data models
---
# dbt Unit Test Generator Skill

## Purpose

Generate and maintain dbt unit tests for changed SQL models under `dbt/models/`. The skill should inspect changed `*.sql` files, infer the relevant use cases in each model, add atomic dbt unit tests to the model's companion YAML file, and ensure the SQL model is tagged with `unitt`.

## Scope

Use this skill when the user asks to create, update, review, or enforce dbt unit tests for models in a dbt project.

Target files:

- SQL models: `dbt/models/**/*.sql`
- Companion YAML files: normally the YAML file with the same basename as the SQL model:
  - `dbt/models/path/to/x.sql`
  - `dbt/models/path/to/x.yml`

If the companion YAML does not exist, create it next to the SQL model.

## Core behavior

For each changed SQL model under `dbt/models/`:

1. Detect the changed SQL model.
2. Locate the companion YAML file with the same basename.
3. Parse the SQL model to identify independent business/use cases.
4. Generate dbt unit tests in the YAML file.
5. Keep each unit test atomic: one test should validate one specific behavior or edge case.
6. Ensure the SQL model has the dbt tag `unitt`.
7. Preserve existing model config, YAML metadata, descriptions, columns, and tests unless an update is required.

## Changed file detection

Prefer detecting changed SQL files using git:

```bash
git diff --name-only --diff-filter=ACMRT HEAD -- 'dbt/models/**/*.sql'
```

Also include staged files when relevant:

```bash
git diff --cached --name-only --diff-filter=ACMRT -- 'dbt/models/**/*.sql'
```

If the user provides a specific file, only process that file unless they ask for all changed files.

Ignore files outside `dbt/models/`.

## Companion YAML resolution

For a SQL model:

```text
dbt/models/some/path/my_model.sql
```

Use this YAML file by default:

```text
dbt/models/some/path/my_model.yml
```

If a model is already documented in another YAML file, prefer the existing YAML location only if it clearly contains the matching model entry. Otherwise, create or update the homonymous YAML file.

## Tag enforcement

Every processed SQL model must have the `unitt` tag.

### Preferred dbt config style

If the SQL model already has a `config()` block, add `unitt` to the existing `tags` config without removing other tags.

Example:

```sql
{{ config(
    materialized='table',
    tags=['daily', 'unitt']
) }}
```

If there is no `config()` block, add one at the top of the file:

```sql
{{ config(tags=['unitt']) }}
```

Rules:

- Do not duplicate `unitt` if it already exists.
- Preserve existing tags.
- Preserve formatting as much as practical.
- If tags are defined as a string, convert safely to a list only when needed:
  - `tags='daily'` -> `tags=['daily', 'unitt']`
- If tags are inherited from project-level config and not visible in the model SQL, still add the explicit SQL-level tag unless the user says not to.

## Unit test principles

Unit tests must be atomic to each SQL use case.

A use case is a distinct behavior in the SQL model, such as:

- A `case when` branch.
- A filter condition.
- A join match.
- A join miss / null-side behavior.
- A deduplication rule.
- A window function selection rule.
- An aggregation rule.
- A type casting or normalization rule.
- A default/fallback value.
- A date boundary condition.
- An array or struct construction rule.
- A union branch.
- A surrogate key or ID construction rule.

Do not combine multiple independent behaviors in the same unit test unless the model cannot be evaluated otherwise.

## dbt unit test format

### Default fixture format

Use `format: dict` by default for unit test inputs, including in BigQuery projects.

Reason:
- `format: dict` is the simplest and most readable default for most unit tests.
- It keeps fixtures concise for common flat inputs and expected outputs.
- It is easier to maintain when testing straightforward model behavior.

Use `format: sql` only when:
- Inputs require nested fields such as `STRUCT` or `ARRAY`
- Precise SQL typing or casting is required
- The fixture would be clearer as a query than as YAML rows
- `format: dict` would cause compilation issues for the specific case

Use dbt native unit tests in YAML.

Default structure:

```yaml
unit_tests:
  - name: test_<model_name>__<behavior>
    model: <model_name>
    given:
      - input: ref('<upstream_model>')
        rows:
          - {column_a: <value>, column_b: <value>}
    expect:
      rows:
        - {output_column: expected_value}
```

SQL fixture structure when needed:
```yaml
unit_tests:
  - name: test_<model_name>__<behavior>
    model: <model_name>
    given:
      - input: ref('<upstream_model>')
        format: sql
        rows: |
          SELECT
            <value> AS column_a,
            <value> AS column_b
          UNION ALL
          SELECT
            <value> AS column_c,
            <value> AS column_d

    expect:
      rows:
        - {output_column: expected_value}
```

For sources, use:

```yaml
given:
  - input: source('<source_name>', '<table_name>')
    rows:
      - {column_a: value}
```

Name tests using this pattern:

```text
test_<model_name>__<specific_behavior>
```

Examples:

```text
test_fct_orders__keeps_completed_orders
test_fct_orders__excludes_cancelled_orders
test_dim_users__defaults_missing_country_to_unknown
test_int_sessions__deduplicates_by_latest_event_timestamp
```

## YAML placement rules

If the companion YAML already contains a top-level `unit_tests:` section, append new tests there.

If the YAML contains only `models:`, add `unit_tests:` as a top-level sibling:

```yaml
version: 2

models:
  - name: my_model
    description: ...

unit_tests:
  - name: test_my_model__some_behavior
    model: my_model
    given: ...
    expect: ...
```

If creating a new YAML file, use:

```yaml
version: 2

models:
  - name: <model_name>
    description: ""

unit_tests:
  - name: test_<model_name>__<behavior>
    model: <model_name>
    given: []
    expect:
      rows: []
```

Replace placeholder tests with real tests before finalizing.

## Test generation workflow

For each SQL model:

1. Read the SQL.
2. Identify model name from the filename.
3. Identify dependencies:
   - `ref('...')`
   - `source('...', '...')`
4. Identify output columns.
5. Identify logic branches and edge cases.
6. Build minimal input rows for each dependency.
7. Build expected output rows.
8. Add one unit test per behavior.
9. Avoid over-testing dbt, database syntax, or upstream models.
10. Ensure tests are deterministic.

## Atomicity rules

Each unit test should normally include:

- The minimum number of input rows needed.
- The minimum number of input columns needed.
- Exactly one behavior being validated.
- A clear name describing the behavior.

Bad example:

```text
test_model__all_logic
```

Good examples:

```text
test_model__filters_out_inactive_rows
test_model__uses_default_status_when_status_is_null
test_model__selects_latest_record_per_user
test_model__keeps_left_join_rows_without_match
```

## Handling joins

For joins, create separate tests for:

- Matching rows.
- Missing right-side rows for left joins.
- Multiple matches if the model logic handles them.
- Null join keys if relevant.

## Handling filters

For filters, create separate tests for:

- Rows that should pass.
- Rows that should be excluded.
- Boundary values for dates, numbers, or flags.

## Handling `case when`

For `case when` expressions, create one test for each meaningful branch:

- Each explicit `when` branch.
- The `else` branch.
- Null input behavior if it can alter the result.

## Handling aggregations

For aggregations, create separate tests for:

- Basic aggregation result.
- Group separation.
- Null handling.
- Duplicate handling when relevant.

## Handling window functions

For window functions, create separate tests for:

- The selected row.
- Tie-breaking behavior.
- Partition separation.

## Existing tests

Before adding a new test:

1. Check existing `unit_tests` for the same model.
2. Do not duplicate an equivalent test.
3. Update an existing test only if it clearly targets the same behavior and is incomplete or wrong.
4. Preserve manual tests unless they conflict with the SQL logic.

## Validation

Since the project uses custom auth dbt execution permissions, there is no need to execute the tests validation. It's up to the user to do this task.

## Output summary

When finished, report:

- SQL models processed.
- YAML files created or updated.
- Unit tests added or updated.
- Whether the `unitt` tag was added or already present.
- Any assumptions or tests that could not be generated confidently.

## Guardrails

- Do not invent columns that are not required by the SQL logic.
- Do not rewrite the model SQL except to add or update the `unitt` tag.
- Do not remove existing YAML content.
- Do not add broad integration-style tests under `unit_tests`.
- Do not create one giant test for all model behavior.
- Do not add tests for unchanged SQL files unless the user explicitly asks.
- If SQL logic is ambiguous, generate the safest minimal tests and clearly mention the assumption.

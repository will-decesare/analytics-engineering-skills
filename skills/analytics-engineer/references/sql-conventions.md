# SQL Conventions

## Resolve local style first

Inspect formatter and linter configuration, neighboring models, contribution
guides, and the target dialect. Follow those rules exactly. When the repository
is silent, use these portable defaults:

- use lowercase keywords, functions, and unquoted identifiers;
- indent with four spaces and keep lines near 80 characters;
- use leading commas for multi-line select lists;
- use explicit table aliases and explicit aliases for expressions;
- put spaces around operators and blank lines between major clauses;
- use CTEs instead of non-trivial subqueries in `from` or `join` positions;
- use the repository's established positional or named `group by` style.

Do not reformat untouched legacy or retired files merely to impose these
defaults.

## Structure transformations

- In dbt, declare `ref()` and `source()` relations in clearly named CTEs near
  the top when that is the local pattern.
- Name each CTE for its relation, entity, or transformation purpose.
- Give each logical step one job: source, clean, aggregate, enrich, calculate,
  or assemble.
- Keep final CTEs limited to assembly, aliases, column order, and projection.
- Reuse an existing source CTE rather than calling the same relation twice.
- Remove unused CTEs and columns that do not serve the output or an intermediate
  calculation.

## Write safe joins

- Qualify ambiguous columns and make join intent readable.
- Prepare normalized keys in an upstream CTE. Do not cast, trim, lower, hash, or
  coalesce keys inside the `on` clause unless the repository explicitly favors
  that pattern.
- Reduce the many side to the required grain before joining.
- Express temporal joins with explicit inclusive/exclusive boundaries.
- Decide whether unmatched rows should be retained before choosing inner or
  outer join semantics.

## Write reliable calculations

- Make integer, decimal, and timestamp types deliberate for the target dialect.
- Guard division with a dialect-appropriate zero/null strategy.
- State null handling rather than relying on accidental aggregate behavior.
- Use window functions for ordered business questions, not as a default
  deduplication tool.
- Prefer conditional aggregation to repeated scans when it remains readable.
- Avoid `select *` in durable consumer contracts unless a controlled staging
  convention requires it.

## Name contracts consistently

Defer to repository rules. In their absence:

- use plural nouns for relation names representing collections;
- use `_id` for identifiers, `_count` for counts, `_amount` for currency-like
  numerics, `_date` for dates, and `_at` for timestamps;
- use `is_` or `has_` prefixes for booleans;
- choose fact, dimension, intermediate, staging, and application prefixes only
  if the project already uses a layered naming system;
- order output columns predictably: keys, descriptors, measures, booleans,
  dates, then timestamps.

## Document intent durably

- Use SQL comments for local implementation constraints that cannot be made
  obvious in code.
- Put business definitions, caveats, ownership, and consumer guidance in model
  or column documentation.
- Explain any necessary `distinct`, fan-out, source cutoff, or window-based
  record selection in durable documentation and tests.

# Audits
Audits are one of the tools SQLMesh provides to validate your data. Along with `sqlmesh.core.test`,
audits are a great way to ensure the quality of your data and to build trust in your data across your organization.
A comprehensive suite of audits can identify data issues upstream, whether they are from your vendors or other teams.
Audits also empower your data engineers and analysts to work with confidence by catching problems early as they work on new features or make updates to your models.

## What exactly are audits?
Audits are SQL queries that should not return any rows. In other words, they query for bad data, which would indicate something is wrong. In its simplest form, an audit is defined with the custom AUDIT expression along with a query as in the following example:

```sql
AUDIT (
  name assert_item_price_is_not_null,
  model sushi.items,
  dialect spark,
)
SELECT * from sushi.items
WHERE ds BETWEEN @start_ds AND @end_ds AND
   price IS NULL
```

In the  example, we defined an audit named `assert_item_price_is_not_null` on the model `sushi.items` to ensure that every sushi item has a price. If the query is in a different dialect than the rest of your project, you can specify it here as we did in the example, and SQLGlot will automatically know how to execute the query. While the query can technically be on any model, or even multiple models, the model specified in the audit definition tells SQLMesh when to run the audit during your pipeline's execution. If the query returns any records, it means there may be an issue that requires your attention.

Audits are defined in `.sql` files in an `audit` directory in your SQLMesh project. Multiple audits can be defined in a single file, so you can organize them to your liking.

## Running audits

### The CLI audit command

You can execute audits with the `sqlmesh audit` command, as in the following example:
```
% sqlmesh --path project audit -start 2022-01-01 -end 2022-01-02
Found 1 audit(s).
assert_item_price_is_not_null FAIL.

Finished with 1 audit error(s).

Failure in audit assert_item_price_is_not_null for model sushi.items (audits/items.sql).
Got 3 results, expected 0.
SELECT * FROM sqlmesh.sushi__items__1836721418_83893210 WHERE ds BETWEEN '2022-01-01' AND '2022-01-02' AND price IS NULL
Done.
```

### Automated auditing
When you apply a plan, SQLMesh will automatically run each model's audits. By default, SQLMesh will halt the pipeline when an audit fails in order to prevent potentially invalid data from propagating further downstream. This behvavior can be changed for individual audits. Refer to [Non-blocking audits](#non-blocking-audits).

## Advanced usage

### Skipping audits

Audits can be skipped by setting the skip argument to true as in the following example:

```sql
AUDIT (
  name assert_item_price_is_not_null,
  model sushi.items,
  skip true
)
SELECT * from sushi.items
WHERE ds BETWEEN @start_ds AND @end_ds AND
   price IS NULL
```

### Non-blocking audits

By default, audits that fail will stop the execution of the pipeline in order to prevent bad data from propagating further downstream. An audit can be configured to notify you when it fails without blocking the execution of the pipeline, as in the following example:

```sql
AUDIT (
  name assert_item_price_is_not_null,
  model sushi.items,
  blocking false
)
SELECT * from sushi.items
WHERE ds BETWEEN @start_ds AND @end_ds AND
   price IS NULL
```
# Flow design

## Flow 1: Value Assurance (18 actions)

1. **Trigger:** a file is created or modified in the SharePoint library that holds the value calculators.
2. **Parse:** an Office Script reads the locked template and returns clean JSON: project details plus one record per benefit. Text is trimmed and normalised.
3. **Find:** query the target board for an existing project group and item. Matching is case-insensitive and ignores extra whitespace, which prevents duplicates when someone retypes a project name.
4. **Upsert:** create the group and item if missing, otherwise update the mapped columns.
5. **Subitems:** for each benefit, update the matching subitem or create a new one.
6. **Log:** record the outcome for the runbook's troubleshooting section.

## Flow 2: Monthly Snapshots (9 actions)

Runs on a monthly schedule and writes a snapshot of key values to a history board, which gives trend reporting without keeping old files.

## Idempotency

Running the flow twice on the same file must not create duplicates. Every write is preceded by a lookup on a normalised key, and every create path has a matching update path. This made it safe to re-run after failures, which mattered during handover.

## Column mapping

17 columns across 4 boards were mapped from template cells to monday column IDs, with the value format each column type expects (text, numbers, status labels, dates). The mapping is documented in the runbook so a future maintainer can change a column without reading the flow.

## Known limitation

A portfolio health view that depended on monday formula columns was not shipped, because those columns cannot be read through the API. Shipping it would have meant showing blank or wrong values.

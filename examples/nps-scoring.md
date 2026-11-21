# NPS scoring as flow expressions

monday.com formula columns return null through the API, which means scores had to be computed before writing. Generic version of the logic (not company code):

## Classify one response (0 to 10)

```
if(greaterOrEquals(int(item()?['score']), 9), 'Promoter',
   if(greaterOrEquals(int(item()?['score']), 7), 'Passive', 'Detractor'))
```

## Score for a project

```
NPS = 100 * (promoters - detractors) / total responses
```

As an expression, after counting each group with a Filter array action and `length(body('Filter_array'))`:

```
mul(100, div(sub(float(variables('promoters')), float(variables('detractors'))), float(variables('total'))))
```

## Notes

- Guard against `total = 0` before dividing.
- Store the category on each response and the score on the project, which keeps the board readable without formulas.

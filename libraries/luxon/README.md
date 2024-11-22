## Luxon

### Quick References

- https://moment.github.io/luxon/demo/global.html
- https://moment.github.io/luxon/#/formatting?id=presets
- https://moment.github.io/luxon/#/formatting?id=table-of-tokens
- https://momentjscom.readthedocs.io/en/latest/moment/04-displaying/01-format/

### Calculate duration between to dates

```typescript
const date1 = luxon.DateTime.fromISO("2020-09-06T12:00");
const date2 = luxon.DateTime.fromISO("2019-06-10T14:00");

const diff = date1.diff(date2, ["years", "months", "days", "hours"]);

console.log(diff.toObject());
```

https://stackoverflow.com/a/63763404/10415969

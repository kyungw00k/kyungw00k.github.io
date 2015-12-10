# Log Rotate using Curator

```
curator --host localhost delete indices --older-than 30 --time-unit days --timestring '%Y.%m.%d'
```
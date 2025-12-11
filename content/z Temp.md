```
SELECT CAST(BulkColumn AS VARCHAR(MAX)) as FileContent FROM OPENROWSET(BULK 'C:\temp\sample.txt', SINGLE_BLOB) AS x;
```

```

```
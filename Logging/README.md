```

requests
| where name == 'Calculator'
| summarize count() by bin(timestamp,1m), tostring(success)
| render barchart 


requests
| where name == 'Calculator'
| summarize count() by tostring(success)
| render piechart 



```
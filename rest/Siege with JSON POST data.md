```
# Changed to use content-type flag instead of header: -H 'Content-Type: application/json'
siege -c50 -t60S --content-type "application/json" 'http://domain.com/path/to/json.php POST {"ids": ["1","2","3"]}'
```

If you were stuck like me with no JSON being read at server-side, then use --content-type "application/json" instead of the header.

https://gist.github.com/MikeNGarrett/f34531570625973c4cac
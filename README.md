# how-i-build
How I build software. For explanation to others, and to remind myself.

# Server

```
var http = require('http');
var server = http.createServer( /*...routing logic...*/ );
server.listen(config.port || 8080);
```

## Routing/serving

### Modules
 - Routing: sea-lion
 - File serving: dion
 - svg -> png server/scaler/cacher: idol
 
```
...
var SeaLion = require('sea-lion');
var router = new SeaLion();
var Dion = require('dion');
var fileServer = new Dion(router);

router.add({
    '/': serveHomePage,
    '/`path...`': fileServer.serveDirectory('./public', {
        '.js': 'application/javascript'
        //...etc...
    })
});

var server = http.createServer(router.createHandler());

//...
```

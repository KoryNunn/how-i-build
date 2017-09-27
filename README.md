# how-i-build
How I build software. For explanation to others, and to remind myself.

# Synopsis

This is not dogma, all opinions and methods are subject to change based on experiance and evidence. This *is* opinion, since I am a human.

AKA: PULL REQUESTS WELCOME! (I might not agree but it will probably create a good discussion :))

I approach software development from a position that I feel is pragmatic. What I consider to be a pragmatic approach will not necisarily align with what others feel.

# TL;DR's

1. Don't use any dependancy that you couldn't write yourself in a day.* 
2. Skim the source of **every** dependancy you include.
3. Don't use frameworks.
4. Don't transpile.
5. Ship less code.

\* Exceptions are things like DB connections or proper maths/physics. I wouldn't suggest writing your own FFT lib for example other than for fun.

# Server

```javascript
var http = require('http');
var server = http.createServer( /*...routing logic...*/ );
server.listen(config.port || 8080);
```

## Routing/serving

### Modules
 - Routing: sea-lion
 - File serving: dion
 - svg -> png server/scaler/cacher: idol
 
```javascript
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

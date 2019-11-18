# how-i-build
How I build software. For explanation to others, and to remind myself.

## A work in progress.

# Synopsis

This is not dogma, all opinions and methods are subject to change based on experience and evidence. This *is* opinion, since I am a human.

AKA: PULL REQUESTS WELCOME! (I might not agree but it will probably create a good discussion :))

I approach software development from a position that I feel is pragmatic. What I consider to be a pragmatic approach will not necisarily align with what others feel.

# TL;DR's

1. Don't use any dependancy that you couldn't write yourself in a day.*
    - I'm not saying to rewrite them, just be confident you know how they work, and would be able to write your own if something goes wrong.
1. Skim the source of **every** dependancy you include.
1. Don't use frameworks.
1. Don't transpile.
1. Ship less code.

\* Exceptions are things like DB connections or proper maths/physics. I wouldn't suggest writing your own FFT lib for example other than for fun.

# Reading guide

I write modules functionally bottom-to-top, exported functions will be the last to be defined, and will call functions defined above them. In a given block of code, it's probably best to skip to the last function, and read from there.

# Axioms

1. Assumptions increase the chance of bugs, but improve short-term velocity. Aim to reduce.
1. Coupling increases future development effort, and reduces feature agility. Aim to reduce.
1. The fewer responsibilities a module of code has, the better.
1. Empirical > Theoretical.
1. Understanding why and how code works is more important than that it works.

# If you couldn't write it, don't use it.

Assume every depedancy you import will break, or not meet a requirement at some point. *When* it breaks, you will need to either fix it, or replace it. If you assume you'll have to replace something, you automatically avoid coupling and complexity. If you know how you would write it yourself, your assumptions about how it works are much more likely to be correct. This is a fairly daunting suggestion for most, but it shouldn't be, most software isn't as amazing as people believe.

Getting rid of frameworks makes this much easyer to enact. Frameworks are just a coupled set of simple tools. Every UI framework contains an aproximation of the same set of freatures: data -> DOM (or other UI) manipulation, routing, some form of functional structure/pattern, and a sprinkling of unbelievably decouplable tools like an ajax implemntation, async management, and data manipulation tools. Every webserver framework is just a router with frills.

Decomposing frameworks into responsible components reveals them to be substantially less bewildering than they are generally considered to be.

# Server

I build servers up from the least possible components as they are needed. Generally, the core of all of my servers look like this:

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

## Business logic

Mostly, controllers are going to be simple request -> response handlers:

Here's an example of how you might build a simple controller that asynchronously checks a users permissions to a `user`, then loads that user, and returns it:

```javascript

function getUser(session, query, callback){
    var validPermissions = righto(checkPermissions, session, 'getUser', query);
    var user = righto(dataStorage.query, /*...*/ , righto.after(validPermissions));

    user(callback);
}

```

Some key points:
1. There is no error handling.
    - Rejections are bubbled, since there is nothing useful we can do with them.
    - Exceptions crash the process, stoping the server from entering an invalid state. Crashing servers get fixed fast.

Obviously rejections are a normal part of business logic, so they shouldn't be ignored, but this is not the right place to deal with them.

In the above code, there are two places where rejection errors might be produced: In the permissions check, and the dataStorage query.

For creating errors I use [generic-errors](https://www.npmjs.com/package/generic-errors) which leverages the well established error codes from HTTP.

```javascript

// checkPermissions.js

var errors = require('generic-errors');

function checkPermissions(session, action, query, callback){
    if(!session.userId){
        return callback(new errors.Unauthorised('User not loggegd in'));
    }

    if(!/* check if user has permissions to do the action requested */){
        // Generally it's best to return a NotFOund instead of Forbidden, so you don't reveal any more information than you need to.
        return callback(new errors.NotFound());
    }

    callback();
}

```

If an error is produced by the above, it will be passed through the controller layer, to the router, which is the other place where errors get touched.

```javascript

function handleError(error, callback){

    // Check if the error passed is one we created
    if(error instanceof errors.BaseError){
        return callback(error);
    }

    // If we did not create the error ourselves, DO NOT SENT TO THE USER!
    // Log it, and respond with a generic error message.
    // If you want to go hardcore mode, crash the server here.

    logger.log(error);

    callback(new errors.BaseError('An unknown error occured'));
}

function handleReponse(response, error, result){
    if(error){
        response.writeHead(error.code);
        response.end(JSON.stringify(error));
        return;
    }

    if(!result){
        response.end();
        return;
    }

    // Allow for the response of streamed data
    if(result instanceof ReadStream){
        result.pipe(response);
        return;
    }

    response.end(JSON.stringify(result));
}

function createHandler(controller){
    return function(request, response, tokens, data){
        var session = /* Get session somehow */;

        var query = righto(createQuery, tokens, data);

        var result = righto(controller, session, query);

        var complete = righto.handle(result);

        complete(function(error, result){
            handleResponse(response, error, result);
        });
    };
}

module.exports = createHandler;

```


# Client

## SPA's

### Structure/Components

Bindable components: https://www.npmjs.com/package/fastn
Swipeable tabs: https://www.npmjs.com/package/slabs
Swipeable menus: https://www.npmjs.com/package/flaps
Swipeable cards: https://www.npmjs.com/package/swipeout

### Data

Observable state: https://www.npmjs.com/package/enti
AJAX requests: https://www.npmjs.com/package/cpjax

### "Routing"
Handling anchor clicks / push-state: https://www.npmjs.com/package/spath
Detecting URL change: https://www.npmjs.com/package/on-url-change

### Interaction

Scrolling: https://www.npmjs.com/package/scroll-into-view
Touch interaction: https://www.npmjs.com/package/interact-js

## DOM interaction

Wait for CSS layout: https://www.npmjs.com/package/laidout
Class manipulation: https://www.npmjs.com/package/classist

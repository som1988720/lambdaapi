[![Lambda API](https://www.jeremydaly.com/wp-content/uploads/2018/03/lambda-api-logo.svg)](https://serverless-api.com/)

[![Build Status](https://travis-ci.org/jeremydaly/lambda-api.svg?branch=master)](https://travis-ci.org/jeremydaly/lambda-api)
[![npm](https://img.shields.io/npm/v/lambda-api.svg)](https://www.npmjs.com/package/lambda-api)
[![npm](https://img.shields.io/npm/l/lambda-api.svg)](https://www.npmjs.com/package/lambda-api)
[![Coverage Status](https://coveralls.io/repos/github/jeremydaly/lambda-api/badge.svg?branch=master)](https://coveralls.io/github/jeremydaly/lambda-api?branch=master)

### Lightweight web framework for your serverless applications

Lambda API is a lightweight web framework for use with AWS API Gateway and AWS Lambda using Lambda Proxy Integration. This closely mirrors (and is based on) other web frameworks like Express.js and Fastify, but is significantly stripped down to maximize performance with Lambda's stateless, single run executions.

## Simple Example

```javascript
// Require the framework and instantiate it
const api = require('lambda-api')()

// Define a route
api.get('/status', (req,res) => {
  res.json({ status: 'ok' })
})

// Declare your Lambda handler
module.exports.handler = (event, context, callback) => {
  // Run the request
  api.run(event, context, callback)
}
```

For a full tutorial see [How To: Build a Serverless API with Serverless, AWS Lambda and Lambda API](https://www.jeremydaly.com/build-serverless-api-serverless-aws-lambda-lambda-api/).

## Why Another Web Framework?
Express.js, Fastify, Koa, Restify, and Hapi are just a few of the many amazing web frameworks out there for Node.js. So why build yet another one when there are so many great options already? One word: **DEPENDENCIES**.

These other frameworks are extremely powerful, but that benefit comes with the steep price of requiring several additional Node.js modules. Not only is this a bit of a security issue (see Beware of Third-Party Packages in [Securing Serverless](https://www.jeremydaly.com/securing-serverless-a-newbies-guide/)), but it also adds bloat to your codebase, filling your `node_modules` directory with a ton of extra files. For serverless applications that need to load quickly, all of these extra dependencies slow down execution and use more memory than necessary. Express.js has **30 dependencies**, Fastify has **12**, and Hapi has **17**! These numbers don't even include their dependencies' dependencies.

Lambda API has **ZERO** dependencies.

Lambda API was written to be extremely lightweight and built specifically for serverless applications using AWS Lambda. It provides support for API routing, serving up HTML pages, issuing redirects, serving binary files and much more. It has a powerful middleware and error handling system, allowing you to implement everything from custom authentication to complex logging systems. Best of all, it was designed to work with Lambda's Proxy Integration, automatically handling all the interaction with API Gateway for you. It parses **REQUESTS** and formats **RESPONSES** for you, allowing you to focus on your application's core functionality, instead of fiddling with inputs and outputs.

## Table of Contents
- [Installation](#installation)
- [Requirements](#requirements)
- [Configuration](#configuration)
- [Recent Updates](#recent-updates)
- [Routes and HTTP Methods](#routes-and-http-methods)
- [Returning Responses](#returning-responses)
  - [Async/Await](#asyncawait)
  - [Promises](#promises)
- [Route Prefixing](#route-prefixing)
- [Debugging Routes](#debugging-routes)
- [REQUEST](#request)
- [RESPONSE](#response)
  - [attachment()](#attachmentfilename)
  - [cache()](#cacheageprivate)
  - [clearCookie()](#clearcookiename-options)
  - [cookie()](#cookiename-value-options)
  - [cors()](#corsoptions)
  - [download()](#downloadfile--filename--options--callback)
  - [error()](#errormessage)
  - [etag()](#etagboolean)
  - [getHeader()](#getheaderkey)
  - [getLink()](#getlinks3path--expires--callback])
  - [hasHeader()](#hasheaderkey)
  - [header()](#headerkey-value)
  - [html()](#htmlbody)
  - [json()](#jsonbody)
  - [jsonp()](#jsonpbody)
  - [location](#locationpath)
  - [modified()](#modifieddate)
  - [redirect()](#redirectstatus-path)
  - [removeHeader()](#removeheaderkey)
  - [send()](#sendbody)
  - [sendFile()](#sendfilefile--options--callback)
  - [status()](#statuscode)
  - [type()](#typetype)
- [Enabling Binary Support](#enabling-binary-support)
- [Path Parameters](#path-parameters)
- [Wildcard Routes](#wildcard-routes)
- [Middleware](#middleware)
- [Clean Up](#clean-up)
- [Error Handling](#error-handling)
- [Namespaces](#namespaces)
- [CORS Support](#cors-support)
- [Lambda Proxy Integration](#lambda-proxy-integration)
- [Configuring Routes in API Gateway](#configuring-routes-in-api-gateway)
- [Contributions](#contributions)

## Installation
```
npm i lambda-api --save
```

## Requirements
- AWS Lambda running **Node 8.10+**
- AWS Gateway using [Proxy Integration](#lambda-proxy-integration)

## Configuration
Require the `lambda-api` module into your Lambda handler script and instantiate it. You can initialize the API with the following options:

| Property | Type | Description |
| -------- | ---- | ----------- |
| version  | `String` | Version number accessible via the `REQUEST` object |
| base  | `String` | Base path for all routes, e.g. `base: 'v1'` would prefix all routes with `/v1` |
| callbackName | `String` | Override the default callback query parameter name for JSONP calls |
| mimeTypes | `Object` | Name/value pairs of additional MIME types to be supported by the `type()`. The key should be the file extension (without the `.`) and the value should be the expected MIME type, e.g. `application/json` |

```javascript
// Require the framework and instantiate it with optional version and base parameters
const api = require('lambda-api')({ version: 'v1.0', base: 'v1' });
```

## Recent Updates
For detailed release notes see [Releases](https://github.com/jeremydaly/lambda-api/releases).

### v0.7: Restrict middleware execution to certain paths
Middleware now supports an optional path parameter that supports multiple paths, wildcards, and parameter matching to better control middleware execution. See [middleware](#middleware) for more information.

### v0.6: Support for both `callback-style` and `async-await`
In additional to `res.send()`, you can now simply `return` the body from your route and middleware functions. See [Returning Responses](#returning-responses) for more information.

### v0.5: Remove Bluebird Promises Dependency
Now that AWS Lambda supports Node v8.10, asynchronous operations can be handled more efficiently with `async/await` rather than with promises. The core Lambda API execution engine has been rewritten to take advantage of `async/await`, which means we no longer need to depend on Bluebird. We now have **ZERO** dependencies.

### v0.4: Binary Support
Binary support has been added! This allows you to both send and receive binary files from API Gateway. For more information, see [Enabling Binary Support](#enabling-binary-support).

### v0.3: New Instantiation Method
Please note that the invocation method has been changed. You no longer need to use the `new` keyword to instantiate Lambda API. It can now be instantiated in one line:

 ```javascript
 const api = require('lambda-api')()
 ```

`lambda-api` returns a `function` now instead of a `class`, so options can be passed in as its only argument:

```javascript
const api = require('lambda-api')({ version: 'v1.0', base: 'v1' });
```

**IMPORTANT:** Upgrading from <v0.3.0 requires either removing the `new` keyword or switching to the one-line format. This provides more flexibility for instantiating Lambda API in future releases.

## Routes and HTTP Methods

Routes are defined by using convenience methods or the `METHOD` method. There are currently eight convenience route methods: `get()`, `post()`, `put()`, `patch()`, `delete()`, `head()`, `options()` and `any()`. Convenience route methods require two parameters, a *route* and a function that accepts two arguments. A *route* is simply a path such as `/users`. The second parameter must be a function that accepts a `REQUEST` and a `RESPONSE` argument. These arguments can be named whatever you like, but convention dictates `req` and `res`. Examples using convenience route methods:

```javascript
api.get('/users', (req,res) => {
  // do something
})

api.post('/users', (req,res) => {
  // do something
})

api.delete('/users', (req,res) => {
  // do something
})
```
Additional methods are support by calling the `METHOD` method with three arguments. The first argument is the HTTP method (or array of methods), a *route*, and a function that accepts a `REQUEST` and a `RESPONSE` argument.

```javascript
api.METHOD('trace','/users', (req,res) => {
  // do something on TRACE
})

api.METHOD(['post','put'],'/users', (req,res) => {
  // do something on POST -or- PUT
})
```

All `GET` methods have a `HEAD` alias that executes the `GET` request but returns a blank `body`. `GET` requests should be idempotent with no side effects. The `head()` convenience method can be used to set specific paths for `HEAD` requests or to override default `GET` aliasing.

Routes that use the `any()` method or pass `ANY` to `api.METHOD` will respond to all HTTP methods. Routes that specify a specific method (such as `GET` or `POST`), will override the route for that method. For example:

```javascript
api.any('/users', (req,res) => { res.send('any') })
api.get('/users', (req,res) => { res.send('get') })
```

A `POST` to `/users` will return "any", but a `GET` request would return "get". Please note that routes defined with an `ANY` method will override default `HEAD` aliasing for `GET` routes.

## Returning Responses

Lambda API supports both `callback-style` and `async-await` for returning responses to users. The [RESPONSE](#response) object has several callbacks that will trigger a response (`send()`, `json()`, `html()`, etc.) You can use any of these callbacks from within route functions and middleware to send the response:

```javascript
api.get('/users', (req,res) => {
  res.send({ foo: 'bar' })
})
```

You can also `return` data from route functions and middleware. The contents will be sent as the body:

```javascript
api.get('/users', (req,res) => {
  return { foo: 'bar' }
})
```

### Async/Await  

If you prefer to use `async/await`, you can easily apply this to your route functions.

Using `return`:
```javascript
api.get('/users', async (req,res) => {
  let users = await getUsers()
  return users
})
```

Or using callbacks:
```javascript
api.get('/users', async (req,res) => {
  let users = await getUsers()
  res.send(users)
})
```

### Promises

If you like promises, you can either use a callback like `res.send()` at the end of your promise chain, or you can simply `return` the resolved promise:

```javascript
api.get('/users', (req,res) => {
  getUsers().then(users => {
    res.send(users)
  })
})
```

OR

```javascript
api.get('/users', (req,res) => {
  return getUsers().then(users => {
    return users
  })
})
```

**IMPORTANT:** You must either use a callback like `res.send()` **OR** `return` a value. Otherwise the execution will hang and no data will be sent to the user. Also, be sure not to return `undefined`, otherwise it will assume no response.

## Route Prefixing

Lambda API makes it easy to create multiple versions of the same api without changing routes by hand. The `register()` method allows you to load routes from an external file and prefix all of those routes using the `prefix` option. For example:

```javascript
// handler.js
const api = require('lambda-api')()

api.register(require('./routes/v1/products'), { prefix: '/v1' })
api.register(require('./routes/v2/products'), { prefix: '/v2' })

module.exports.handler = (event, context, callback) => {
  api.run(event, context, callback)
}
```

```javascript
// routes/v1/products.js
module.exports = (api, opts) => {
  api.get('/product', handler_v1)
}
```

```javascript
// routes/v2/products.js
module.exports = (api, opts) => {
  api.get('/product', handler_v2)
}
```

Even though both modules create a `/product` route, Lambda API will add the `prefix` to them, creating two unique routes. Your users can now access:
- `/v1/product`
- `/v2/product`

You can use `register()` as many times as you want AND it is recursive, so if you nest `register()` methods, the routes will build upon each other. For example:

```javascript
module.exports = (api, opts) => {
  api.get('/product', handler_v1)
  api.register(require('./v2/products.js'), { prefix: '/v2'} )
}
```

This would create a `/v1/product` and `/v1/v2/product` route. You can also use `register()` to load routes from an external file without the `prefix`. This will just add routes to your `base` path. **NOTE:** Prefixed routes are built off of your `base` path if one is set. If your `base` was set to `/api`, then the first example above would produce the routes: `/api/v1/product` and `/api/v2/product`.

## Debugging Routes

Lambda API has a `routes()` method that can be called on the main instance that will return an array containing the `METHOD` and full `PATH` of every configured route. This will include base paths and prefixed routes. This is helpful for debugging your routes.

```javascript
 const api = require('lambda-api')()

 api.get('/', (req,res) => {})
 api.post('/test', (req,res) => {})

 api.routes() // => [ [ 'GET', '/' ], [ 'POST', '/test' ] ]
```

You can also log the paths in table form to the console by passing in `true` as the only parameter.

```javascript
 const api = require('lambda-api')()

 api.get('/', (req,res) => {})
 api.post('/test', (req,res) => {})

 api.routes(true)

// Outputs to console
╔═══════════╤═════════════════╗
║  METHOD   │  ROUTE          ║
╟───────────┼─────────────────╢
║  GET      │  /              ║
╟───────────┼─────────────────╢
║  POST     │  /test          ║
╚═══════════╧═════════════════╝
```


## REQUEST

The `REQUEST` object contains a parsed and normalized request from API Gateway. It contains the following values by default:

- `app`: A reference to an instance of the app
- `version`: The version set at initialization
- `id`: The awsRequestId from the Lambda `context`
- `params`: Dynamic path parameters parsed from the path (see [path parameters](#path-parameters))
- `method`: The HTTP method of the request
- `path`: The path passed in by the request including the `base` and any `prefix` assigned to routes
- `query`: Querystring parameters parsed into an object
- `headers`: An object containing the request headers (properties converted to lowercase for HTTP/2, see [rfc7540 8.1.2. HTTP Header Fields](https://tools.ietf.org/html/rfc7540))
- `rawHeaders`: An object containing the original request headers (property case preserved)
- `body`: The body of the request. If the `isBase64Encoded` flag is `true`, it will be decoded automatically.
  - If the `content-type` header is `application/json`, it will attempt to parse the request using `JSON.parse()`
  - If the `content-type` header is `application/x-www-form-urlencoded`, it will attempt to parse a URL encoded string using `querystring`
  - Otherwise it will be plain text.
- `rawBody`: If the `isBase64Encoded` flag is `true`, this is a copy of the original, base64 encoded body
- `route`: The matched route of the request
- `requestContext`: The `requestContext` passed from the API Gateway
- `auth`: An object containing the `type` and `value` of an authorization header. Currently supports `Bearer`, `Basic`, `OAuth`, and `Digest` schemas. For the `Basic` schema, the object is extended with additional fields for username/password. For the `OAuth` schema, the object is extended with key/value pairs of the supplied OAuth 1.0 values.
- `namespace` or `ns`: A reference to modules added to the app's namespace (see [namespaces](#namespaces))
- `cookies`: An object containing cookies sent from the browser (see the [cookie](#cookiename-value-options) `RESPONSE` method)
- `context`: Reference to the `context` passed into the Lambda handler function

The request object can be used to pass additional information through the processing chain. For example, if you are using a piece of authentication middleware, you can add additional keys to the `REQUEST` object with information about the user. See [middleware](#middleware) for more information.

## RESPONSE

The `RESPONSE` object is used to send a response back to the API Gateway. The `RESPONSE` object contains several methods to manipulate responses. All methods are chainable unless they trigger a response.

### status(code)
The `status` method allows you to set the status code that is returned to API Gateway. By default this will be set to `200` for normal requests or `500` on a thrown error. Additional built-in errors such as `404 Not Found` and `405 Method Not Allowed` may also be returned. The `status()` method accepts a single integer argument.

```javascript
api.get('/users', (req,res) => {
  res.status(401).error('Not Authorized')
})
```

### header(key, value)
The `header` method allows for you to set additional headers to return to the client. By default, just the `content-type` header is sent with `application/json` as the value. Headers can be added or overwritten by calling the `header()` method with two string arguments. The first is the name of the header and then second is the value.

```javascript
api.get('/users', (req,res) => {
  res.header('content-type','text/html').send('<div>This is HTML</div>')
})
```

**NOTE:** Header keys are converted and stored as lowercase in compliance with [rfc7540 8.1.2. HTTP Header Fields](https://tools.ietf.org/html/rfc7540) for HTTP/2. Header convenience methods (`getHeader`, `hasHeader`, and `removeHeader`) automatically ignore case.

### getHeader([key])
Retrieve the current header object or pass the optional `key` parameter and retrieve a specific header value. `key` is case insensitive.

### hasHeader(key)
Returns a boolean indicating the existence of `key` in the response headers. `key` is case insensitive.

### removeHeader(key)
Removes header matching `key` from the response headers. `key` is case insensitive. This method is chainable.

### getLink(s3Path[,expires][,callback])
This returns a signed URL to the referenced file in S3 (using the `s3://{my-bucket}/{path-to-file}` format). You can optionally pass in an integer as the second parameter that will changed the default expiration time of the link. The expiration time is in seconds and defaults to `900`. In order to ensure proper URL signing, the `getLink()` must be asynchronous, and therefore returns a promise. You must either `await` the result or use a `.then` to retrieve the value.

There is an optional third parameter that takes an error handler callback. If the underlying `getSignedUrl()` call fails, the error will be returned using the standard `res.error()` method. You can override this by providing your own callback.

```javascript
// async/await
api.get('/getLink', async (req,res) => {
  let url = await res.getLink('s3://my-bucket/my-file.pdf')
  return { link: url }
})

// promises
api.get('/getLink', (req,res) => {
  res.getLink('s3://my-bucket/my-file.pdf').then(url => {
    res.json({ link: url })
  })
})
```

### send(body)
The `send` methods triggers the API to return data to the API Gateway. The `send` method accepts one parameter and sends the contents through as is, e.g. as an object, string, integer, etc. AWS Gateway expects a string, so the data should be converted accordingly.

### json(body)
There is a `json` convenience method for the `send` method that will set the headers to `application/json` as well as perform `JSON.stringify()` on the contents passed to it.

```javascript
api.get('/users', (req,res) => {
  res.json({ message: 'This will be converted automatically' })
})
```

### jsonp(body)
There is a `jsonp` convenience method for the `send` method that will set the headers to `application/json`, perform `JSON.stringify()` on the contents passed to it, and wrap the results in a callback function. By default, the callback function is named `callback`.

```javascript
res.jsonp({ foo: 'bar' })
// => callback({ "foo": "bar" })

res.status(500).jsonp({ error: 'some error'})
// => callback({ "error": "some error" })
```

The default can be changed by passing in `callback` as a URL parameter, e.g. `?callback=foo`.

```javascript
// ?callback=foo
res.jsonp({ foo: 'bar' })
// => foo({ "foo": "bar" })
```

You can change the default URL parameter using the optional `callback` option when initializing the API.

```javascript
const api = require('lambda-api')({ callback: 'cb' });

// ?cb=bar
res.jsonp({ foo: 'bar' })
// => bar({ "foo": "bar" })
```

### html(body)
There is also an `html` convenience method for the `send` method that will set the headers to `text/html` and pass through the contents.

```javascript
api.get('/users', (req,res) => {
  res.html('<div>This is HTML</div>')
})
```

### type(type)
Sets the `content-type` header for you based on a single `String` input. There are thousands of MIME types, many of which are likely never to be used by your application. Lambda API stores a list of the most popular file types and will automatically set the correct `content-type` based on the input. If the `type` contains the "/" character, then it sets the `content-type` to the value of `type`.

```javascript
res.type('.html');              // => 'text/html'
res.type('html');               // => 'text/html'
res.type('json');               // => 'application/json'
res.type('application/json');   // => 'application/json'
res.type('png');                // => 'image/png'
res.type('.doc');               // => 'application/msword'
res.type('text/css');           // => 'text/css'
```

For a complete list of auto supported types, see [mimemap.js](lib/mindmap.js). Custom MIME types can be added by using the `mimeTypes` option when instantiating Lambda API

### location(path)
The `location` convenience method sets the `Location:` header with the value of a single string argument. The value passed in is not validated but will be encoded before being added to the header. Values that are already encoded can be safely passed in. Note that a valid `3xx` status code must be set to trigger browser redirection. The value can be a relative/absolute path OR a FQDN.

```javascript
api.get('/redirectToHome', (req,res) => {
  res.location('/home').status(302).html('<div>Redirect to Home</div>')
})

api.get('/redirectToGithub', (req,res) => {
  res.location('https://github.com').status(302).html('<div>Redirect to GitHub</div>')
})
```

### redirect([status,] path)
The `redirect` convenience method triggers a redirection and ends the current API execution. This method is similar to the `location()` method, but it automatically sets the status code and calls `send()`. The redirection URL (relative/absolute path OR a FQDN) can be specified as the only parameter or as a second parameter when a valid `3xx` status code is supplied as the first parameter. The status code is set to `302` by default, but can be changed to `300`, `301`, `302`, `303`, `307`, or `308` by adding it as the first parameter.

```javascript
api.get('/redirectToHome', (req,res) => {
  res.redirect('/home')
})

api.get('/redirectToGithub', (req,res) => {
  res.redirect(301,'https://github.com')
})
```

### cors([options])
Convenience method for adding [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) headers to responses. An optional `options` object can be passed in to customize the defaults.

The six defined **CORS** headers are as follows:
- Access-Control-Allow-Origin (defaults to `*`)
- Access-Control-Allow-Methods (defaults to `GET, PUT, POST, DELETE, OPTIONS`)
- Access-Control-Allow-Headers (defaults to `Content-Type, Authorization, Content-Length, X-Requested-With`)
- Access-Control-Expose-Headers
- Access-Control-Max-Age
- Access-Control-Allow-Credentials

The `options` object can contain the following properties that correspond to the above headers:
- origin *(string)*
- methods *(string)*
- headers *(string)*
- exposeHeaders *(string)*
- maxAge *(number in milliseconds)*
- credentials *(boolean)*

Defaults can be set by calling `res.cors()` with no properties, or with any combination of the above options.

```javascript
res.cors({
  origin: 'example.com',
  methods: 'GET, POST, OPTIONS',
  headers: 'content-type, authorization',
  maxAge: 84000000
})
```

You can override existing values by calling `res.cors()` with just the updated values:

```javascript
res.cors({
  origin: 'api.example.com'
})
```

### error(message)
An error can be triggered by calling the `error` method. This will cause the API to stop execution and return the message to the client. Custom error handling can be accomplished using the [Error Handling](#error-handling) feature.

```javascript
api.get('/users', (req,res) => {
  res.error('This is an error')
})
```

### cookie(name, value [,options])

Convenience method for setting cookies. This method accepts a `name`, `value` and an optional `options` object with the following parameters:

| Property | Type | Description |
| -------- | ---- | ----------- |
| domain   | `String` | Domain name to use for the cookie. This defaults to the current domain. |
| expires  | `Date` | The expiration date of the cookie. Local dates will be converted to GMT. Creates session cookie if this value is not specified. |
| httpOnly | `Boolean` | Sets the cookie to be accessible only via a web server, not JavaScript. |
| maxAge | `Number` | Set the expiration time relative to the current time in milliseconds. Automatically sets the `expires` property if not explicitly provided. |
| path | `String` | Path for the cookie. Defaults to "/" for the root directory. |
| secure | `Boolean` | Sets the cookie to be used with HTTPS only. |
|sameSite | `Boolean` or `String` | Sets the SameSite value for cookie. `true` or `false` sets `Strict` or `Lax` respectively. Also allows a string value. See https://tools.ietf.org/html/draft-ietf-httpbis-cookie-same-site-00#section-4.1.1 |

The `name` attribute should be a string (auto-converted if not), but the `value` attribute can be any type of value. The `value` will be serialized (if an object, array, etc.) and then encoded using `encodeURIComponent` for safely assigning the cookie value. Cookies are automatically parsed, decoded, and available via the `REQUEST` object (see [REQUEST](#request)).

**NOTE:** The `cookie()` method only sets the header. A execution ending method like `send()`, `json()`, etc. must be called to send the response.

```javascript
res.cookie('foo', 'bar', { maxAge: 3600*1000, secure: true }).send()
res.cookie('fooObject', { foo: 'bar' }, { domain: '.test.com', path: '/admin', httpOnly: true }).send()
res.cookie('fooArray', [ 'one', 'two', 'three' ], { path: '/', httpOnly: true }).send()
```

### clearCookie(name [,options])
Convenience method for expiring cookies. Requires the `name` and optional `options` object as specified in the [cookie](#cookiename-value-options) method. This method will automatically set the expiration time. However, most browsers require the same options to clear a cookie as was used to set it. E.g. if you set the `path` to "/admin" when you set the cookie, you must use this same value to clear it.

```javascript
res.clearCookie('foo', { secure: true }).send()
res.clearCookie('fooObject', { domain: '.test.com', path: '/admin', httpOnly: true }).send()
res.clearCookie('fooArray', { path: '/', httpOnly: true }).send()
```
**NOTE:** The `clearCookie()` method only sets the header. A execution ending method like `send()`, `json()`, etc. must be called to send the response.

### etag([boolean])
Enables Etag generation for the response if at value of `true` is passed in. Lambda API will generate an Etag based on the body of the response and return the appropriate header. If the request contains an `If-No-Match` header that matches the generated Etag, a `304 Not Modified` response will be returned with a blank body.

### cache([age][,private])
Adds `cache-control` header to responses. If the first parameter is an `integer`, it will add a `max-age` to the header. The number should be in milliseconds. If the first parameter is `true`, it will add the cache headers with `max-age` set to `0` and use the current time for the `expires` header. If set to false, it will add a cache header with `no-cache, no-store, must-revalidate` as the value. You can also provide a custom string that will manually set the value of the `cache-control` header. And optional second argument takes a `boolean` and will set the `cache-control` to `private` This method is chainable.

```javascript
res.cache(false).send() // 'cache-control': 'no-cache, no-store, must-revalidate'
res.cache(1000).send() // 'cache-control': 'max-age=1'
res.cache(30000,true).send() // 'cache-control': 'private, max-age=30'
```

### modified(date)
Adds a `last-modified` header to responses. A value of `true` will set the value to the current date and time. A JavaScript `Date` object can also be passed in. Note that it will be converted to UTC if not already. A `string` can also be passed in and will be converted to a date if JavaScript's `Date()` function is able to parse it. A value of `false` will prevent the header from being generated, but will not remove any existing `last-modified` headers.

### attachment([filename])
Sets the HTTP response `content-disposition` header field to "attachment". If a `filename` is provided, then the `content-type` is set based on the file extension using the `type()` method and the "filename=" parameter is added to the `content-disposition` header.

```javascript
res.attachment()
// content-disposition: attachment

res.attachment('path/to/logo.png')
// content-disposition: attachment; filename="logo.png"
// content-type: image/png
```

### download(file [, filename] [, options] [, callback])
This transfers the `file` (either a local path, S3 file reference, or Javascript `Buffer`) as an "attachment". This is a convenience method that combines `attachment()` and `sendFile()` to prompt the user to download the file. This method optionally takes a `filename` as a second parameter that will overwrite the "filename=" parameter of the `content-disposition` header, otherwise it will use the filename from the `file`. An optional `options` object passes through to the [sendFile()](#sendfilefile--options--callback) method and takes the same parameters. Finally, a optional `callback` method can be defined which is passed through to [sendFile()](#sendfilefile--options--callback) as well.

```javascript
res.download('./files/sales-report.pdf')

res.download('./files/sales-report.pdf', 'report.pdf')

res.download('s3://my-bucket/path/to/file.png', 'logo.png', { maxAge: 3600000 })

res.download(<Buffer>, 'my-file.docx', { maxAge: 3600000 }, (err) => {
  if (err) {
    res.error('Custom File Error')
  }
})
```

### sendFile(file [, options] [, callback])
The `sendFile()` method takes up to three arguments. The first is the `file`. This is either a local filename (stored within your uploaded lambda code), a reference to a file in S3 (using the `s3://{my-bucket}/{path-to-file}` format), or a JavaScript `Buffer`. You can optionally pass an `options` object using the properties below as well as a callback function `callback(err)` that can handle custom errors or manipulate the response before sending to the client.

| Property | Type | Description | Default |
| -------- | ---- | ----------- | ------- |
| maxAge   | `Number` | Set the expiration time relative to the current time in milliseconds. Automatically sets the `Expires` header | 0 |
| root   | `String` | Root directory for relative filenames. |  |
| lastModified | `Boolean` or `String` | Sets the `last-modified` header to the last modified date of the file. This can be disabled by setting it to `false`, or overridden by setting it to a valid `Date` object | |
| headers | `Object` |  Key value pairs of additional headers to be sent with the file | |
| cacheControl | `Boolean` or `String` | Enable or disable setting `cache-control` response header. Override value with custom string. | true |
| private | `Boolean` | Sets the `cache-control` to `private`. | false |

```javascript
res.sendFile('./img/logo.png')

res.sendFile('./img/logo.png', { maxAge: 3600000 })

res.sendFile('s3://my-bucket/path/to/file.png', { maxAge: 3600000 })

res.sendFile(<Buffer>, 'my-file.docx', { maxAge: 3600000 }, (err) => {
  if (err) {
    res.error('Custom File Error')
  }
})
```

The `callback` function supports returning a promise, allowing you to perform additional tasks *after* the file is successfully loaded from the source. This can be used to perform additional synchronous tasks before returning control to the API execution.

**NOTE:** In order to access S3 files, your Lambda function must have `GetObject` access to the files you're attempting to access.

See [Enabling Binary Support](#enabling-binary-support) for more information.

## Enabling Binary Support
To enable binary support, you need to add `*/*` under "Binary Media Types" in **API Gateway** -> **APIs** -> **[ your api ]** -> **Settings**. This will also `base64` encode all body content, but Lambda API will automatically decode it for you.

![Binary Media Types](http://jeremydaly.com//lambda-api/binary-media-types.png)
*Add* `*/*` *to Binary Media Types*

## Path Parameters
Path parameters are extracted from the path sent in by API Gateway. Although API Gateway supports path parameters, the API doesn't use these values but insteads extracts them from the actual path. This gives you more flexibility with the API Gateway configuration. Path parameters are defined in routes using a colon `:` as a prefix.

```javascript
api.get('/users/:userId', (req,res) => {
  res.send('User ID: ' + req.params.userId)
})
```

Path parameters act as wildcards that capture the value into the `params` object. The example above would match `/users/123` and `/users/test`. The system always looks for static paths first, so if you defined paths for `/users/test` and `/users/:userId`, exact path matches would take precedence. Path parameters only match the part of the path they are defined on. E.g. `/users/456/test` would not match `/users/:userId`. You would either need to define `/users/:userId/test` as its own path, or create another path with an additional path parameter, e.g. `/users/:userId/:anotherParam`.

A path can contain as many parameters as you want. E.g. `/users/:param1/:param2/:param3`.

## Wildcard Routes
Wildcard routes are supported for methods that **match an existing route**. E.g. `options` on an existing `get` route. Wildcards only work at the *end of a route definition* such as `/*` or `/users/*`. Wildcards within a path, e.g. `/users/*/posts` are not supported. Wildcard routes do support parameters, so `/users/:id/*` would capture the `:id` parameter in your wildcard handler.

Wildcard routes will match deep paths if the route exists. For example, an `OPTIONS` method for path `/users/*` would match `/users/:id/posts/latest` if that path was defined by another method.

The best use case is for the `OPTIONS` method to provide CORS headers. For more control over variable paths, use [Path Parameters](#path-parameters) instead.

```javascript
api.options('/users/*', (req,res) => {
  // Do something
  res.status(200).send({})
})
```

## Middleware
The API supports middleware to preprocess requests before they execute their matching routes. Middleware is defined using the `use` method and requires a function with three parameters for the `REQUEST`, `RESPONSE`, and `next` callback. For example:

```javascript
api.use((req,res,next) => {
  // do something
  next()
})
```

Middleware can be used to authenticate requests, log API calls, etc. The `REQUEST` and `RESPONSE` objects behave as they do within routes, allowing you to manipulate either object. In the case of authentication, for example, you could verify a request and update the `REQUEST` with an `authorized` flag and continue execution. Or if the request couldn't be authorized, you could respond with an error directly from the middleware. For example:

```javascript
// Auth User
api.use((req,res,next) => {
  if (req.headers.authorization === 'some value') {
    req.authorized = true
    next() // continue execution
  } else {
    res.status(401).error('Not Authorized')
  }
})
```

The `next()` callback tells the system to continue executing. If this is not called then the system will hang and eventually timeout unless another request ending call such as `error` is called. You can define as many middleware functions as you want. They will execute serially and synchronously in the order in which they are defined.

**NOTE:** Middleware can use either callbacks like `res.send()` or `return` to trigger a response to the user. Please note that calling either one of these from within a middleware function will return the response immediately.

### Restricting middleware execution to certain path(s)

By default, middleware will execute on every path. If you only need it to execute for specific paths, pass the path (or array of paths) as the first parameter to the `use` function.

```javascript
// Single path
api.use('/users', (req,res,next) => { next() })

// Wildcard path
api.use('/users/*', (req,res,next) => { next() })

// Multiple path
api.use(['/users','/posts'], (req,res,next) => { next() })

// Parameterized paths
api.use('/users/:userId',(req,res,next) => { next() })

// Multiple paths with parameters and wildcards
api.use(['/comments','/users/:userId','/posts/*'],(req,res,next) => { next() })
```

Path matching checks both the supplied `path` and the defined `route`. This means that parameterized paths can be matched by either the parameter (e.g. `/users/:param1`) or by an exact matching path (e.g. `/users/123`).


## Clean Up
The API has a built-in clean up method called 'finally()' that will execute after all middleware and routes have been completed, but before execution is complete. This can be used to close database connections or to perform other clean up functions. A clean up function can be defined using the `finally` method and requires a function with two parameters for the REQUEST and the RESPONSE as its only argument. For example:

```javascript
api.finally((req,res) => {
  // close unneeded database connections and perform clean up
})
```

The `RESPONSE` **CANNOT** be manipulated since it has already been generated. Only one `finally()` method can be defined and will execute after properly handled errors as well.

## Error Handling
The API has simple built-in error handling that will log the error using `console.log`. These will be available via CloudWatch Logs. By default, errors will trigger a JSON response with the error message. If you would like to define additional error handling, you can define them using the `use` method similar to middleware. Error handling middleware must be defined as a function with **four** arguments instead of three like normal middleware. An additional `error` parameter must be added as the first parameter. This will contain the error object generated.

```javascript
api.use((err,req,res,next) => {
  // do something with the error
  next()
})
```

The `next()` callback will cause the script to continue executing and eventually call the standard error handling function. You can short-circuit the default handler by calling a request ending method such as `send`, `html`, or `json` OR by `return`ing data from your handler.

## Namespaces
Lambda API allows you to map specific modules to namespaces that can be accessed from the `REQUEST` object. This is helpful when using the pattern in which you create a module that exports middleware, error, or route functions. In the example below, the `data` namespace is added to the API and then accessed by reference within an included module.

The main handler file might look like this:

```javascript
// Use app() function to add 'data' namespace
api.app('data',require('./lib/data.js'))

// Create a get route to load user details
api.get('/users/:userId', require('./lib/users.js'))
```

The users.js module might look like this:

```javascript
module.exports = (req, res) => {
  let userInfo = req.namespace.data.getUser(req.params.userId)
  res.json({ 'userInfo': userInfo })
}
```

By saving references in namespaces, you can access them without needing to require them in every module. Namespaces can be added using the `app()` method of the API. `app()` accepts either two parameters: a string representing the name of the namespace and a function reference *OR* an object with string names as keys and function references as the values. For example:

```javascript
api.app('namespace',require('./lib/ns-functions.js'))

// OR

api.app({
  'namespace1': require('./lib/ns1-functions.js'),
  'namespace2': require('./lib/ns2-functions.js')
})
```

## CORS Support
CORS can be implemented using the [wildcard routes](#wildcard-routes) feature. A typical implementation would be as follows:

```javascript
api.options('/*', (req,res) => {
  // Add CORS headers
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Methods', 'GET, PUT, POST, DELETE, OPTIONS');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization, Content-Length, X-Requested-With');
  res.status(200).send({});
})
```

You can also use the `cors()` ([see here](#corsoptions)) convenience method to add CORS headers.

Conditional route support could be added via middleware or with conditional logic within the `OPTIONS` route.

## Lambda Proxy Integration
Lambda Proxy Integration is an option in API Gateway that allows the details of an API request to be passed as the `event` parameter of a Lambda function. A typical API Gateway request event with Lambda Proxy Integration enabled looks like this:

```javascript
{
  "resource": "/v1/posts",
  "path": "/v1/posts",
  "httpMethod": "GET",
  "headers": {
    "Authorization": "Bearer ...",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
    "Accept-Encoding": "gzip, deflate",
    "Accept-Language": "en-us",
    "cache-control": "max-age=0",
    "CloudFront-Forwarded-Proto": "https",
    "CloudFront-Is-Desktop-Viewer": "true",
    "CloudFront-Is-Mobile-Viewer": "false",
    "CloudFront-Is-SmartTV-Viewer": "false",
    "CloudFront-Is-Tablet-Viewer": "false",
    "CloudFront-Viewer-Country": "US",
    "Cookie": "...",
    "Host": "...",
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_3) ...",
    "Via": "2.0 ... (CloudFront)",
    "X-Amz-Cf-Id": "...",
    "X-Amzn-Trace-Id": "...",
    "X-Forwarded-For": "xxx.xxx.xxx.xxx",
    "X-Forwarded-Port": "443",
    "X-Forwarded-Proto": "https"
  },
  "queryStringParameters": {
    "qs1": "q1"
  },
  "stageVariables": null,
  "requestContext": {
    "accountId": "...",
    "resourceId": "...",
    "stage": "prod",
    "requestId": "...",
    "identity": {
      "cognitoIdentityPoolId": null,
      "accountId": null,
      "cognitoIdentityId": null,
      "caller": null,
      "apiKey": null,
      "sourceIp": "xxx.xxx.xxx.xxx",
      "accessKey": null,
      "cognitoAuthenticationType": null,
      "cognitoAuthenticationProvider": null,
      "userArn": null,
      "userAgent": "...",
      "user": null
    },
    "resourcePath": "/v1/posts",
    "httpMethod": "GET",
    "apiId": "..."
  },
  "body": null,
  "isBase64Encoded": false
}
```

The API automatically parses this information to create a normalized `REQUEST` object. The request can then be routed using the APIs methods.

## Configuring Routes in API Gateway
Routes must be configured in API Gateway in order to support routing to the Lambda function. The easiest way to support all of your routes without recreating them is to use [API Gateway's Proxy Integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-set-up-simple-proxy.html#api-gateway-proxy-resource?icmpid=docs_apigateway_console).

Simply create a `{proxy+}` route that uses the `ANY` method and all requests will be routed to your Lambda function and processed by the `lambda-api` module. In order for a "root" path mapping to work, you also need to create an `ANY` route for `/`.

## Reusing Persistent Connections
If you are using persistent connections in your function routes (such as AWS RDS or Elasticache), be sure to set `context.callbackWaitsForEmptyEventLoop = false;` in your main handler. This will allow the freezing of connections and will prevent Lambda from hanging on open connections. See [here](https://www.jeremydaly.com/reuse-database-connections-aws-lambda/) for more information.

## Contributions
Contributions, ideas and bug reports are welcome and greatly appreciated. Please add  [issues](https://github.com/jeremydaly/lambda-api/issues) for suggestions and bug reports or create a pull request.

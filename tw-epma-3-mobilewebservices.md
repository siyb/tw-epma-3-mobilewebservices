% Mobile Webservice
% Patrick Sturm
% 01.12.2015

# Agenda

## Agenda

* Relevance
* Theory - General / Mobile specific
* Introduction to Node.js and mongoDB
* Code Example

# Relevance

## Relevance - 1 - Webservices

* Quite a few mobile applications are powered by webservices
* Some of the Webservices that you will encounter are legacy systems, that are not optimized (for mobile consumption)
* Sometimes, you will be able to cooperate creating the Webservice for your application ...
* ... you might even be responsible for desinging and implementing the Webservice

## Relevance - 2 - Webservices

* Badly written Webservices have an impact on:
    * The developers mood
    * The projects cost
    * UX


## Relevance - 3 - Why is Mobile special

* Transmitting mobile data is expensive
    * data plans
* Mobile devices have a limited amount of power
    * parsing huge chunks of data in combination with unnecessary requests causes power drain
* Poor connectivity
    * connection can cut off any time, large amounts of data might even be risky to transfer
* Transmitting on demand causes the device to waste energy
    * many requests that wake the radio
* Memory limitations
    * Android
        * especially on older platforms (devices), goes down to 16MB (24MB, 32MB, 48MB)
        * can be vendor / device specific, never assume anything!

# Theory

## Theory - 1 - General

* There are certain criteria that should be considered for all Webservices
* There is no silver bullet, there are downsides to almost all things discussed here:
    * Best decision: the one that you gain the most from
* Use integration tests (black box) to cover your API spec

## Theory - 1 - gzip

* gzip your responses! It sounds really trivial, but a lot of Webservices in the wild (especially legacy systems) do not do that
* I've heard a lot of excuses like "Compression puts extra strain on the servers"
    * gzip is a must to reduce transmitted data
* Results:
    * Reduction of transmitted data
    * Decompression is CPU intensive, thus more power is required.

## Theory - 2 - not all requests are equal

* HEAD requests are obviously less costly than GET requests
* We can use HEAD to "sneak a peek"
* Last-Modified
    * Make sure to support it!
    * Only request data (on the mobile side) if the last update was before Last-Modified
* Results:
    * Only obtain data when changes occured -> less traffic
    * Two requests required to obtain data

## Theory - 3 - support filtering

* Again, this is actually pretty obvious, allow API consumers to filter response data
* http://mywebservice.com/entity/1/filter=name|age|height (note that some consider "|" to be a bad character to use in an url)
* Results:
    * Decrease the amount of transmitted data
    * Allow developers to optimize requests
        * Decrease (database)server load
    * More options -> more ways to fail

## Theory - 4 - support pagination

* If you are providing collections of data to a consumer, always support pagination
* http://mywebservice.com/entity/page=1&items=10, where page is the page number and items are the items per page to request (skip / limit respectivly)
* Do not forget to support filtering as well! Pagination only is not enough!
* Results:
    * same as filtering
    * prevent large return bodies that may cause issues when loading data into memory
        * no need to use stream parsers like SAX (error prone / maintainability hell)
    
## Theory - 5 - slim data formats / general pointers

* Use slim data formats - meaning formats without huge overhead
* i.e. perfer JSON to lets say XML(SOAP!) for instance
    * there are other alternatives to JSON, but you get the point (ProtoBuf, YAML, etc)
* Make sure not to send out redundant data, e.g. id fields referencing to encapsulating instance
* Do not use base64 (or similar algorithms / measures) to encode binary data to be encoded by the consumer
* Results:
    * reduce traffic by reducing (unnecessary) protocol overhead
    * can be problematic if structural meta data is required (Java object serialization for instance -> RPC)
    
## Theory - 6 - versioning / mobile specific API
* API versioning is another MUST have, not just for mobile, pick your poison:
    * URL: http://mywebservice.com/v1/entity/1
        * not REST, because the URL does not soley identify a resource any more
    * API Version Header:
        * Custom Header
        * May cause server and client side caching issues
    * Use the Accept header
        * Accept: application/vnd.myname+json; version=2.0
        * vnd = Vendor Mime Type
        * Harder to test, header needs to be prepared
* i18n
    * use Accept-Language if you are planning to display content to the user

* Provide a mobile specific API gateway (facade)!

# Intro node.js / mongodb

## node.js

* node.js = server side JavaScript
* Build on top of google's v8
* Newest version is not entirely es6 complient
* node.js is event based, which can result in "callback hell"
* A node.js developer will work on a single thread only, IO (blocking) calls executed with callbacks
* Object Prototypes vs. Classes

## mongoDB

* document store
* JSON (binary) storage
* aggregation pipeline / map reduce
* not really ACID complient (split brain / clustering / sharding), atomic transaction on document basis
* schema less
* referred to as "yolo-db" by our sysop ;)

## node.js - 1 - first steps

* node.js code is organized in modules: can be anything assignable to a variable

```javascript
const MyClass = class MyClass {
     constructor() {
        this.a = 1;
     }
     printA() {
         console.log(this.a);
     }
}
module.exports = MyClass;
```
```javascript
// loading module
var MyClass = require('./path/to/MyClass');
var myInstance = new MyClass();
```

## nodejs - 2 - functions

* functions are first class citizens (Objects)
* can be used as other Objects: assigned to variables, passed as function parameters, getting prototype info
* they can be bound to a context:
```javascript
var fna = function a() {
    console.log(this.x);
};
// create new function by binding to
// different context
var boundFna = fna.bind(this);
```
* can be exported by modules as well
* function signatures are not set in stone
    * arguments!

## nodejs - 3 - objects

* JavaScript is Object Oriented - Prototype vs. Class
* node.js 4.X added supports for classes, lets use those for now (where applicable)
* JavaScript knows null / undefined, variables do not have to be set

## nodejs - 4 - callbacks

* node.js relies on callbacks to establish a control flow
```javascript
doSomeThing(1,2,3, function (err, result) {
    if (err) {
        return;
    }
    save(result);
});
``` 
* usually: callback(err, result, ...)

## nodejs - 5 - frameworks / npm

* we will rely heavyly on frameworks
    * express - nodejs web framework
    * mongoose - nodejs mongoDB framework (allows schemas)
    * lodash - neat utilities for data manipulation
* we will not be using promises (for now)
* npm
    * package manager
        * npm init - initialize your npm package
        * npm install <module> --save - install module and save to package.json
        * npm install - (without parameters) install packages listed in package.json


# [Example Code](https://github.com/SphericalElephant/nodejs-example-webservice)


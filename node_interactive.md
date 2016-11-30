[back](index)

# Keynotes

## Security
Adam Baldwin, ^Lift Security

Run node security project

* Evangelize

Moving vendor neutral into nodejs foundation.

## Express

Doug Wilson, talking about governance after moving into nodejs foundation.

"Incubating project" in 2016

* mentors
* governance guideline
* find founders
* work to graduate. (welfare?) metrics: 
  * participatory
  * effective
* Technical committee:
  * meet biweekly
  * make decisions if necessary
  * new ideas

express is 3 github organizations: expressjs, pillarjs (underlying core components), jshttp (http parsing/handling modules)

Discussed the express 5.0 features - very comprehensible

THEN talked about trying to get more committers.  Weak but effective pitch!

## Security (er, a sales pitch)

Joe McCann of nodesource

n|solid - "enterprise node"

This is a sales pitch.

NASA is using node - is that even good?

Available in amazon marketplace - but what the fuck is it?

Starting a beta for "certified modules" - seems like an excellent idea.

## Something something node something

Andy Hoyt, IBM/Strongloop

Bizarre "you can API too!" pitch.  Whatever.

Disrupt yourself.  Stupid assertions without evidence.

Maybe just trying to convince people that IBM isn't dying?

openWhisk - open source serverless framework.

## contributing

William Kapke (@williamkapke)

"coding not required"

Does a great review of the history of openness of the node foundation, re: io.js.  Evidently iojs was an open fork that was merged back in nodejs in the last couple years.

The slides are all beautiful hi res images, almost no text - really really engaging, and beautiful.  Made using images from https://pexels.com which now is going to change my life, presentation made in keynote

Relationship with linux foundation: nodejs pays linux for consultation services. "Like homeowners association management company"

Use github for almost everything.  Each working group has a repo dedicated to collaboration - docs, issue tracking, etc.  Use github notifications feature.

The project is a complex 501c6 organization with official working group/committee structure.  Each group has official charter documents, together with philosophic material.

## Cloud/enterprise

Jonathan Carter, Microsoft @lostintangent

He demoed on a mac!

Most of the stuff in the demo is OSS

https://github.com/scotch-io/node-todo

### visualstudio-code

Demo using vsualstudio-code.  Dev environment is controlled as git repo itself.

Autocomplete for node core APIs - that would be great.

Reference to 12-factor app.

Uses node-inspect inline in running project.  Pretty baller!  And a fucking integrated front-end debugger as well!

vs-code has a docker extension which figures out a simple docker file.

### Azure

New capability for container deployment.

New CLI - demoed a couple of easy commands.

Environment variables configured through azure, app-service "app settings" is just environment variables.

## accelerating solutions

david stewart, intel

Morked w/ java to speed up dramatically through integrated software/hardware changes.

lead, leap, link

### servant leadership

Added node to http://languagesperformance.intel.com

Try to help with perf regressions - intel engineers locate patches which cause issues.

http://github.com/Node-DC - model application datacenter workloads

### leap - big perf gains

Working on network accelerator - big opportunity for big perf improvement in api gateway (io bound)

### link - collaborate with OS

---

# Tuesday morning

## Reading node core

Rich @Trott, UC San Fran

https://github.com/nodejs/node/tree/master/test

Tests use their own test framework https://github.com/nodejs/node/blob/master/test/common.js - looks to use TAP

http://nodetodo.org/ This guy is one of the two doing nodetodo - a way to get into node core.

## Developing nirvana

Corey Butlery, https://Author.io (static desktop web server)

"Time is the currency of life"

Manage
* time
* stress

This feller created NVM for windows around node 0.11 - managing tons of change.  What makes it popular?

* Trust.  Trick: piggyback off trusted tech.
* keep asking questions

Not the most excellent talk.

## Nodifying the enterprise

Prince Soni, To The New (company)

Architecting node projects at work.

If one more person tells me that JS is the fastest growing language I'm going to do a backflip. Holy crap, he's giving a JS history lesson.

### Evolution (this is kind of bogus):

* lightweight server and API layer
* real-time socket apps
* enterprise apps
* IOT
* Desktop, electron

### How do enterprises benefit from node?

* Rapid delivery
  * Fallacy that nodejs <=> microservice architecture
* change management:
  * Fallacy that nodejs <=> microservice architecture
* LTS:
  * That's idiotic - as though nodejs invented LTS.  They seem to be doing it well, though.

Literally no content.

### Security

Configuration security - helmet package helps with headers.

Session security: csrf, cookies.

Code: don't eval, be careful with stack traces.  No help!

NSP, N|Solid - frameworks with sec and monitoring frameworks.

### Enterprise toolkits

IDEs - atom, sublime, webstorm

Frameworks - sails, hapi, etc

I could have fucking googled the content of this talk in a couple of hours.  Why did this cost $600?

Did they PAY this speaker?

---

# Tuesday early afternoon

## Don't let node take the blame

Daniel Khan Lead at dynatrace @dkhan

paypal - started using nodejs to prototype, now use for production

One gets enterprise-y stuff to talk to a new node service - slowness in one of the reliant services can manifest in a node hub.

Proactive defense - know how it works.  Look into CPU v memory.

memory is code segment, stack, heap, used heap - given with `process.memoryUsage()` - can just write out to a csv and analyze!

How to synthesize a memory leak? shared on twitter.  `unused` is not called, `someMethod` is not called.  Closure contains a reference to an unused method - it's a circular reference.  Heap is a graph, and circular references trick GC because it doesn't detect them.

v8-profiler - http://bit.ly/1Pvijiy - can feed directly into chrome dev tools.  blog: http://bit.ly/1fb0Xm (image is a heat graph on cpu usage)

Reference to netflix blog "node.js in flames" - growing routing table problem.  express routing table lookups are `O(n)` because it's stored in an array, not a hashtable.

Mentions trace by risingstack, N|Solid.  Tools that give nodejs metrics.

"Boundaries between tiers in a stack are boundaries between generations, in a way"

backpressure - node service calls out to slower system, requests can back up and _cause node to fail_.

Solution: one-off monitoring?
Thomas watson will talk about transactional tracing in node.  (dynatrace does it as well), zipkin?.  At then, go for one single alert.

Summary:

* Use dedicated monitoring tools for debugging and tracing, and look to what node provides
* Consider transactional tracing tools that follows the tiers

## Solving service discovery

Or, why each microservice should believe it's the only one in the world

Richard Rodger nearForm @rjrodger

"Here's the story we went on"

Perfection - the assumption that there will be no bugs creates bureaocracy.
gossamer albatross - more appropriate than shuttle flight system.  System was tuned for iteration - replaceable parts, etc.

Node modules - subpar for for components of an infrastructure.

composibility - the service interface determines how composable components are => yak shaving.

What is a microservice?

* independent process
* communicate via messages (rest? http? whatever)
* component model

```
A -> B

bad /\

-------

   A ->
-> B

good /\
```
You can't get this second bit writing HTTP rest calls in your app.  Requires identity.

507 Mechanical Movements - book with mechanical components.

* Pattern match:
  * messages are first-class in code
  * Basically talking about seneca-type messaging
* Transport independence

Requires peer-to-peer.  "scalable weakly-consistent infection-style membership protocol" SWIM

OK, nearform is seneca.
@taomicroservice

## full stack testing

Stacy Kirk, QualityWorks @queenofagileqa

"The testing pyramid" - unit, integration, functional, manual

OK, you don't need to sell me on testing...

Assertion: "nodejs makes test automation a lot easier"

### Some tools

* nightwatch - https://github.com/nightwatchjs/nightwatch - ui testing framework
* casperjs w/ phantom
* chakram - REST test framework
* webdriver.io - mobile - recommended with mocha and appium
* intern.io - sort of a "do literally everything" framwork
* mocha (ugh...)
* jasmine - BDD
* karma - angular
* qualitywatcher.io - reporting dashboard!
* phantomas - perf, looks like a well-featured tool
* qualitymeter - perf, visualize perf data, maybe from phantomas

### best practices

* Integrate tests in `test/` - means you can just write `npm test`
* Data-driven testing (you know about that)
* Use page-object pattern
* Be able to _show_ data from testing results - to show leadership!
* Test performance up front

`expect().to.have.schema` - Check that out! http://chaijs.com/plugins/chai-json-schema/

http://assess.qualityworkscg.com

---

# Tuesday late afternoon

## Documentation

Rand McKinney IBM/strongloop  @randmckinney
http://loopback.io

* testing info
* style guide
* process guidelines

### How to fix?

* Triage doc tasks - put them into the process:
  * strongloop uses a 'needsdoc' label to note things which require doc changes
* Have a supportive doc framework
* require tests and docs for all pulls
* Have a curator (README czar/editor)
* Make it easy to contribute
* Provide authoring components
* Organized:
  * usable search
  * navigation
* Up-to-date

### Types

* API (ie jsdoc)
* Task (how-to)
* Concept (apologia for design)
* Tutorial (getting started/quickstart)

loopback using strongdoc (like jsdoc, looks proprietary)

Examples are critical

### Frameworks

* README:
  * auto-published
  * limited to markdown
  * should basically look like the top of POD
* built-in wiki:
  * Doesn't "clutter" project with docs
  * https://github.com/netflix/hysterix
* Jekyll site:
  * customizable layout, branding, etc
  * A bit more work
  * loopback.io and expressjs.org both use jekyll

## ChakraCore

Arunesh Chandra MS @aruneshc

ChakraCore is the core of the JS engine for Microsoft Edge browser

Is OSS, developed entirely in https://github.com/Microsoft/chakracore

Is x-platform: linux and OS X (experimental, interpreter only)

ARM thumb-2 was not targeted by node, but was by windows IoT - so they started on node-chakracore

Started with v8 emulating shim

https://aka.ms/NodeTTD - time-travel debugging

I didn't pay much attention.

## Perf stores

David Clements, nearform @davidmarkclem

http://davidmarkclements.github.io/perf-stories

high-performance code that looked like it was written by a novice - comment optimized code.

* perf-sym - mapping from osx hex addresses to symbols
* 0x - native flame graph generation from node perf
* autocannon (alt to wrk or apache bench) - stress testing web applications
* pino - high-performance logger, compatible with bunyon

The eval trick: take inputs to a function and function def and eval as a string

## Javascript minus javascript

Sarah Meyer, buzzfeed @meyerini

js first - site only works with JS enabled

URL - should change as resource changes

* Create bidirectional mapping between URLs and page state
* respond to XHR headers with JSON and non-XHR with HTML

Fun talk.  Pretty high-level, not much detail.


# Wednesday morning

## Event loop

Sam Roberts, IBM/strongloop @octetcloud

Text on the presentation a bit tough to read from my seat.

###

```
int s = socket();
```
/\ file descriptor for a socket

problem: thread per socket
```
int server = socket();
bind(server, 80);
listen(server);  // <- at this point, now accepting instead of writing
while (int con = accept(server))
```
`accept` blocks, and when resolves, get a new socket connection

While threads are lighter now, thread/socket is still relatively heavy.

soln: epoll (linux, BSD uses kqueue)

```
int s = ...;
int eventId = epoll_create1(0);
struct epoll_event ev = { events = EPOLLIN, ...};
while (int max = epoll_wait(eventId, events, 10)) // blocking
{for n=0,n<max, n+++) {
 //
 int con = accept(server);
 // subscribe to this conn in epoll_event
 }}
 
```

This is hard, thus `libuv`

"A semi-infitite lopp poling on OS until some file descriptors ready"

semi-infinite: `unref` marks handles as unused; node exits when nothing is being listened.  Nice trick for allowing node to free on non-critical resources!

What can be polled?

* pollable file descriptors (not real files, but other handles)
* time
* other things: tough

file-system: can't be selected on directly, but use single-byte 2-ended pipe.  *Thread pool*

Sometimes pollable: dns.  DNS not always used.  `dns.lookup()` calls `getaddrinfo()` is not async-able.

C++ addons are only sometimes pollable.

Useful metric: loop time.  Should finish 8-15ms.

http://goo.gl/N7oLVK - code & slides

http://goo.gl/EIPclI - Bert Belder's talk

Really excellent talk!

Spoke to Sam after.  He described backpressure:
Server is not able to read off TCP socket, client is blocked, or queues - whatever.  Client is taking the brunt, but server is safe - distributed pressure.  But only works when server is using `streams` for that TCP conn. Read https://nodejs.org/dist/latest-v6.x/docs/api/stream.html - are streams used natively by the http/tcp server/client libs?

## Security

Guy Podjarny, Snyk @guypod

The things making node awesome make it vulnerable.

~14% of NPM packages have known vluns (snyk internal data october 2016)

Demo in https://github.com/snyk/goof

st has a directory traversal vuln : `public/../..` doesn't work, but `public/%2e%2e/%2e%2e` does

`[Gotcha](javascript:alert(1))` - he demos that the browser interacts with sanitization to break encoding and allow an XSS attack

DoS vuln - since thing is single-threaded, if you choke a single thread with a bad regexp, you choke all your clients

mongoose buffer vuln - JSON deserialization: passing in an integer instead of string to buffer instantiation leaks proc memory

Shit, Snyk has a _really_ nice tool for this.  Literally created a PR in github that patches the exploits.

https://snyk.io/test

## Crypto

Adam Englander, iovation @adam_englander

Cool slide: AES-ECB encrypted Tux is still recognizable (AES-CBC is good)

Good entropy: IVs and feedback loops (like CBC)

### symmetric shared secrets

block or stream ciphers - CBC

Don't use ECB (electronic cookbook)

Dig signature for authenticity - HMAC (SHA256 better than MD5 for collision resistance)

### public/private key pairs

Private key can encrypt, decrypt, sign, and verify

Public key can only encrypt and verify

2048 is the current minimum recommended

RSA can only encrypt or sign up to key length, which can be problematic.  Can be mixed with symmetric key techniques (I don't understand how)

`PKCS1_PADDING` no good, `PKCS1_OAEP_PADDING` good

### key derivation

Inject salt, iterate to make expensive, moar threads moar memory

* argon21 new hotness [https://npm.im/argon2]
* scrypt preferable
* bcrypt acceptable...
* pbkdf2 not good

### Stuff

ecryption reversible, key derivation/digital signature not

Dig signature for authenticity - HMAC (SHA256 better than MD5 for collision resistance)

key derivation : computationally expensive by design - important

### recommendations

* use `crypto.randomBytes` for randomness
* Use asym RSA for transfer, and AES-CBC generally
* AES: aes-256-cvc /sha256
* RSA: 20148 PKCS1_OAEP / rsa-sha256

## Performance

@wa7son

Old: patch literally everything async to install a handler (done by opbeat, new relic, etc)

New: AsyncHooks https://github.com/nodejs/diagnostics (diag working group) - still experimental, not documented.

```
var aW = process.binding('async_wrap');

aW.setupHooks({ init, pre port, destroy});
aW.enable();
function init ();
function pre()
...
```

Secret function: `process._rawDebug()` is a synchronous `console.log`

IO in user space is a problem.  Embedder API to fix - provides an async hook emitter API.

## Debugging

Josh Gavant at microsoft @joshugav

https://github.com/nodejs/diagnostics
https://github.com/nodejs/post-mortem

* node-inspect, v8 inspector
* nodereport - dump state on sigs and crashes
* llnode - extension to lldb
* v8 tracingcontroller - still just a PR

### v8 inspector

Same protocol as chromium, step, conn to live script, heap, CPU profiling

Used 
* tshark - wireshark tool
* wscat - websocket connection tool.  Kinda opens a WS terminal.

### Inspector

localhost:9229
* `/json/list`
* `/version`
* `/protocol` - full schema for inspector

You get the UUID to use with inspector by the end of the chrome-devtools string.
 `wscat -c ws://localhost:9229/<uuid>`
 
 Search chrome debugger protocol
 
 Used chrome-remote-interface
 Used node-inspect, which is a CLI client, as opposed to opening in chrome
 
## Building and shipping with Docker

Sophia Parafina, Docker @spara

Owl drawer!  Good metaphor.  Someone gives you two circles, and you have to draw the owl.

"A virtual machine is a house, a container is an apartment building"

Tiers: image, container, engine

docker-swarm-visualizer - super useful tool if using swarm

---

# Wednesday early afternoon

## Real life troubleshooting

Damian Schenkelman, Auth0 @dschenkelman

### Meory leaks

Sawtooth pattern: gc periodically cleaning up

`--max-old-space-size` - node arg to up heap size when there is a problem

Drain connections when memory starts to spike - "nice" restarts

https://github.com/node-inspector/v8-profiler

Strings in the heap snapshot debugger can be useful - often contain context

forever-agent - for persistent connections (caused them problems when logging)

kinesis - stream service (aside)

### CPU bottlenecks

Used flame graphs

https://github.com/auth0/node-baas - same interface as bcrypt, but calls out to a service

## scaling state

Matteo Collina nearForm @matteocollina

Round robin LB - state changing ops w/ round robin imply some kind of single bottleneck.  Local caching doesn't work.

Application-level sharding.  But how to assign?  And what if topology changes?  Every peer needs to know about sibling health.

application-level sharding
https://www.npmjs.com/package/upring

upring-kv
upring-pubsub

server sent events

Read the dynamo paper!

All data laid out in consistent hashring.  "scalable weakly-consistent infection-style membership protocol" SWIM. farmesh by google.

message passing just like seneca.

Node streams passed around as messages in upring.

`upring.track` - moving events about to peers

upring-pub/sub

Uses?

* kv store
* cache
* pub/sub
* RT app
* discovery system
* anything with distributed state

Slides: https://mcollina.github.io/scaling-state/

Uses https://www.npmjs.com/package/bespoke for slides.

---

# Final keynotes

## Contributor panel

Bryan English (bengl), Anna Henningsen, William Kapke, Jeremiah Spenkpiel, James Snell moderator

`Buffer` constructor - a ruckus.  deprecation warning "didn't have messaging we wanted to have"

Talking about code & learn - they are trying to bring way more people into node core as contributors and mentorship, seem to be pretty happy with it.

"goal is to spread across as many people as possible, for redundancy..."

"I consider myself bilingual, I speak fluent nonsense"

`url.parse` totally fails the new URL working group test suite.  `url parse` and `new url` - don't know what to do with it.

## Google cloud

@justinbeckwith

Google has 2.5 billion lines of JS

google cloud vision API - extract text, generic label analysis (train a model), common logos

CLOUD CATS app.

https://npm.im/google-cloud

google BigQuery - SQL shim over backend, free public datasets

https://redit.com/r/listentothis

google cloud stackdriver debugger

## node @homeaway

Patrick Ritchie, HomeAway @pritchie

Began by consolidating vacation rental companies

2008 - built java platform

2013 - optimize to client-side building

Observe orient decide act <=> build measure learn

"node has the best open source ecosystem" - actually biggest...

docker, mesos, fastly is a CDN tool

## npm

Ashley Williams npm is a company, @ag_dubs

npm:
* 23 employess
* "wombat developers union" WDU (npm upside down)
* unpublishing has been removed from the registry (except within 24 hrs of initial publish):
  * instead, dissociate and deprecate process
* Uses "follower" pattern, "changes feed"
* greenkeeper.io - tool for keeping dependencies up to date
* yarn - https://yarnpkg.com (It looks really good!)
* https://replicate.npmjs.com - changes feed

Slides: http://bit.ly/npm-numbers

## State of the node

Rod Vagg, at nodesource, on TSC

https://groups.google.com/forum/#!forum/nodejs-sec - subscribe

## Road Forward

tracy hinds, education community manager, nodejs foundation

Within last couple years, nodejs had only 4 core contributors.

"tyranny of structurelessness" - lack of organization isn't actually structureless: implicit structure can be troubled.

iojs forked, and had: faster, predictable releases, open governance, brought lots of people in.  Joyent was corporate steward of nodejs, then announced the foundation.  Then months later, iojs merged back (late summer 2015, v4)

Maintaining the health of the project is a serious concern - they almost didn't actually survive, and (reading between the lines) thus the foundation.

Diversity training for the board/leadership to set the bar - outside perspective.  Then, peer mediation: speculation that it would help alleviate tension where arises. Then, trusted, moderated communications channel - IRC can't be official because code of conduct, logging  not really in effect.

Slack - not really open by default, but possible channel.

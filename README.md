<a href="http://www.wtfpl.net/"><img
       src="http://www.wtfpl.net/wp-content/uploads/2012/12/wtfpl-badge-4.png"
       width="80" height="15" alt="WTFPL" /></a>

# Kermit - the sloth

> Kermit is an extensible and feature rich web-scraper providing many useful extensions for
> automated data collection. It was built to lower the barrier of web-scraping complexity by providing
> clean abstractions and extension points for custom plugins.
> All complexity of request scheduling and coordination, resource discovery, url filtering, duplicate control etc.
> is dealt with using configurable extensions (yes, most of Kermit's core functionality is built based on its own
> plugin mechanism!)

> Kermit especially loves to free data from the web. If Kermit wasn't a sloth, she would be a pirate...yargh!


Kermit in a nutshell
========================

  * Provides composable abstraction to simplify extension with custom features. See [Extension](http://open-medicine-initiative.github.io/kermit/main/class/Extension.html)
  * Uses streaming API for handling of response data. Provides simple [Pipeline](http://open-medicine-initiative.github.io/kermit/main/class/Pipeline.html) abstraction to register [writable streams](https://nodejs.org/api/stream.html#stream_class_stream_writable) guarded by custom filters (content-type, length etc.)
  * Provides communication between components using [postal](https://www.npmjs.com/package/postal) as a shared message bus.
  * Comprehensive set of standard extensions for
    * Configurable **request filtering** (blacklist/whitelist) based on regular expressions on URLs or custom filter functions
    * Queueing system with configurable **rate limits** based on regular expression over URLs
    * Pluggable automated **resource discovery** (links, images, resources etc.) schedules all URLs found in html(-ish) documents
    * Pluggable **monitoring** to provide runtime statistics for traceability
    * Pluggable **REST based remote control** allows to interact with a scraper instance using the REST gui of your choice
    * **Lazy logging**: Log facilities only generate log messages if log level actually exists.
    * Pluggable **Offline Mode**: Download URLs to local storage to subsequently collect data offline from your local repositories (no rate limits! :-P )
  * Built around solid js libraries
    * [request](https://www.npmjs.com/package/request) and [socks5](https://www.npmjs.com/package/socks5-http-client)
    for calling the web - includes support for [Tor](https://www.torproject.org/) proxying
    * [lokijs](https://www.npmjs.com/package/lokijs) and [nedb](https://www.npmjs.com/package/nedb) as efficient backends for request queuing and URL backlog
    * [koa](https://www.npmjs.com/package/koa) as middleware for [serving pages](https://www.npmjs.com/package/koa-static) from local storage 
     and REST based remote control
    * [html-to-json](https://www.npmjs.com/package/html-to-json) and [cheerio](https://www.npmjs.com/package/cheerio) for efficient and syntax friendly dom traversals
    * [mitm](https://www.npmjs.com/package/mitm) for transparent redirecting of requests
    * [must](https://www.npmjs.com/package/must) for testing done right - well, admittedly the code base needs a bit more of testing  :-/ (WIP)
  * Thoroughly documented: Read the [API docs](https://open-medicine-initiative.github.io/kermit/main/index.html) generated with [codo](https://github.com/coffeedoc/codo) 


Kermit has been designed with care. Have a look at this simplified architectural diagram.

![Kermit architectural diagram](/doc/assets/arcitecture.png?raw=true , "Kermit architecture")

# Installation
    
Currently not available as npm library because it is still in beta. Use the github repository to install
and run your local copy. It is planned to release an npm version soon(ish) but I am still waiting for
user feedback (see section **Contribute**)
    
## Prerequisites
    
  * Running installation of Node.js and git
    > Recommendation: Use nvm and install 5.3 (not tested on versions below 5.0)
  * (optional) Installation of Tor  (if you want to collect data anonymously...and slowly, of course :)

## Setup
    
	$ npm install -g gulpjs/gulp-cli#4.0 -g
	$ git clone https://github.com/open-medicine-initiative/kermit.git
	$ cd kermit
	$ npm install
	$ npm test

# Usage

To execute a Kermit setup simply run

    $ node kermit.js --script=<relative-path-to-script>

For starters, here is an example of a simple setup that will download online content
to local storage (you can scrape the offline content later).

```coffeescript

Kermit = new Crawler
  name: "example"
  basedir : '/tmp/kermit'
  autostart: true
  extensions : [
    new ResourceDiscovery
    new Monitoring
    # new AutoShutdown # This would exit the process as soon as no work is left in queue
    # new Histogrammer # Histogrammer collects metadata on the visited URLs
    new RemoteControl # This will start a REST API for interacting with the crawler
    new RandomizedDelay # Introduce random pauses (reduce risk of bot detection)
      delays: [
        ratio: 1/2
        interval: 10000
        duration: 30000
      ]
    new OfflineStorage
      basedir: '/tmp/kermit/example'
    # new OfflineServer # Serve request from local file system for each previously downloaded URL  
    #  basedir : '/tmp/kermit/some/repository'
  ]
  options:
    Logging: logconf.production
    Streaming:
      agentOptions:
        maxSockets: 15
        keepAlive:true
        maxFreeSockets: 150
        keepAliveMsecs: 1000
    Queueing:
      # queue.items.db and queue.urls.db will be stored in /tmp/kermit/example
      filename : '/tmp/kermit/example/queue'
      limits : [
        {
          pattern :  /.*en.wikipedia\.org.*/
          to : 1
          per : 'second'
          max : 1
        }
      ]
    Filtering:
      allow : [
        /.*en.wikipedia\.org.*/
      ]
# Anything matching the whitelist will be visited
      deny : [
      ]

Kermit.execute "http://en.wikipedia.org/wiki/Web_scraping"

```

For a deeper understanding, read the [tutorial](http://open-medicine-initiative.github.io/kermit/main/index.html) 
regularly generated from [main.intro.md](./doc/main.intro.md). Also have a look at the [examples](./src/examples). 

# Documentation

The code ships with a lot of documentation and it is highly recommended to have a look at
the sources as well as the [API docs](https://open-medicine-initiative.github.io/kermit/main.index.html).

# Contribute

Because Kermit is currently in beta testing the most important contribution is feedback on functionality/bugs. 
Please provide log excerpts when submitting bug reports.
Another welcome contribution are more extensions. Create a gist of your extension
code and link it in the wiki page for [Extensions](https://github.com/open-medicine-initiative/kermit/wiki/Extensions) or create a PR.
Spread the word and invite more developers to use Kermit for freeing data.

# Ethnographical excourse on web scraping sloths
Kermit is not only a convenient web scraper, no, Kermit is also a sloth - and one remarkable sloth indeed! 
In fact, before Kermit it was not known that sloths actually do have a passion for working with data. 
They appear as uninterested and unexciting about any form of technology as one could possibly imagine. 
Only more careful studies have revealed that many of them have a second life as programmers, data analysts,
number crunchers or, as Kermit, data collectors.

Starting with this incredible discovery, more investigations have shown that there is a whole sub-culture
of technology enthusiasm among many other mammals, too. But only few of them do their data collection work as
carefully and patiently as Kermit. This is why you should seriously think about hiring Kermit for your
data collection jobs before contracting any of those human mammals that put expensive price
tags on their (mostly imaginary) CV's.

##License
For fun and personal enjoyment of rebellious acts the code is currently released under the [WTFPL](https://en.wikipedia.org/wiki/WTFPL)
(Do What The Fuck You Want To Public License). See the [License.md](License.md)
http://unlicense.org/
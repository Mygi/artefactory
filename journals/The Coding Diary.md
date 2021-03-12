# The Coding Diary

Journal entries for my life in Coding

### Sunday 18th October 2020

First of all I realised Microsoft have a released a new docker image for MSSQL on linux. Thought I should give this a try so that I don't need to keep rebooting to Windows. 

```bash
docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=18J3nS0n!Tw1c3' -p 1433:1433 -d mcr.microsoft.com/mssql/server:2017-latest
```

My ideal scenario is to use one development box purely for hosting docker environments - whilst using the other as my coding base. I don't really like giving up my performance on my development machine to the application hosting process.

`docker container ls`

```bash
docker exec -it <container_id|container_name> /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P 18J3nS0n!Tw1c3
```

Then added Azure Data Studio. All good and connected. Big thanks to the guide here: 

https://www.quackit.com/sql_server/mac/install_sql_server_on_a_mac.cfm

and updated dox from Microsoft here:

https://hub.docker.com/_/microsoft-mssql-server



**Choosing a framework and a language - once and for all**

I'ver been looking at Rust for a while and I've been thinking that it might fit me better than c++. One of the limitations that I found with c++ was the lack of package management and the build system - especially when it came to large projects. I tend to spend most of my backend dayas in c# at the moment and my front-end is exclusively in typescript. I'd love to settle on one great language for all purposes. It would make it that much easier to focus.

I also thought about what I wanted to build for.  I wanted scalability and performance. Performance was critical. For one reason, it would make hosting that much cheaper - especially for small development jobs or proof-of-concepts. Secondly, everything is simpler when you don't have to question the performance of your framework. If your application is slow - it is on you. Plain and simple. Rust has felt like a good fit even though the frameworks around it lack the maturity of others... but hey, I'm entirely self-interested here so I can help out on projects where the need requires. 

So I thought I'd look into existing options for Rust at the server side. That journey commenced at the Rust userlang forum and this question: https://users.rust-lang.org/t/rust-speed-in-server-side/42567(Rust Speed in server side).

This allowed me to follow the great tree of wisdom - and remember that techempower have benchmarking for this sort of thing here: https://www.techempower.com/benchmarks/#section=data-r19&hw=ph&test=composite. The Web Framework benchmarks are not without their limitations - the creator of the Haskell-driven web framework Servant provides an insightful post on Reddit's Haskell forum here: https://www.reddit.com/r/haskell/comments/6b3dlt/techempower_benchmarks_14_released_with_servant/. However, the benchmarks make it abundantly clear that c++/c and Rust are currently leading the pack. I should note that dotnetCore (my current professional framework of choice) performed well, especially considering everything you get in the box. 

It looked like I would need to run postgresql if I was to achieve anything close to the recorded benchmarks. However, that also made me curious about a write-based normalised database for the working model vs a read-based non-sql data store for retrieving the published model. Once you add complexity of queries, how would they compare? I'll likely need to follow that up at some future point-in-time.

So now I am curious about the look and feel of these top performers. Actix, drogon and lithium looked like good ones to check out. So I thought I would have a look at building my cellular automata concept via each of them.

I also wanted to look at depper support for things like caching, websockets, redis and ORMs. I think I would find it hard to drop the ORM even if that incurred the performance hit. 

I write in Angular but I really want to see what Web Assembly can do for me today.

Stack 1 (RUST):

* Seed (Front) via WebAssemply
* Actix and actin-web for the API and webserver
* DIESEL for ORM
* Postgresql for DB

Stack 2 (C++)

* QT WebAssembly (Front) VIA EMSCRIPTEN
* Drogon for API, Webserver and ORM
* Postgresql for DB

Stack 3 (C#):

* Blazor (Front)
* DotnetCore for API
* Kestral for Server
* Dapper  vs Linq2DB for ORM
* postgresql for database

NB: Worth a look ASSEMBLYSCRIPT (https://github.com/AssemblyScript/assemblyscript)

GOOD ARTICLE：https://www.webassemblyman.com/webassembly_front_end_web_development.html

## Sunday 25th October

I have actix and Rust running. I also set up Postgres. That was pretty straightforward. There are a few steps that I need to take next (that I want):

* Json serializer
* Diesel ORM with postgresql
* Add Protobuf support
* Add Web Socket support
* Add Redis Support.
* Add seeder to the front and build with Angular (existing work).
* Tets it all out.



## Tuesday 27th October

Displays setup: MAC

LHS: 2880 x 1800

RHS: 2560 x 1440

= 5440 x 1440 + (2880 * 360)

= 7.83 million + 1.036 Million = 8.866 Million

Displays Setup Windows:

LHS: 3840 x 2160

RHS: 1920 x 1200

= 5760 x 1200 + *960 * 3840)

= 6.91 million +  3.68 million = 10.59 Million

= 19.456 Million pixels

What would be really could would be to tile a 4:4 or 16:9 ration 20MP photo to genuinely look at it.

Can I render 20 million automatons? - How would I make it circular?

Then I could render a full CT image and sit inside it!

Then I would need multiple cameras - maybe cheap phones with 6mb Video Capturing? Get the correct focal length.



## November 8th 2020

The first development result will be a simple blu setup using actix-web. This works by providing the following:

* Markdown for content
* JSON for index entries, meta data and front cover.

In this way the entire data structure can be committed to Github. 

The intent is that the JSON will likely end up in postGresql whilst the document strcuture will reside in MongoDB. Ideally we would then use neo4j to link content bubbles together. 

However tis initial undertaking is sizeable. Importantly we need to translate markdown to html - which could be done in the client or the API. Additionally, we could set the entire article in a single markdown file or break it up. There are pros and cons for each which will need to be identified. 

This is before we consider a client.

The client is intended to be RUST Wasm - with some basic Angular extensions. This is the product to be released first and is due by the end of November.

Then we can see if we still think the testing of other frameworks is still required. Considerations like GPU and MPI will be key - as compared to package management, ease of use. We may need to abstract c++ libraries for MPI or PU usage. Perforamnce is still key.




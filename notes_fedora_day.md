## Fedora day

Notes for [NEFUG fedora-training day](https://wiki.duraspace.org/display/Events/Northeast+Fedora+User+Group+Meeting%3A+11-12+May+2015)


### fedora overview

- fedora acronym; existed as concept before software

- is an idea, software, community

- supports complex semantic relationships between objects inside and outside the repository

- middleware; interoperability a primary goal

- exposing/connecting content
    - atomic resources
    - rdf-based metadata using linked data
    - restful api with native rdf response format

- f4 project goals
    - 2012
    - performance, storage-options, research data management, linked open-data support
    - improved platform for developers
        - tdd, sprints, documentation

- whole new codebase from f3

- priorities: linked-data high; performance less so because other tools can improve performance (ie caching)
    - newer priorities: use of standards & APIs


#### Component stack

- rest framework
    - fedora services
        - [modeshape](http://modeshape.jboss.org)
                - "ModeShape is a distributed, hierarchical, transactional, and consistent data store with support for queries, full-text search, events, versioning, references, and flexible and dynamic schemas."
            - [infinispan](http://en.wikipedia.org/wiki/Infinispan)
                    - "Infinispan is a distributed cache and key-value NoSQL data store software developed by Red Hat. Java applications can embed it as library, use it as a service in WildFly[1] or any non-java applications[2] can use it as remote service through TCP/IP.[3]"
                - storage (objects & datastreams)


#### core features & standards

- main
    - crud-ldp
    - versioning - [memento](http://www.mementoweb.org/guide/)?
    - authorization - WebAC?
    - transactions
    - fixity
    - import/export - rdf export?

- Versioning
    - Can be created on resources with API call.
    - So there is versioning -- it's just that the api call uses a fedora-specific syntax -- may adopt a standard syntax

- Authorization
    - off by default
    - based on role-based ACL -- can be inherited -- parent objects will be examined for info
    - XACML still available allowing very detailed authorization

- Transactions
    - interesting, can _improve_ performance, since certain operations won't happen until they have to

- Fixity
    - handled by a fixity api, called manually

- Export/Import
    - some folk not happy that modeshape stores non-human-readable format -- there is capability for maintaining a human readable format
    - Apache Camel may be a pattern to set up a route for outputting human-readable output

- Backup/Restore
    - whole thing only at moment (not partial)
    - will see how it plays out in terms of implementing partial backup/restore

#### Non core

- pluggable components -- common patterns
    - rest service, authorization engine, oai-pmh service
    - oai-pmh module put together by community person
    - SWORDv2 module put together by community person

- external components
    - consume and act off repository messages
    - [Apache Camel](http://camel.apache.org)
        - "Camel empowers you to define routing and mediation rules in a variety of domain-specific languages..."
        - camel-to-solr already set up on training vagrant VM
    - indexing - content can be assigned the rdf:type propety 'indexable' to filter from non-indexable content
        - good example -- custom indexing code is hundreds of lines of code, whereas the apache camel hookup is maybe 50 lines of code
        - solr well-established
    - triplestore
        - fuseki and sesame have been tested
        - any triplestore that supports sparql-update can be plugged in
        - fuseki with camel/sesame configured in training VM
    - audit service
        - optional service generates triples for audit events & stores them in triple-store
        - both internal repository events and events from external sources can be recorded


#### LDP

- ldp & linked data

- fc3 -- fc4:
    - objects & datastreams -- resources
    - objects -- containers
    - datastreams -- binaries

- containers & binaries are resources
    - container resources can have both conainers and binaries as resources

- properties
    - resources have a number of properties, expressed as rdf triples
    - name-value pairs are translated to rdf on rest-api responses

- content-models modeled using properties and types
    - fc3 - image-content model with many datastreams -- fc4 content-types
    - pcdm


#### Performance & Scalability

- fc3: one option
- fc4: lots of options
    - projection -- fedora can federate itself over an external filesystem
        - objects never ingested, but fedora (read-only) operations happen

- fcrepo-storage-policy
    - a module that allows objects of a particular type to be stored in location A, whereas other objects are stored in location B
    - can also be done with particular properties
    - use case: large tiffs stored on amazon; derivitives stored locally
        - bpl wants to store tiffs on slow storage and have jp2s and jpgs on faster access storage
            - this is mostly available now -- but only via mimetypes

- metrics
    - terabyte via rest api
    - 16 million objects via [projection](https://wiki.duraspace.org/display/FEDORA40/Filesystem+Federation)
    - 10 million objects via rest api

- transactions improve performance

- clustering tested successfully -- can use load-balancer


#### Migrating from v3 to v4

- difference, migration tools, possibilities for enhancing data

- differences
    - objects and resources
        - f3, foxml objects; incline xml and xml datastreams
        - f4, web resources (containers and binaries) -- no more inline xml - would convert it to binaries
            - and repo is natively a hierarchy descending from a root resource
                - due to modeshape -- get best performance from using a tree structure -- don't want to have a root node with 10,000 objects
                - f4 autogenerates default balanced hierarchy for you (so default tree not meant to be meaningful; meaning via semantic properties) -- easiest just to use default tree hierarchy and not deal with that
    - flesystem
        - f3: objects and datastreams in pairtree
        - f4: containers dir and binaries dir; containers in db; binaries in pairtree
            - so not as human-readable
    - pid vs path
        - f3: pid; can't be altered
        - f4: resources have an internal uuid, and have a repo path, which can be user-defined or generated via a path-minter
            - likely pattern -- have a name-service where the resource-path is mapped to the service id

- mapping properties - descriptive
    - pid
        - f3 pid; f4 dcterms:identifier; eg 'prefix:123'
    - state
        - f3 state; f4 fedora3:objeState; eg 'active'
    - label
        - f3 label; f4 dcterms: title; eg 'some title'
    - created date & modified date
        - f3 createdDate; f4 :created -- but will be autocreated on migration, so may want to add a term in descriptive metadata
    - owner id also

- mapping properties - datasteams
    - pattern below: property -- f3 term -- f4 term -- example
    - dsid -- id -- dcterms:identifer -- prefix:123
    - state -- state -- fedora3:objectState -- 'active' -- ben noted f4 doesn't act on this anymore -- it would be application-logic that would deal with this
    - versionable
    - label
    - created/modified
    - mimetype -- MIMETYPE -- fedora:mimeType -- image/jpg
    - size -- SIZE -- premis:hasSize -- 50000

- two tools
    - fedora-based 'migration-utils'
    - hydra based tool

- migration-utils
    - assumes foxml is complete representation of object
    - foxml migration doesn't require the fedora3 repo software to be running
    - considerations
        - non-repo data (config, global xacml policies) would need special handling
    - process
        - process foxml documents
        - migrate pids
        - convert any inline xml to managed xml or rdf properties
        - datastreams to binaries or rdf properties
        - convert/map access controls to f4
        - migrate versions

- fedora-migrate hydra tool
    - need working hydra app using fedora4
    - all models defined in your hydra/f3 instance
    - penn-state scholarsphere is the first in-production site using f4

---


### hands-on fedora

        $ vagrant up --no-provison
        $ vagrant ssh

- now i can go to `http://localhost:8080/fcrepo/` and port-forwarding gives me the ubuntu VM's fedora.

- architecture: me -> ldp layer f4 -> messages go to camel ->A-> solr, and ->B->Triplestore/Fuseki(audit-service & sparql)

- goal:
    - objects
        - cover
            - files
                - cover.jpg
                - cover.tiff
        - book (bonus)
            - members
                - coverProxy

- f4 has notion of top-level object, here 'objects'

- in above, files is a direct-container; members is an indirect-container

- note to self -- this is already useful, because I've already begun thinking about f4 in terms of pcdm instead of the true ldp framework

- final relationships (subject-predicate-object)
    - cover pdcdm:hasFile cover.jpg
    - cover pcdm:hasFile cover.tiff
    - book pcdm:hasMember cover

- resource being requested is almost always the subject

- in this training we're naming the identifier -- initially 'objects', then a child 'cover' -- but programmatically we won't name the identifier, so fedora will auto-balance the tree structure.

- added

        PREFIX pcdm: <http://pcdm.org/models#>
        INSERT { <> a pcdm:Object } WHERE { }

    ...to the update-properties form field -- left in the other prefix default entries, and after clicking 'update', the pcdm addition now appears in two areas: 'fedora:mixinTypes' and 'rdf:type'

- an ldp 'direct-container' must have 2 things: a membershipResource property and a hasMemberRelation property

- an ldp 'indirect-container' has a proxyFor relationship
    - <> ldp:insertedContentRelation ore:proxyFor

- after adding cover.jpg, i'm at url: <http://localhost:8080/fcrepo/rest/objects/cover/files/cover.jpg/fcr:metadata>

- if I go back to http://localhost:8080/fcrepo/rest/objects/cover, it knows about cover.jpg

- another example of what we did on the wiki: 'https://wiki.duraspace.org/display/FEDORA4x/LDP-PCDM-F4+In+Action'

- demo of running triplestore: 'http://localhost:3030/'
    - sparql query

            ?s ?p ?o where
            { ?s ?p ?o }

- solr, http://localhost:8080/solr/#/

---


### integration patterns with fedora

Aaron Coburn -- Apache Camel

- "asynchronous, distributed architecture is complex -- why do it?"

- triplestore
    - f3, mulgara 3store embedded
    - f4, external 3store needed

- software stack
    - many simple and intermediate stacks are synchronous
    - in fedora, web-ui would have to wait for fedora to update a 3store, and then solr -- and even more -- so waiting would take too long.
        - further, failure chance increases with more components

- messaging is a solution to allow asynchronous processing
    - jargon: jms, soa, eip (enterprise integration patterns), esb (enterprise service bus), message oriented middleware, event driven architecture, PubSub, Message Broker, OSGi

- camel, mule, there are a lot of these message-handlers / workflow-routers

- simple to run a listener in any language, but once you have a lot, the _management_ of them (logging, starting them up, etc) takes a lot of work -- that's where Camel & its ilk comes in

- camel uses a DSL

- patterns in messaging:
    - direct routing (from -> to)
    - transformation ( from -> process -> to) -- transformations can be in Java or xml (jython?)
    - filter ( from -> filter -> to )

- camel can have a command to get a fedora message and multicast it to x number of routes

- deploying camel
    - can use a war file
    - aaron likes [karaf](http://karaf.apache.org) a tiny jvm that lets you have lots of little services running independently -- as opposed to a war file that runs in the same container as other services like solr

- Camel in Action, by Ibsen & Anstey

- fedora activeMQ defaults to using a 'topic' instead of a 'queue'. If you care about ultimate reliability, use a queue.
    - has nice monitoring tools using [hawtio](http://hawt.io)

- aaron's documentation
    - [camel-routes](https://acdc.amherst.edu/wiki/solr-routes)
    - [karaf info](https://acdc.amherst.edu/wiki/karaf_runtime)

- [fedora camel code](https://github.com/fcrepo4-labs/fcrepo-camel-toolbox)

---


### my questions

- restful api with native rdf response format
    - [json-ld](http://json-ld.org) too?
    - transformations on output possible via [LDPath](http://marmotta.apache.org/ldpath/language.html)?

- no versioning currently? -- no, there is, it's just that the current syntax may be replaced with a standard versioning syntax.

- audit service says tracks internal repo events plus external events -- what are some common external events? Oh, the external events are things triggered by a user, internal are things that fedora triggers itself internally

- projection -- if a change is made to a projected resource, does fedora auditing/versioning get triggered? -- No

- karaf seems really cool -- what other kinds of services can be run in it that traditionally use jetty or tomcat?

---

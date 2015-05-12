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


### Component stack

- rest framework
    - fedora services
        - [modeshape](http://modeshape.jboss.org)
                - "ModeShape is a distributed, hierarchical, transactional, and consistent data store with support for queries, full-text search, events, versioning, references, and flexible and dynamic schemas."
            - [infinispan](http://en.wikipedia.org/wiki/Infinispan)
                    - "Infinispan is a distributed cache and key-value NoSQL data store software developed by Red Hat. Java applications can embed it as library, use it as a service in WildFly[1] or any non-java applications[2] can use it as remote service through TCP/IP.[3]"
                - storage (objects & datastreams)


### core features & standards

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

### Non core

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


### LDP

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


### Performance & Scalability

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


### Migrating from v3 to v4

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


---


### my questions

- restful api with native rdf response format
    - [json-ld](http://json-ld.org) too?
    - transformations on output possible via [LDPath](http://marmotta.apache.org/ldpath/language.html)?

- no versioning currently? -- no, there is, it's just that the current syntax may be replaced with a standard versioning syntax.

- audit service says tracks internal repo events plus external events -- what are some common external events?

- projection -- if a change is made to a projected resource, does fedora auditing/versioning get triggered? -- No

---

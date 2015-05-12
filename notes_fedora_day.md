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


---


### my questions

- restful api with native rdf response format
    - [json-ld](http://json-ld.org) too?
    - transformations on output possible via [LDPath](http://marmotta.apache.org/ldpath/language.html)?

- no versioning currently? -- no, there is, it's just that the current syntax may be replaced with a standard versioning syntax.

- audit service says tracks internal repo events plus external events -- what are some common external events?

---

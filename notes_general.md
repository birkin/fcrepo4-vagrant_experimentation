## General

### PCDM

- Overview
    - flexible, extensible model that can underlie a wide array of repo and DAMS apps
    - primary goal interoperability
    - built to satisfy simpel and complex use cases

- ontology is on github

- combines common ontology with LDP interaction
    - specifies relationships between types of objects

- tomorrow
    - fedora: an ldp server

- you can use PCDM without thinking about LDP at all, but it facilitates LDP usage if desired

- collections -> objects -> files
    - access on any of those
    - desicriptive at collection and object
    - technical at file

- ordering in pcdm...
    - pdcm pages in a book have no order
    - page gets associated with an ore:Proxy object -- and the ore:Proxy objects have an iana:next/previous attribute
        - technically, don't _need_ the ore:Proxy linkage to the book, but it makes book-level sparkl queries simpler

- pcdm examples on [wiki](https://wiki.duraspace.org/display/FF/PCDM+Examples)
    - talk examples not on wiki
    - sufia collections -> sufiaWork -> GenericFile -> Content/Thumbnail/ExtractedText files
    - map -- no collection object, just a few objects each with files hanging off
        - vector object -> shape-files hanging off
        - scanned object
        - postcards -- like book with ordering only to front/back

### Fedora 4 overview

- Community Participation
    - 3 to 4 migrations
    - maintenance sprints

- Long-term
    - asynchronous storage
    - [LDP](http://en.wikipedia.org/wiki/Linked_Data_Platform)
    - community standards
        - part of reason for this is to use supported standard packages for certain services, so as to enable fedora-specific development focus on more repo-specific services
        - interesting: one vision for fedora to be able to be the back-end for a linked-data server that doesn't know anything about fedora specifically but can make http api calls
    - shrinking codebase -- related to above focus on repo-specifc code -- getting away frome 'bespoke' code.

- Short-term
    - finalized first round of upgration pilots by end of may

- Audit service
    - community-developed

- PCDM
    - collections --> objects --> files (note: 'file' in this sense is a file _object_, with a bit-stream, _and_ optional access/technical metadata)
    - interoperability

- Columbia's one of the upgrade pilots

- tools (on github)
    - migration-utils( generic + islandora)
    - fedora-migrate (Hydra)

---

## My questions

- is the fuseki triple store an inherent part of fedora4? Or an add-on providing capability if desired.
    - B: add-on; said it's not part of the one-click install

- For Ben: at Brown, what would be involved in creating a provisioning-ish script to export x random objects (say, 100 to 1000) (along with internally associated objects) to a fully running fedora3 instance mirroring ours.
    - ok, sounds like one could install fedora3, copy over foxml-directory and data-binaries, and run the 'fedora-rebuild.sh' script which would build the database.
    - our risearch triplestore is turned off, so now I'm understanding that the triplestore isn't essential for fedora storage, rather, if turned on, the bundled gets autopopulated from the foxml as a convenience service.
    - also, there is an api export for archival purposes. In our 3.7 version, there is a bug for large files (fixed in 3.8), so I'd have to avoid large files.

- When running either of the migration scripts, do they try to duplicate existing models directly? Or are the new models some version of the Portland Common Data Model?

- For A., technical metadata, does some of that data blur into descriptive metadata? I'm thinking of exif data -- i've seen descriptive values from adobe lightroom flow into the exif data.
    - yes, can happen, but no prob, can just pull out the descriptive metadata & populate up the stack

- Also for A., what are the objections to PCDM?

---

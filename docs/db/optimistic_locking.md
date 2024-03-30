```shell


Initial State:
+------------------+     +------------------+
| Entity in DB     |     | Version in DB    |
|------------------|     |------------------|
| Document Content |     |        1         |
+------------------+     +------------------+

Transaction A and B start -> Both read the same entity with version 1

+------------------+     +------------------+     +------------------+ 
| Transaction A    |     | Entity in DB     |     | Transaction B    |
| Reads version: 1 |     | Version: 1       |     | Reads version: 1 |
+------------------+     +------------------+     +------------------+

Transaction A updates the entity first:
+------------------------------------------------------------+
| Transaction A completes its update                         |
| -> Checks version (1 == 1) -> Proceeds with update         |
| -> Entity's version in DB is incremented to 2              |
+------------------------------------------------------------+
                +------------------+
                | Version in DB    |
                |       2          |
                +------------------+

Transaction B tries to update the entity:
+------------------------------------------------------------+
| Transaction B attempts to update                           |
| -> Checks version (1 != 2) -> Conflict detected!           |
| -> Update is not applied                                   |
+------------------------------------------------------------+
                +------------------+     +---------------------------+
                | Version in DB    |     | Transaction B is informed |
                |       2          |     | of the version conflict    |
                +------------------+     +---------------------------+

Handling the Conflict:
+------------------------------------------------------------+
| Transaction B can now decide how to proceed:               |
| -> Reload the updated entity and view the latest changes   |
| -> Merge their changes with the new version and retry      |
+------------------------------------------------------------+


```
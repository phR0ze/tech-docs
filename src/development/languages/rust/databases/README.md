# Databases <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Documenting my learning experience with Rust Databases

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Index](#index)
  * [Foreign Key](#foreign-key)

### Linked pages
- [SeaORM](sea_orm/README.md)
- [SQLx](sqlx/README.md)

## Overview

### Index
In relational databases an index is an additional data structure that helps improve the performance 
of a query. SQLite uses B-tree for organizing indexes. 

* [SQLite Index docs](https://www.sqlitetutorial.net/sqlite-index/)

### Foreign Key
Links data in one table to a parent table such that the parent data can't be removed until the child 
data is first removed. Likewise data can't be inserted into the child referring to a parent that 
doesn't exist. It essentially lets the database know about the releationship so it can set up some 
contraints that the database will enforce.

* [SQLite Foreign Key docs](https://www.sqlitetutorial.net/sqlite-foreign-key/)

<!-- 
vim: ts=2:sw=2:sts=2
-->

# What is IndexedDB

IndexedDB 是浏览器中的数据库

# about this doc

本文档会介绍 IndexedDB 的 concepts and terminology.
- 如果想知道怎么使用 API，请参考[Using IndexedDB](./tutorial.md)
- 如果想查阅详细的 API 文档，请参考[IndexedDB API](./api.md)
- 如果想了解浏览器如何存储数据以及 IndexedDB 的限制，请参考[background](./background.md)

# 概述

- IndexedDB is an indexed table system, not a relational database access system.
- IndexedDB lets you store and retrieve objects that are indexed with a "key".
- All changes that you make to the database happen within transactions.
- IndexedDB follows [same-origin policy](http://www.w3.org/Security/wiki/Same_Origin_Policy).
- IndexedDB is an asynchronous api that can be used in most contexts.
- 竞争者 WebSQL 死于 2010-11-18

# 基本概念

## Key-value pairs

- IndexedDB database store key-value pairs
- The values can be complex structured objects, and keys can be properties of those objects.
- You can create indexes that use any property of the objects for quick searching, as well as sorted enumeration.

## Transactional database model

- IndexedDB is built on a transactional database model
- Everything you do in IndexedDB always happens in the context of a [transaction](#transaction)
- All objects from IndexedDB API are tied to a particular transaction.
- Transactions auto-commit and cannot be committed manually.
- Attempting to use a transaction after it has completed throws exceptions.

## Asynchronous API

- You "request" that a database operation happens.
- You get notified by a DOM event when the operation finishes, and the type of event you get lets you know if the operation successed or failed.

## IndexedDB Requests

- Request objects are generated as a result of doing some database operation.
- They receive the success or failure DOM events.
- They have `onsuccess` and `onerror` properties, and you can call `addEventListener()` and `removeEventListener()` on them.
- They also have `readyState`, `result` and `errorCode` properties that tell you the status of the request.
- The content of `result` property depends on how the request is generated.

## DOM events

- IndexedDB uses DOM events to notify you when results are available.
- DOM events always have a `type` property (mostly `"success"` or `"error"`).
- DOM events also have a `target` property, which is the corresponding request object.
- Success events don't bubble up and can't be canceled.
- Error events do bubble and can be cancelled.
- Error events abort whatever transactions they're running in, unless they are cancelled.

## Object-oriented

- [object database](https://en.wikipedia.org/wiki/Object_database)
- You create an object store for a type of data and simply persist JavaScript objects to that store.
- Each object store can have a collection of indexes that makes it efficient to query and iterate across.

## NoSQL

- [NoSQL](https://en.wikipedia.org/wiki/NoSQL)
- It uses queries on an index that produces a cursor, which you use to iterate across the result set.

## Same-origin

- An origin is the domain, application layer protocol, and port of a URL.

# Definitions

此章节介绍 IndexedDB API 用到的术语

## Database

### database

A repository of infomation, comprising one or more [object stores](#objectstore). Each database must have the following:
- Name, String identifier
- Version, initial 1

### object store

- Object store holds records, which are key-value pairs.
- Records are sorted according to the [keys](#key) in an ascending order.
- Every object store have a name that is unique within its database.
- Object store can optionally have a [key generator](#keygenerator) and a [key path](#keypath).
- If the object sotre has a key path, it is using [in-line keys](#in-linekey); otherwise, it is using [out-of-line keys](#out-of-linekey).

### version

- When a database is first created, its version is 1.
- The only way to change the version is by opening it with a greater version than the current one.
    - This will start a `versionchange` transaction and fire an `upgradeneeded` event.
    - The only place where the schema of the database can be updated is inside the handler of that event.

### database connection

Its created by opening a databse. A database can have multiple connections at the same time.

### transaction

- An atomic set of data-access and data-modification operations on a database.
- Any reading or changing of records in a database must happen in a transaction.
- A database connection can have several active transaction at a time.
- Three modes: `readwrite`, `readonly`, and `versionchange`.
- The [scope](#scope) of a transaction is defined at creation, which determines which object sotres the transaction can interact with.
- Writing transactions can not have overlapping scopes, however, reading transactions can.

### request

Every request represents one read or write operation.

### index

- An index is a specialized object store for looking up record in another object store, called the referenced object store.
- The value part of its records is the key part of a record in the referenced object store.
- The records in an index are automatically populated whenever records in the referenced object store are inserted, updated, or deleted.
- Alternatively, you can also look up records in an object store using [key](#key).

## Key and value

### key

- A value by which records are organized and retrieved in object store.
- Ojbect store can derive the key from one of three sources: a [key generator](#keygenerator), a [key path](#keypath), or an explicitly specified value.
- The key must be greator than the one before it.
- Each record in an object store must have a key that is unique within the object store, so you cannot have multiple records with the same key.
- A key can be: string, date, float, and array.
- Alternatively, you can look up records in an object store using [index](#index);

### key generator

- A mechanism for producing new keys in an ordered sequence.
- If an object store does not have a key generaotr, then the application must provide keys for records being stored.

### key path

- Defines where the browser should extract the key from in the object store or index.
- A valid key path can include one of the following:
    - an empty string
    - a JavaScript identifier
    - multiple JavaScript identifiers seperated by periods
    - an array containing any of those

### in-line key

- A key that is stored as part of the record.
- It is found using a [key path](#keypath).
- An in-line key can be generated using a [generator](#keygenerator). After the key has been generated, it can then be stored in the record using the key path.

### out-of-line key

- A key that is stored separately from the record.

### value

- Each record has a value, which could be any valid JavaScript value, including boolean, number, string, date, object, array, regexp, undefined, and null.
- When an object or array is stored, the properties and values in that object or array can also be anything that is a valid value.
- Blobs and files can be stored.

## Range and scope

### scope

- The set of [object stores](#objectstore) and [indexes](#index) to which a [transaction](#transaction) applies.
- The scopes of `readonly` transactions can overlap.
- The scopes of `writing` transactions cannot overlap.
- You can still start several `writing` transactions with the same scope at the same time, but they just queue up and execute one after another.

### key range

- Records can be retrieved from object stores and indexes using keys or a range of keys.
- You can limit or filter the range using lower and upper bounds.

### cursor

- A mechanism for iterating over multiple records within a key range.
- The cursor has a source that indicates which index or object store it is iterating.
- It has a position within the range, and moves in a direction that is increasing or decreasing in the order of record keys.

# Limitations

- Full text searching. The API does not have an `LIKE` operator.
# About this document

本文介绍如何使用 IndexedDB API

# Basic pattern

1. Open a database.
2. Create an object store in the database.
3. Start a transaction and make a request to do some database operations, like adding or retrieving data.
4. Wait for the operation to complete by listening to DOM event.
5. Do something with the results.

# Creating and structuring the store

## Open a database

- Most asynchronous IndexedDB APIs return an `IDBRequest` object.
    - Add success and error handlers to the request objects.
        - error event is a DOM event whose `type` property is set to `"error"`
        - success event is a DOM event whose `type` property is set to `"success"`
    - The `event` has a `target` property set to the request object, which has `error` or `result` properties contains the corresponding error or result.
- As for open request, you can listen to the `onupgradeneeded` event
    - it's the only place you can alter the sturcture of the database.
        - create or delete object stores or indexes.
    - in the callback
        - only create new object stores or delete object stores from previous version
        - if you need to update an existing object store, you must delete and then create it.
            - Note this will delete the information in the object store
            - if you need save it, you can read it out before upgrading the database.
        - create an object store with a name that already exists or try delete an object store with a name that does not already exist will throw an error.
    - If the `onupgradeneeded` event exits successfully, the `onsuccess` handler will then be triggered.

```javascript
// IDBOpenDBRequest
var request = window.indexedDB.open('foo-db', 1);
request.onerror = function(event) {
    console.log(event);
    console.log(event.type);
    console.log(event.target);
    console.log(event.target.error);
};
request.onsuccess = function(event) {
    console.log(event);
    console.log(event.type);
    console.log(event.target);
    console.log(event.target.result);
};
request.onupgradeneeded = function(event) {
    console.log(event);
    console.log(event.type);
    console.log(event.target);
    console.log(event.target.result);

    var db = event.target.result,
        os = db.createObjectStore('user', {keyPath: 'account'});
    os.createIndex('nick', 'nick', {unique:false});
    os.createIndex('email', 'email', {unique:true});
};
```

- The second parameter of `open` is database version, which determines the database schema - the object stores in the database and their structure.
    - If the database doesn't exist, it is created by the `open` operation, then an `onupgradeneeded` event is triggered and you create the database schema in its handler.
    - If the database does exist but you specify an upgraded version number, an `onupgradeneeded` event is triggered right away, allowing you to provide an upgraded schema in its handler.
- The version number is an `unsigned long long` number.

## Sturcturing the database

- A database can contains any number of object stores.
- Whenever a record is stored in an object store, it is associated with a key.

Different ways the keys are supplied:

Key Path<br>(keyPath)|Key Generator<br>(autoIncrement)|Description
-----------------|----------------------------|-----------
no|no|can hold any kind of value. You must supply a key when added a new value.
yes|no|can only hold JavaScript objects, which must have a property with the same name as the keyPath
no|yes|can hold any kind of value. The key is generated automatically, or you can supply a key.
yes|yes|can only hold JavaScript objects. If the obj has a keyPath property, then use it as the key; if not, generate one, store it in the keyPath property.

- You can also create indexes on any object store, provided the object store holds objects, not primitives.
- Additionally, indexes can enforce simple constraints on the records.
    - By setting the unique flag, the index ensures that no two objects stored having the same value for the index's keyPath.

# Transaction

- Before you can do anything, you must start a transaction.
- Transaction comes from the database object, you have to specify
    - which object stores you want the transaction to span.
        - if just one, pass string
        - if multiple, pass array
    - transaction mode: `readonly`(default), `readwrite`, `versionchange`.
- Then you can access object stores and make your requests on the transaction.
    - When the request succeeds, you will have opportunity to make another request.
    - If you return to the main event loop, the transaction will be inactive.
- Transaction can receive DOM events of type: `error`, `abort`, and `complete`.
    - error events bubbles, so a transaction receives error events from any requests that are generated from it.
    - the default behavior of an error is to abort the transaction (which will be rolled back) unless you call `stopPropagation()` on the error event.
    - if you don't handle an error event or if you call `abort()` on the transaction, the transaction will be rolled back and an abort event will be fired.
    - If all pending requests have completed, you'll get a success event.

# add data

```javascript
function addUsers(users) {
    var t = db.transaction('user', 'readwrite'),
        os = t.objectStore('user');
    t.onerror = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
        console.log(event.target.error);
    };
    t.onabort = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
    };
    t.oncomplete = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
    };
    users.forEach(function(user) {
        os.add(user);
    });
}
function addUser(user) {
    var t = db.transaction('user', 'readwrite'),
        os = t.objectStore('user');
    t.onerror = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
        console.log(event.target.error);
    };
    t.onabort = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
    };
    t.oncomplete = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
    };
    os.add(user);
}
```

# update data

```javascript
function updateUser(user) {
    console.info('updateUser');
    var t = db.transaction('user', 'readwrite'),
        os = t.objectStore('user');
    t.onerror = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
        console.log(event.target.error);
    };
    t.onabort = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
    };
    t.oncomplete = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
    };
    os.put(user);
}
```

# remove data

```javascript
function deleteUser(account) {
    console.info('deleteUser');
    var t = db.transaction('user', 'readwrite'),
        os = t.objectStore('user');
    t.onerror = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
        console.log(event.target.error);
    };
    t.onabort = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
    };
    t.oncomplete = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
    };
    os.delete(account);
}
```

# get data

```javascript
function getUser(account, cb) {
    console.info('getUser');
    var t = db.transaction('user'),
        os = t.objectStore('user'),
        user = null;
    t.onerror = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
        console.log(event.target.error);

        cb(user);
    };
    t.onabort = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
    };
    t.oncomplete = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
        console.log(event.target.result);

        cb(user);
    };
    os.get(account).onsuccess = function(event) {
        user = event.target.result;
    };
}
function getUsers(cb) {
    console.info('getUsers');
    var t = page.db.transaction('user'),
        os = t.objectStore('user'),
        users = [];
    t.onerror = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
        console.log(event.target.error);

        cb(users);
    };
    t.onabort = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
    };
    t.oncomplete = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);

        cb(users);
    };
    os.openCursor().onsuccess = function(event) {
        var cursor = event.target.result;
        if (cursor) {
            users.push(cursor.value);
            cursor.continue();
        }
    };
}
```

# use index

```javascript
// get the one with the lowest key
function getFirstNick(nick, cb) {
    var t = db.transaction('user'),
        os = t.objectStore('user'),
        i = os.index('nick'),
        user = null;
    t.onerror = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
        console.log(event.target.error);

        cb(user);
    };
    t.onabort = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
    };
    t.oncomplete = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);

        cb(user);
    };
    i.get(nick).onsuccess = function(event) {
        user = event.target.result;
    };
}
function getByNick(nick, cb) {
    var t = db.transaction('user'),
        os = t.objectStore('user'),
        i = os.index('nick'),
        kr = IDBKeyRange.only('nick'),
        users = [];
    t.onerror = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);
        console.log(event.target.error);

        cb(users);
    };
    t.onabort = function(event) {
        console.log(event);
        cosnole.log(event.type);
        console.log(event.target);
    };
    t.oncomplete = function(event) {
        console.log(event);
        console.log(event.type);
        console.log(event.target);

        cb(users);
    };
    i.openCusror(kr/* , 'prev' */).onsuccess = function(event) {
        var cursor = event.target.result;
        if (cursor) {
            users.push(cursor.value);
            cursor.continue();
        }
    };
}
```

# Version changes while a web app is opened in another tab

# Security

# Warning About Browser Shutdown

# Locale-aware sorting

# See also

- Source: [Using IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB)
- Concepts: [Basic Concepts Behind IndexedDB](basic.md)
- Limit: [limit](limit.md)
- Reference: [IndexedDB API reference](api.md)
- Specification: [IndexedDB Database API Specification](http://www.w3.org/TR/IndexedDB/)

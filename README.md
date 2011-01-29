<!-- -*- mode: markdown ; coding: utf-8 -*- -->

README
======
The CouchDB HTTP View API defines many query options.

The group query option provides a set of distinct keys in the  specified view.
This is similar to the "SELECT DISTINCT(column)" query of SQL.

The "group\_numrows" option works with the "group" option, and provides the total number of distinct keys.

The behavior of the "group\_numrows" is similar to the "SELECT COUNT(DISTINCT column)" query.

However, if you get the total number without it, you have to need much more memory and cpu resources.
It might be a problem if the set is quite large.

The "group_numrows" query option for CouchDB 1.0.2
--------------------------------------------------
Please refer the explanation for CouchDB 1.0.1.

The "group_numrows" query option for CouchDB 1.0.1
--------------------------------------------------

### Overview

The following pseudo code shows an example of the "group\_numrows";

    ## pseudo code in ruby
    json = couch.get("/example/_design/test/_view/test?group=true&group_numrows=true")
    h = JSON.parse(json)
    total_numrows = h["group_numrows"]

In above code, the "json" is single string expression, '{"group\_numrows":102}'.

#### without "group\_numrows"
If you want to get the total number of distinct keys without the "group\_numrows" option.

You have to get the set of distinct keys, convert to the Hash object or something, and finally call the length or size method like in the following way.

    ## pseudo code in ruby
    json = couch.get("/example/_design/test/_view/test?group=true")
    h = JSON.parse(json)
    total_numrows = h["rows"].length

The processing time is dependent on the memory consumption of the json string at the both server and client sides.

Installation
------------
To recompile beam files are recommended from the source.

    $ cd apache-couchdb-1.0.1/src/couchdb/
    $ mv couch_db.hrl couch_db.hrl.orig
    $ mv couch_httpd_view.erl couch_httpd_view.erl.orig
    $ curl -o couch_db.hrl https://github.com/YasuhiroABE/CouchDB-Group_NumRows/raw/master/couch_db.hrl
    $ curl -o couch_httpd_view.erl https://github.com/YasuhiroABE/CouchDB-Group_NumRows/raw/master/couch_httpd_view.erl
    $ make
    $ make install

Otherwise, replace the *.beam* with the *.erl* as an instant way.

    $ rm /usr/local/lib/couchdb/erlang/lib/couch-1.0.1/ebin/couch_db.hrl
    $ cp couch_db.hrl /usr/local/lib/couchdb/erlang/lib/couch-1.0.1/ebin/
    $ rm /usr/local/lib/couchdb/erlang/lib/couch-1.0.1/ebin/couch_httpd_view.erl
    $ cp couch_httpd_view.erl /usr/local/lib/couchdb/erlang/lib/couch-1.0.1/ebin/

Examples
--------
As an example, the "example" database has four document and defines the "all" view.

    $ curl 'http://localhost:5984/example/_design/all/_view/all?group=true'

The result is;

    {"rows":[
    {"key":["bar","35"],"value":3},
    {"key":["foo","25"],"value":3},
    {"key":["somebody","20"],"value":8},
    {"key":["yasu","32"],"value":4}
    ]}

To send the same request with the "group\_numrows";

    $curl 'http://localhost:5984/example/_design/all/_view/all?group=true&group_numrows=true'
    
    {"group_numrows":"4"}

Implementation Detail
---------------------
The operation of the "group\_numrows" counts up all the number of the 'rows' array in the CouchDB by the erlang thread.

If the "limit=3" is set in the above example, the results of the group\_numrows will be the '{"group\_numrows":"3"}'.

The results is always same as the number of the array["rows"].length.

The "total\_rows" of the \_all\_docs always returns the total rows, so it is the difference between "total\_rows" and "group\_numrows."

Appendix
--------

* [English - Blog; CouchDB: Implementation of SELECT COUNT(DISTINCT field)...](http://yasu-2.blogspot.com/2010/12/couchdb-implementation-of-select.html "CouchDB: Implementation of SELECT COUNT(DISTINCT field)")
* [Japanese - Blog; CouchDBでSELECT COUNT(DISTINCT ...)をするために必要なこと](http://yasu-2.blogspot.com/2010/12/couchdbselect-countdistinct.html "CouchDBでSELECT COUNT(DISTINCT ...)をするために必要なこと")

License
-------
The original code is licensed under the Apache License, Version 2.0.

The partial code of the couch\_db.hrl and couch\_httpd\_view.erl are also licensed under the Apache License, Version 2.0.

    Copyright (C) 2010,2011 Yasuhiro ABE <yasu@yasundial.org>

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at
    
         http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

__EOF__

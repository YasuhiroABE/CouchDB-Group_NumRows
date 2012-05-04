<!-- -*- mode: markdown ; coding: utf-8 -*- -->

README
======
The "group\_numrows" is new query option for the CouchDB HTTP View API.
It provides a functionality like the "SELECT COUNT(DISTINCT column)" query of SQL.

The "group=true" query option provides a set of distinct keys in the specified view.
This is similar to the "SELECT DISTINCT(column)" query.

The "group\_numrows" works with the "group=true" query option,
and provides the total number of distinct keys.

### With a Example
The number of unique group keys will be calculated like the following;

          query with "group=true"                [Ruby] temporary object
    +-----------------------------------+       +------------------------+		   
    | {"key":"A","value":xxxx}          |  ---> | results["rows"].length |  ---> 12345
    | {"key":"B","value":xxxx}          |       +------------------------+
    | ... skip ...                      |
    | {"key":"ZZZZZZZZZZ","value":xxxx} |
    +-----------------------------------+

    ## pseudo code in ruby
    json = couch.get("/example/_design/test/_view/test?group=true")
    h = JSON.parse(json)
    total_numrows = h["rows"].length


In this case, the "results" hash object has to store all results temporarily.
It might be a problem if the number of results is quite large.

The "group\_numrows=true" query returns only the number of unique keys.

    query with "group_numrows=true&group=true"
    +--------------------------+
    | {"group_numrows":12345}  |
    +--------------------------+

    ## pseudo code in ruby
    json = couch.get("/example/_design/test/_view/test?group=true&group_numrows=true")
    h = JSON.parse(json)
    total_numrows = h["group_numrows"]

For instance, there are almost 100K result lines (almost 3MB json string) in my linux box.
The group\_numrows=true operation is 8.5 times faster than calculating the length of a hash object.

Tested Platforms
----------------
This release tested on following systems.

* CouchDB 1.0.1 with Ubuntu 10.04 LTS x86_64, Debian 5 and 6 (i386, x86_64)
* CouchDB 1.0.2 with Ubuntu 10.04 LTS x86_64, Debian 5 and 6 (i386, x86_64)
* CouchDB 1.2.0 with Ubuntu 12.04 LTS x86_64, Debian 5 and 6 (i386, x86_64)

Installation
------------
To recompile beam files are recommended from the source.

    $ cd apache-couchdb-1.2.0/src/couchdb/
    $ mv couch_db.hrl couch_db.hrl.orig
    $ mv couch_httpd_view.erl couch_httpd_view.erl.orig
    $ curl -o couch_db.hrl https://github.com/YasuhiroABE/CouchDB-Group_NumRows/raw/master/couch_db.hrl
    $ curl -o couch_httpd_view.erl https://github.com/YasuhiroABE/CouchDB-Group_NumRows/raw/master/couch_httpd_view.erl
    $ make
    $ make install

## Another way
Otherwise, replacing *.beam* files with *.erl* files is a simple way on existing system.

    $ rm /usr/local/lib/couchdb/erlang/lib/couch-1.2.0/ebin/couch_db.beam
    $ cp couch_db.hrl /usr/local/lib/couchdb/erlang/lib/couch-1.2.0/ebin/
	$ cp couch_db.hrl /usr/local/lib/couchdb/erlang/lib/couch-1.2.0/include/
    $ rm /usr/local/lib/couchdb/erlang/lib/couch-1.2.0/ebin/couch_httpd_view.beam
    $ cp couch_httpd_view.erl /usr/local/lib/couchdb/erlang/lib/couch-1.2.0/ebin/

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

* English [CouchDB: Implementation of SELECT COUNT\(DISTINCT field\)...](http://yasu-2.blogspot.com/2010/12/couchdb-implementation-of-select.html "CouchDB: Implementation of SELECT COUNT\(DISTINCT field\)")
* Japanese [CouchDBでSELECT COUNT\(DISTINCT ...\)をするために必要なこと](http://yasu-2.blogspot.com/2010/12/couchdbselect-countdistinct.html "CouchDBでSELECT COUNT\(DISTINCT ...\)をするために必要なこと")

License
-------
The original code is licensed under the Apache License, Version 2.0.

The partial code of the couch\_db.hrl and couch\_httpd\_view.erl are also licensed under the Apache License, Version 2.0.

    Copyright (C) 2010-2012 Yasuhiro ABE <yasu@yasundial.org>

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

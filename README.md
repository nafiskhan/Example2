PGSqlitePlugin: SQLitePlugin for Phonegap
=========================================

This plugin exists because I developed some large enterprise (boring) mobile
applications and found some problems using WebkitSQLite

- Quota limit (~5Mb on ios)
- [Transaction problems related to user interaction](http://stackoverflow.com/questions/8741000/html5-web-sql-transactions-skipped-without-error-when-touch-triggered-in-ios)
- [Persistence problems](https://issues.apache.org/jira/browse/CB-330)

The plugin is meant to work with Phonegap 1.7.0 (Cordova).

The API is not the same as HTML5 Web Sql one. I'm trying to move things in that
direction though.

Installing
==========

SQLite library
-------------------

In order to use the plugin you need to link the sqlite library in your phonegap application.

In your projects "Build Phases" tab, select the _first_ "Link Binary with Libraries" dropdown menu and add the library `libsqlite3.dylib` or `libsqlite3.0.dylib`.

**NOTE:** In the "Build Phases" there can be multiple "Link Binary with Libraries" dropdown menus. Please select the first one otherwise it will not work.

PGSQLite Plugin
---------------

Drag .h and .m files into your project's Plugins folder (in xcode) -- I always
just have "Create references" as the option selected.

Take the precompiled javascript file from build/, or compile the coffeescript
file in src/ to javascript WITH the top-level function wrapper option (default).

Use the resulting javascript file in your HTML.

Look for the following to your project's PhoneGap.plist:

    <key>Plugins</key>
    <dict>
      ...
    </dict>

Insert this in there:

    <key>PGSQLitePlugin</key>
    <string>PGSQLitePlugin</string>

General Usage
=============

`www/index.html` contains a test application that runs very simple queries
using plain javascript. It is recommended that you first run this one and check
the XCode console to see that everything is working fine.

The following examples show you how you can use transactions.

## Coffee Script

    db = new PGSQLitePlugin("test_native.sqlite3")
    db.executeSql('DROP TABLE IF EXISTS test_table')
    db.executeSql('CREATE TABLE IF NOT EXISTS test_table (id integer primary key, data text, data_num integer)')

    db.transaction (tx) ->

      tx.executeSql [ "INSERT INTO test_table (data, data_num) VALUES (?,?)", "test", 100], (res) ->

        # success callback

        console.log "insertId: #{res.insertId} -- probably 1"
        console.log "rowsAffected: #{res.rowsAffected} -- should be 1"

        # check the count (not a part of the transaction)
        db.executeSql "select count(id) as cnt from test_table;", (res) ->
          console.log "rows.length: #{res.rows.length} -- should be 1"
          console.log "rows[0].cnt: #{res.rows[0].cnt} -- should be 1"

      , (e) ->

        # error callback

        console.log "ERROR: #{e.message}"

## Plain Javascript

    var db;
    db = new PGSQLitePlugin("test_native.sqlite3");
    db.executeSql('DROP TABLE IF EXISTS test_table');
    db.executeSql('CREATE TABLE IF NOT EXISTS test_table (id integer primary key, data text, data_num integer)');
    db.transaction(function(tx) {
      return tx.executeSql(["INSERT INTO test_table (data, data_num) VALUES (?,?)", "test", 100], function(res) {
        console.log("insertId: " + res.insertId + " -- probably 1");
        console.log("rowsAffected: " + res.rowsAffected + " -- should be 1");
        return db.executeSql("select count(id) as cnt from test_table;", function(res) {
          console.log("rows.length: " + res.rows.length + " -- should be 1");
          return console.log("rows[0].cnt: " + res.rows[0].cnt + " -- should be 1");
        });
      }, function(e) {
        return console.log("ERROR: " + e.message);
      });
    });


Lawnchair Adapter Usage
=======================

Include the following js files in your html:

-  lawnchair.js (you provide)
-  pgsqlite_plugin.js
-  lawnchair_pgsqlite_plugin_adapter.js (must come after pgsqlite_plugin.js)


The `name` option will determine the sqlite filename. Optionally, you can change it using the `db` option.

In this example, you would be using/creating the database at: *Documents/kvstore.sqlite3* (all db's in PGSQLitePlugin are in the Documents folder)

    kvstore = new Lawnchair { name: "kvstore", adapter: PGSQLitePlugin.lawnchair_adapter }, () ->
      # do stuff

Using the `db` option you can create multiple stores in one sqlite file. (There will be one table per store.)

    recipes = new Lawnchair {db: "cookbook", name: "recipes", ...}
    ingredients = new Lawnchair {db: "cookbook", name: "ingredients", ...}

### Other notes from @Joenoon:

I played with the idea of batching responses into larger sets of
writeJavascript on a timer, however there was only a barely noticeable
performance gain.  So I took it out, not worth it.  However there is a
massive performance gain by batching on the client-side to minimize
PhoneGap.exec calls using the transaction support.

### Other notes from @davibe:

I used the plugin to store very large documents (1 or 2 Mb each) and found
that the main bottleneck was passing data from javascript to native code.
Running PhoneGap.exec took some seconds while completely blocking my
application.


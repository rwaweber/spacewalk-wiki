# PostgreSQL Porting Guide



On this page we are going to collect all valuable information that can help with porting our codebase to PostgreSQL. You will find description of all problems with links into Git with example solutions on the page.
## General rules



  * Setup two testing scenarios prior to fixing any errors
    * spacewalk nightly on Oracle
    * spacewalk nightly on PostgreSQL
  * Always fix only one problem and then test on both
  * Turning on detailed logging is a really good idea:
    * `sed -i 's/^#*hibernate.show_sql=true.*$/hibernate.show_sql=true/' /usr/share/rhn/config-defaults/rhn_hibernate.conf`
    * `echo "debug = 5" >> /etc/rhn/rhn.conf`
    * `echo "log_statement = 'all'" >> /var/lib/pgsql/data/postgresql.conf`
  * The most valuable log file is the postgresql.conf (you have to turn on the logging - see above). You will likely see all database related errors there.
  * Checkout these logs:
    * /var/lib/pgsql/data/pg_log/postgresql-$(date +%a).log
    * /var/log/tomcat5/catalina.out
    * /var/log/httpd/error_log
    * /var/log/rhn/*log
  * If you edit a file in schema/spacewalk/oracle you need to "synchronize" the counterpart in schema/spacewalk/postgres and make sure the oracle equivalent source header has the correct sha1sum of the oracle part. There is an automatic check that verifies all sha1 values. Example: `--- oracle equivalent source sha1 de23851fb6217e6b57baf09fabe5c0e2e41f15b6`
  * For synchronizing SQL files we recommend Meld tool (or any other nice diff) because the PLSQL and PgSQL syntax is very similar. Its also a good learning tool if you are not sure about PLSQL or PgSQL syntax.

We use some shortcuts on the page:
 * ORA = Oracle
 * PG = PostgreSQL
 * ERROR = the most important part of the typical error in general form
 * EXAMPLE = an example error
 * EX = example git commits links
## Problems and how to solve them

### The VARCHAR-NULL problem

 #empty_string_null


ERROR: new row for relation "tablename" violates check constraint "vn_tablename_columnname"

EXAMPLE: new row for relation "rhnservernetinterface" violates check constraint "vn_rhnservernetinterface_broadcast"

This is probably the most common issue we are facing to. On ORA the empty varchars are treated as NULLS. On PG the empty VARCHAR is not equivalent to NULL.

The solution is to avoid inserting empty varchars from the database at all. For each nullable varchar we have created a constraint that will check it (a1efdb099baafd8b71680bf55d99545244ffb1e0). All these constraints starts with "vn_" as in VARCHAR-NULL. If you see the above example error you are home.

In our Java code we have created an Hibernate Interceptor (22df0ce35498a3a98f3685305ba892ba18d07c0c) that checks for empty varchars and issues WARNINGs in the Catalina.log when such a field is found. It can be also configured to convert empty varchars to NULLs automatically. It has been configured to do so. The interceptor will be switched off in the future.

If you see a warning in form of 'Object XY is setting empty string on' then you should fix the java code (probably some HTML form) trying to save empty string in the varchar. This should be changed to NULL.

EX: f73476582d8243ec6f5539ad93b9e923f76e06d7 a966e661d27742687e28b4c2fa7c21c4716743d8 d62c91f85775663f3e007d69ec23e173ac8ad818 eedebc4a67de4369bd4bcdd3af558f5bb541facc
### The DECODE/NVL2 functions problem



ERROR: function decode(...) does not exist
ERROR: function nvl2(...) does not exist

EXAMPLE: function decode(character, unknown, integer, unknown, integer) does not exist

Either DECODE or NVL2 functions do not exist on PG. The fix is to rewrite the statement in CASE ... WHEN ANSI syntax that works on both platforms.

Please note empty VARCHARS are not permitted to be inserted in the database. We have constraints for that, its not possible to do.

Example:


    ... DECODE(column, 1, 'YES', 0, 'NO') ...

Must be rewritten as


    ... CASE column WHEN 1 THEN 'YES' WHEN 0 THEN 'NO' END

Do not forget the END keyword.

EX: 7d337c4d88e4057d542601e2fb48099557e13e05 f5ef2d83a72b8f4150f17477664ef75d21e45392 3a97bffade6be5edbc399e775ab861a1bd29e02e
### The NVL function problem



ERROR: function nvl(...) does not exist

EXAMPLE: function nvl(integer, integer) does not exist

The NVL function does not exist on PG and we have defined one: `NVL(VARCHAR, VARCHAR)`. This is the only one that will work on both platforms so if you have two VARCHARs you must use NVL (no change here) because of empty varchars problem (see above). For all other cases you need to replace it with COALESCE function that do the same job on both platforms.

EX: b9d3b3f2d8e09fd0a1d130e31f5c98280269cadf
### JOIN in ANSI syntax



ERROR: syntax error at or near ")" at character

We need to rewrite all DML SQL to syntax which is accepted by both Oracle and PostgreSQL. In our codebase we use Oracle outer join syntax:


    SELECT ...
     FROM ROM rhnChannel {c}, rhnChannelCloned c_1_
    WHERE c.id = c_1_.id (+)

This needs to be rewritten to ANSI:


    SELECT ...
     FROM rhnChannel {c}
      LEFT OUTER JOIN rhnChannelCloned c_1_ ON c.id = c_1_.id

This is the same for right outer joins. Inner joins works okay in both Oracle and PostgreSQL so we do not want to rewrite these. This is the same in Hibernate, direct queries, PLSQL/PgSQL, backend and web.

EX: 0b1f33ff900514a03485aa8058b8b277b0d6c66e 126169e289ea91f06960cfdb3dc408961ee10d02 
### SELECT column AS alias



ERROR: "alias" does not exist

EXAMPLE: "name" does not exist

In ORA you can skip AS keyword in column alias definition but PG needs it. We need to add AS keywords in `SELECT col AS alias...` constructs.

EX: 9296d280982ef8f17815a05fbbb2e98474949da0
### Default cast to integer



ERROR: com.redhat.rhn.common.util.MethodNotFoundException: Could not find method called: xxx in class: yyy with params: [ttt, value: n](type_)

EXAMPLE: com.redhat.rhn.common.util.MethodNotFoundException: Could not find method called: setSelectable in class: com.redhat.rhn.frontend.dto.SystemOverview with params: [java.lang.Integer, value: 1](type_)

The PG server casts numbers to INTEGER but ORA casts to NUMERIC. In Java INTEGER is mapped to Integer class and NUMERIC to Long class. Unfortunately wrapping classes are not subject of implicit type conversion and our code is trying to call method with incorrect parameter.

In this case the error message tells us everything we need. We need to create new method SystemOverview.setSelectable(Integer param) and just delegate call to setSelectable(Long) method using explicit type conversion. Always do explicit type conversion and do not rely on boxing/unboxing to prevent endless loop.

Please note you might want to explicitely cast in the SQL using CAST (x AS BIGINT) to solve this but the preferred solution is to fix this in Java. It seems Python and Perl have no problems with it since they recast it properly. You may see such commits (f65e7c6932b4a7024f30d0db1ec81975c2ce57dc) but they was fixed in the proper way later on.

EX: c2fda0ad08ff8c7c637eae6a36cf28940f8bda1a 203a68650582ba695a8b730bb8c4be39c199a155
### The sysdate problem



ERROR: column "sysdate" does not exist at character XX

On ORA we use DATE for columns with dates but on PG we use TIMESTAMP WITH TIMEZONE. On ORA the sysdate function returns current date. On PG there is no such function and since we use the timestamp we need to use current_timestamp. Fortunately this is an ANSI function and it works also on ORA but it has some drawbacks.

When we insert a value into table with DATE/TIMESTAMP columns its ok to use it since on ORA the proper date is inserted (and on PG as well). If you run into this problem you can just replace the function call and test if everything is ok.

If this function is used in WHERE clauses in a simple comparsion (` col1 < current_timestamp OR col2 > current_timestamp`) this is also ok and it works on both platforms but please make sure the column type is DATE/TIMESTAMP on ORA or PG respectively.

If there is any date/time arithmetics (operators plus/minus, function calls) you cannot simply replace sysdate with current_timestamp. Please see other recepies on this page (Date arithmetics) for more info.

EX: 25fcd0c602946d676ab1b070a64ceba072e214d3 d72a4c04b6d0c6659a89093e5a8c52488db1ad61 aaee1dd0f369ac11476a7af3bd4acd82f63aa089
### Date arithmetics



ERROR: ???

On ORA we use DATE for columns with dates but on PG we use TIMESTAMP WITH TIMEZONE. In the current codebase there are many places where numbers or intervals are being added or substracted from dates (timestamps in PG). This wont work on PG as there are no such operators defined and its not easy to define them in a portable form.

The easiest situation is in constant addition or subctraction (days are most relevant here). This select

`select sysdate as one, sysdate - 1 as two from dual`

can be rewritten as

`select current_timestamp as one, current_timestamp - interval '1' day as two from dual`

Please note you cant bind parameters to this form and the only situation this will work is a constant. The same applies for second, minute, month, year. Do not use plurals and enquote only the number.

The situation complicates with parameters and we have defined several functions for arithmetics. Which function to use depends on what you need as a result:
  * number of days -> date_diff_in_days (typical scenario is `somedate - otherdate`)
  * date -> numtodsinterval (typical scenario is `something < somedate - something_else`)

Please note when using numtodsinterval it currently only accepts the second parameter set to 'second' (as in time not as in counting) for now. So you need to recompute the input value to seconds. For this you need to know first in which scale is the input parameter. Is it in days, months, years or seconds?

Two examples:


     select date_column - otherdate as number_of_days_until_expiration from ...

We obviously expect number of days se we rewrite to


     select date_diff_in_days(somedate, otherdate) as number_of_days_until_expiration from ...

In this example date vs date are compared:


     select ... where column > sysdate - :age

After finding out that :age parameter is given in days we can multiply by 86400 and do the timestamptz vs timestamptz comparsion:


     select ... where column > current_timestamp - numtodsinterval(:age * 86400, 'second')

The numtodsinterval now accepts: second, minute, hour, day. You can use any of these.

EX: e43029294891b49940b5c75ddc01227280b89b08 b9dc390cb5101a391f424bd0a5a873076cd7e4b5 93fd4371dfb524b0e9a9e4e9a01b81a9843d4aa3 1959728caebe5c88b0563dc5276f5f8b27128cc5
### Triggers must return something



ERROR: control reached end of trigger procedure without RETURN

In PG all triggers must return record (the "new" in most cases) or null.

EX: 19e1daa3bb5b236e777adb4afce8fe263bda2390
### Triggers mustn't touch old if they are on insert



ERROR:

        record "old" is not assigned yet
        DETAIL:  The tuple structure of a not-yet-assigned record is indeterminate.
        CONTEXT:  PL/pgSQL function "rhn_package_mod_trig_fun" line 24 at IF

We cannot touch old if tg_op is INSERT. Apparently, if does not use lazy evaluation in PL/pgSQL, so we need to split the ifs into two.

EX: 590e9da99719ce814190f859c940cd94baf4e7ab 6746cbdb36e75b36ac05ab02eba9eb60330c30ee
### Procedure call from Hibernate



We cannot call stored procedure in the form


    <callable-mode name="oai_customer_sync">
       <query params="org_id">
        begin
        XXRH_OAI_WRAPPER.sync_customer(:org_id);
        end;
       </query>
     </callable-mode>

in Hibernate. We need to use the portable form of


    <callable-mode name="oai_customer_sync">
       <query params="org_id">
        { call XXRH_OAI_WRAPPER.sync_customer(:org_id) }
       </query>
     </callable-mode>

which is translated to both PostgreSQL and Oracle in a proper way. If you need to call a function (returning a value) then use `{ :arch = call lookup_package_arch( :label ) `}.

Also please note that in the example above the XXRH_OAI_WRAPPER is not a package in PG but schema. We use schemas to "simulate" packages from ORA.

EX: 73a318ccd254d4abd16ee40f2913dc183f652f8e
### ORDER BY expression in DISTINCT select



ERROR: for SELECT DISTINCT, ORDER BY expressions must appear in select list

ORA allows to sort by columns which are not in the select list but PG need to have it there.

* The temporary solution is to wrap the whole query as a subselect named "X". Example:

  
     select distinct a, b, c, label from rhnchannel
     order by upper(label)
      }}}
    
      Rewritten as:
    
      {{{
     select * from
     (
     select distinct a, b, c, label from rhnchannel
     ) X
     order by upper(X.label)
      }}}	
    
      EX: a283b7f6b7192250e9ab99c3942a3c0e5be3b1f2
    
      You may find some commits that was trying to add a new column in the original query (263859eb85c9aa56d3707d7cfc808c162be1493b f3aa49da371994bbd631272b0f66eda8642511df) but it turned out this is problematic approach and we have removed this.
    
    * Another option is to add the column name from the order by part to to the select part. Example:
    
      {{{
     SELECT  distinct C.name,
             UC.channel_id ID,
     ...
       FROM rhnUserChannel UC
     ...
       WHERE  UC.user_id = :user_id
       ORDER BY UPPER(C.name)
      }}}
    
      Rewritten as:
    
      {{{
      SELECT  distinct C.name,
              UPPER(C.name),
              UC.channel_id ID,
     ...
       FROM rhnUserChannel UC
     ...
       WHERE  UC.user_id = :user_id
       ORDER BY UPPER(C.name)
      }}}
    
      EX: 05be4a6f4d509e55dcc9ef21d5a763ebf37f41d1
### Rownum problem

    

    ERROR:  column "rownum" does not exist
    
    There is no ROWNUM alternative in PG except. There is no such feature in PG with the same behaviour but for limiting results there is a LIMIT - OFFSET syntax.
    
    We use ROWNUM only for limiting results at the moment. There are only 2 occurances in our java code base and three in the backend. As there is no ANSI SQL compatible syntax that could work on both platforms the fix is to create separate queries for both.
    
    There are situations where rownum column is used for separating odd and even records during a presentation in the java UI. The fix is to use `${loop.count % 2 == 0`} syntax instead of the rownum.
    
    EX: TBD
### Subquery with no alias

    

    ERROR:  subquery in FROM must have an alias at
    
    It seems all subqueries in PG must have an alias. Its easy to add one e.g. ` ... (select ....) XXX ... `. We usually name the subselect as SUBSELECT (or SUBSELECT1, SUBSELECT2...).
    
    EX: 62a15afa13dfe890c6bc86ce010699419fbf7383 (please note the AS keyword should not be present and has been fixed in 546adc9416d999ae8f198bf7a5952a9956d204be)
### Composite type accessing

    

    In PG you have to use braces when accessing a composite type.
    
    E.g. when you have 
    
    {{{
    select PE.evr.epoch AS EPOCH ...

PostgreSQL will give error:


    ERROR:  schema "pe" does not exist

EX: 6a511109bde9ef84c3dd9d6dc98c9635acf5765b
### Concatenating of evr



ERROR: DECODE not found...

We often concatenate package evr in many places using this SQL construct


    ... (latest.evr).version || '-' || (latest.evr).release || DECODE((latest.evr).epoch, NULL, '', ':' || (latest.evr).epoch) || (CASE WHEN latest.arch_id IS NULL THEN '' ELSE '.' || latest.arch_label END) ...

There is a function that do the same `... evr_t_as_vre_simple(PE.evr) ...` but please double check the concatenation being replaced is very same. By replacing we got rid of the DECODE function that wont work on PostgreSQL.

EX: 4f9ee9ff1b5d536178d106d4f1541396bf899be5
### Global function evr_t_as_vre_simple



ERROR: cross-database references are not implemented: pe.evr.as_vre_simple

Composite types has no member functions and the fix is really straightforward here. Replace 


    ... PE.evr.as_vre_simple() ...

with global function


    ... evr_t_as_vre_simple(PE.evr) ...

EX: add2a44a2493b877b1a2f869a0d958c706def207 6b4dccb40b9c5b99579b1246623f1afc2db1b1b4
### No autonomous transactions



ERROR: syntax error at or near "autonomous_transaction" at character 

In PG you cannot write


    declare
     pragma autonomous_transaction;
    begin
     -- sql commands
     commit;
    end;

you need to rewrite it into single command(s):


     -- sql commands

EX: eca6fe0fb34bbea9595527f51e743221ece5e275

The autonomous transactions are generally used in lookup situations where we want to either find the entity if it already exists in the table, or insert new one and return the id. When we replace it with normal SQL, obviously we confine it to the transaction, so any concurrent transaction will block before our transaction commits (or rollbacks), and most probably it will then fail with some unique constraint violation.

We can address this issue by making the transactions where we've inserted new lookup records as short as possible (we do not worry if the lookup entries end up in the database even if the main transaction then rollbacks), or we can use second connection to the database to do those parallel inserts. That however seems to be hard because the backend (Python stack) code seems to assume there is only one connection to the database -- we don't even have any dbh object/variable in that code.
### NUMBER to NUMERIC



Since ORA NUMBER is NUMERIC in PG we heed to replace these. All DDL were already fixed but you will find these usually in PLSQL/PgSQL.

EX: dacafca6ba2b1d2a2f90ec07864187c69f013791
### Calling procedures



To call a procedure on ORA we use something like


    BEGIN
     package.proc(...);
    END;

and this wont work on PG. For functions that does not return anything (VOID) use simple


     perform package.proc(...);

and for functionts that returns something you have to use


     select package.proc(...);
### SELECT UNIQUE



ERROR:  syntax error at or near "column"

We use SELECT UNIQUE sometimes but AskTom says that from version 10g its only synonym for SELECT DISTINCT. And DISTINCT works on both platforms.

EX: 62a15afa13dfe890c6bc86ce010699419fbf7383
### TO_NUMBER function



ERROR: function to_number(unknown) does not exist

The function TO_NUMBER(string) is not recognized in PG. We have to use TO_NUMBER(string, string) version - with the second parameter which is the number format. This is the same as on Oracle.

We use to_number(null) very often, the fix is to convert it to to_number(null, null). For other occurances the fix is to include the second parameter.

EX: 62a15afa13dfe890c6bc86ce010699419fbf7383
### TO_DATE function



The function TO_DATE(string, string) returns date (i.e. without hours/minutes/seconds) in PG. We have to use TO_TIMESTAMP(string, string) which returns both date and time of day both in Oracle and PG.

EX: ???
### DELETE without FROM



ERROR: syntax error at or near "tablename"

PG does not allow to skip FROM keyword in DELETE FROM tablename. Its easy fix to add it.
### DUAL table



On PG there is a possibility to do ` SELECT ... ` without any table. We have created a fake DUAL table so you can do ` SELECT ... FROM DUAL ` on both platforms.
### MINUS keyword



ERROR:  syntax error at or near "MINUS" at character xxx

Postgres doesn't support Oracle's MINUS, it supports EXCEPT. But EXCEPT isn't supported by Oracle.

If it is possible to distinguish, whether we're on PG or on ORA, it can be solved in the following way:

EX: 125a1e4cd60785a667f11cd1f921dd7ccaaf92e7

otherwise it's possible to rewrite every MINUS into a OUTER JOIN (recommended way)

EX: 3f95a42cb3640370579ef81d52157c1e53cf2fb5 b8f305136a2f525670ef901a5d986eb7050c98dc
### Bind parameter with space



ERROR:  syntax error at or near ":" at character xxx

There must not be space between : and parameter in ":parameter". Between equal sign and colon must be at least one white space. We have to remove these.

EX: 910e069f689bc91abcf962f5d10f8ef3e4b3073c
### Portable nextval



ERROR: missing FROM-clause entry for table "tablename"

The nextval syntax in different on both platforms. We have defined global function `sequence_nextval('sequence_name')` that works on both platforms. In some cases the explicit sequence insert can be removed at all which is preferred solution.

EX: 2cec4e439e2f64a75b5564656c3761d0a85357ce
### Recursion with opened cursors



ERROR: cursor "cursorname" already in use

ORA allows to set variable open_cursors (we have currently 300) that allows recursion in PLSQL with opened cursor. Unfortunately PG does not allow this but there is possibility to rewrite the loop without any cursor using FOR record IN SELECT ... LOOP syntax.

EX: 0bb8c201eb593018ffcb66aded1321106acda81f
### Anonymous procedural SQL blocks, in Python (backend)
 #anonymous_plpgsql


PG does not support anonymous PL/pgSQL blocks. We workaround this by creating functions on-the-fly, and then calling them.

In the `backend.dbmodule.prepare`, use `params` parameter with list of strings that list the parameter names and types.

EX: 683d908f67cc7d79ad392d377f13ef8c674818a7

If the anonymous block does not take any arguments, the list will be empty.

EX: 30903ca16fa55cab6994dbd97595eb549bd3f619

Since the ORA and PG syntax for cursors differs a bit, you can use comments `/*pg_cs*/` and `/*pg cursor*/` to tell the PG driver to massage the source to make PG happy, while masking those strings from ORA -- the `pg_cs` eats the following `cursor` word, and `pg cursor` puts that `cursor` back for PG.
### Relation (table) does not exists
 #tablenotexist


ERROR:  relation "table_alias" does not exist at character XX

PG does not support table aliases. This could be partially workarounded by creating view from original table but even simple views in PG does not support updates (i.e. updating underlying tables via `update view set ....` query).

So the right fix is replacing table alias with name of referenced table.

EX: d292753948be6718cee2639c68b7a98d58c2f11a, 566bbf371d4cf831705c4a3afe19bb1e7f9efe81, d347f40a137622b4b12a9400a5d4ca8dfdb6ede9
### Inserting / writing blob in Python (backend)
 #blob_in_python


ERROR: AttributeError: 'buffer' object has no attribute 'write'

Oracle does not support direct insert into blob columns. This is traditionally solved by inserting `empty_blob()` instead, then selecting blob id and writing content to it.
On PG we use `bytea`s instead of blobs in most cases which can be inserted directly but content can't be written into it (as it is for blobs in Oracle).
We overcome this difference by extending oracle python driver with support for direct blob inserts.

In the `backend.dbmodule.prepare`, use `blob_map` parameter with dict of {blob1_column_name: bind_variable_name1, ...} pairs.

EX: c795aaae8e3d6d68418c066fc0c190fb60e7df08, 1966210148756644c244fed259e93d2b4f2d5cef
For up-to-date information about the PostgreSQL port, see [[PostgreSQL]]. This page is now obsolete.

----

While a bunch of preliminary schema and app stack prep for PostgreSQL
went out in Spacewalk 0.6 there were a few small problems, now fixed
and soon to land in the devel repo. For those interested in trying this
out and hopefully contributing here's the basic steps to get going now,

 1. Install 0.6 normally, but don't run spacewalk-setup.
   * Yes Oracle packages need to be installed but it doesn't need to be
     configured. We still need to sort out a lot of deps issues.
 1. Switch to Spacewalk nightly repo: http://spacewalk.redhat.com/yum/nightly/
 1. Update packages.
 1. Make sure you have PostgreSQL 8.4 installed: http://yum.pgsqlrpms.org/howtoyum.php I use a script like this to blow it away and reinstall fresh:

    yum remove -y postgresql postgresql-server
    rm -rf /var/lib/pgsql/
    yum install -y postgresql-libs postgresql postgresql-server
    service postgresql initdb
    service postgresql start
    # Enable password authentication for network connection from localhost:
    echo "local all all ident" > /var/lib/pgsql/data/pg_hba.conf
    echo "host all all 127.0.0.1/32 md5" >> /var/lib/pgsql/data/pg_hba.conf
    echo "host all all ::1/128 md5" >> /var/lib/pgsql/data/pg_hba.conf
    service postgresql restart
    chkconfig postgresql on
    su - postgres -c 'createuser --superuser -P spacewalk' <<EOF
    spacewalk
    spacewalk
    EOF
    su - postgres -c 'createdb spacewalk'
    su - postgres -c 'createlang plpgsql spacewalk'
 1. yum install python-pgsql (adding the dep to spacewalk-backend now)
 1. Run spacewalk-setup and specify postgresql for the database type. My answers file looks like:
 
    admin-email = you@example.com
    db-backend = postgresql
    db-user = spacewalk
    db-host = localhost
    db-password = spacewalk
    db-sid = spacewalk
    db-port = 5432
    db-protocol = TCP
    ssl-set-org = Your Org
    ssl-set-common-name = Your Org Test
    ssl-set-org-unit = WHAT
    ssl-set-city = Raleigh
    ssl-set-state = NC
    ssl-set-country = US
    ssl-password = whatever
    ssl-set-email = whatever@example.com

If you have the latest code this should get past schema population,
then fail during satellite activation, rhn-installation.log will report
a problem with modify_org_service procedure. The procedure exists it's
just probably untested and not working. :)

So next steps would be to correct the (IIRC) 2 stored procedure
problems that block spacewalk-setup from completing and then we're
getting quite clear to start diving into the app, fixing page by page
or so. 

DISCLAIMER: We're just talking about SURVIVING setup here, the app and
it's thousands of queries will still not work against PostgreSQL, many
stored procedures have not been ported, others have been ported but
never tested in a live environment so we don't even know if they're
working.

As additional help, when working on postgresql schema it's quite
easy to build the schema on your devel machine and copy it to your
installed spacewalk instance for each change you want to test:


    cd schema/spacewalk/postgresql
    make && scp main.sql root@wherever:/etc/sysconfig/rhn/postgres/main.sql

Then on the spacewalk system re-run spacewalk-setup:


    su - postgres -c 'dropdb spacewalk && createdb spacewalk && createlang
    plpgsql spacewalk'
    
    spacewalk-setup --answer-file=/root/postgresql.txt --disconnected

Be warned that the "do you want to clear schema" option in
spacewalk-setup won't work for postgresql, it should probably be disabled until we figure out how to do this.

Not the smoothest process in the world, but it's a start.

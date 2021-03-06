
    #!div class="important" style="border: 2pt solid; text-align: center" 
    '''''DEPRECATED, NO LONGER USED''''' 
# Phase 1
 

The first phase will be to get the easy packages into Fedora/EPEL via a quick review.  The easiest method is probably to start with the Perl Modules.  I (stahnma) will try to contact the Perl SIG so they can hopefully get them reviewed rather quickly.  There are certainly some of the perl packages that will need to have the RPM spec file completely changed.  

We created a tracking bug that all review bugs should block.   When filing review bugs for Spacewalk stuff just block 'F-Spacewalk'
https://bugzilla.redhat.com/show_bug.cgi?id=452450

We can update the list of packages here and provide bugzilla numbers.
|| perl-Business-CreditCard-0.28-6.el5.src.rpm || || https://bugzilla.redhat.com/show_bug.cgi?id=452601 || stahnma || Now In Fedora || 
|| ~~perl-Crypt-CAST5_PP-1.02-6.el5.src.rpm~~ || || https://bugzilla.redhat.com/show_bug.cgi?id=452457 || Nigel Jones || This package might actually not be needed for Spacewalk at all. --adelton || 
|| perl-DBD-Oracle-1.19-8.el5.src.rpm || || Not compatible with Fedora licensing, as it requires oracle-instantclient-devel || null || || 
|| perl-Filesys-Statvfs-Df-0.72-9.el5.src.rpm || || https://bugzilla.redhat.com/show_bug.cgi?id=455443 || msuchy || Now In Fedora ||
|| perl-Frontier-RPC-0.07-7.el5.noarch.src.rpm || || https://bugzilla.redhat.com/show_bug.cgi?id=226258 || stahnma || Already In Fedora ||
|| perl-GD2-2.17-10.el5.src.rpm || || Already in Fedora, under different name (perl-GD) || ||||
|| perl-Math-FFT-0.26-12.el5.src.rpm || GPL/Artistic || https://bugzilla.redhat.com/show_bug.cgi?id=452832 || msuchy || Now In Fedora ||
|| perl-Schedule-Cron-Events-1.8-11.el5.src.rpm ||  GPL/Artistic || https://bugzilla.redhat.com/show_bug.cgi?id=461736 || msuchy || Now in Fedora ||
||perl-Set-Crontab-1.00-12.el5.src.rpm|| Artistic License || https://bugzilla.redhat.com/show_bug.cgi?id=452449 ||  msuchy || Now in Fedora || 
||perl-Crypt-GeneratePassword-0.03-11.el5.src.rpm||GPL/Artistic|| https://bugzilla.redhat.com/show_bug.cgi?id=452458 ||Nigel Jones || Now in Fedora ||
||~~perl-Crypt-RIPEMD160-0.04-14.el5.src.rpm~~||GPL/Artistic||https://bugzilla.redhat.com/show_bug.cgi?id=452453|| Nigel Jones|| Legal looking into License -- we get rid of this package

| ~~perl-Crypt-Rijndael-1.06-1.el5.src.rpm~~  | LGPLv2 | https://bugzilla.redhat.com/show_bug.cgi?id=452454 | Nigel Jones |  This package might actually not be needed for Spacewalk at all. --adelton (CVS Branched) |
| --- | --- | --- | --- | --- |
| perl-DateTime-Locale-0.4001-1.el5.src.rpm | GPL/Artistic | https://bugzilla.redhat.com/show_bug.cgi?id=452455 | Nigel Jones |  Already in Fedora, just under a slightly different name  |
| perl-DateTime-TimeZone-0.7701-1.el5.src.rpm | GPL/Artistic | https://bugzilla.redhat.com/show_bug.cgi?id=452456 | Nigel Jones |  Already in Fedora, just under a slightly different name  |
## Questions / Concerns

 * Does the Oracle dependency have to be removed before we can be included in Fedora?


   [[msuchy]] In perl we should break dependency and stop on perl-DBI. As connection string can be anything and should be set up in some config file. This part (which is basicaly Monitoring) should not be dependent on Oracle (but Oracle probe in Plugins).

 * Given the number of Perl packages required for Spacewalk, it may be worth considering completing removal of the Perl stack before attempting inclusion in Fedora. Could save some significant time for package reviewers and maintainers.

   [[msuchy]] Packages from above are Monitoring component, which is all in perl. Web part in perl is in packages rhn-sniglets.

 * Some packages, I am not sure if the perl ones are in this list, are not in installating into proper locations.  Just an FYI, nothing should be in /home/*, /usr/local/*, /opt/* or /srv/*.  There is some clean up to do here.  Would moving these items break items?

   [[msuchy]] I already moved /home (but to /opt/home (sic))

 * Also, some versions of packages may vary between the spacewalk repo and what is currently in Fedora.  Can we begin testing with the version in Fedora?

   [[msuchy]] definitely 

 * What is the furthest back for compatibility we need to worry about? Is Enterprise Linux 4 the earliest?

  [[msuchy]] yes, do not worry about RHEL3.
# Phase 2

I am moving phase 2 to be client side tools (stuff that normally would be in the Tools Channels).  


| Name |  License  |  Bugzilla  |  Submitter  |  Notes  |
| --- | --- | --- | --- | --- |
| jabberpy-0.5-0.13.el5.src.rpm |  LGPLv2+ (According to the SRPM)   |  [468299](https://bugzilla.redhat.com/show_bug.cgi?id=468299)  |  stahnma |  now in fedora  |
| osad-5.2.0-1.src.rpm |  GPLv2  |   |    |   |
| rhncfg-5.2.0-0.src.rpm |  GPLv2  |  [491088](https://bugzilla.redhat.com/show_bug.cgi?id=491088)  |  msuchy  |  Now in Fedora  |
| rhn-custom-info-5.2.0-1.src.rpm |  GPLv2 (According source in client/tools/rhncustominform) |  msuchy  |  [553649](https://bugzilla.redhat.com/show_bug.cgi?id=553649)  |  Now in Fedora  |
| rhn-kickstart-5.2.0-5.src.rpm |  GPLv2 (According source in client/tools/rhn-kickstart)   |   |    |    |  |
| spacewalk-koan-0.1.6-1.src.rpm   |  GPLv2 (According source in client/tools/spacewalk-koan)   |  [481668](https://bugzilla.redhat.com/show_bug.cgi?id=481668) |  mmccune/msuchy  |  Now in Fedora  |
| spacewalk-certs-tools-5.2.0-2.el5.src.rpm |  GPLv2  |  [538046](https://bugzilla.redhat.com/show_bug.cgi?id=538046)  |  msuchy  |  Now in Fedora  |
| rhn-virtualization-5.2.0-5.src.rpm | GPL2 (According to the RPM and source)  |   |    |    |  |
| rhnpush-0.3.1-2.fc10.src.rpm  |  GPL2  |  [ 452450](https://bugzilla.redhat.com/show_bug.cgi?id=452450) |  stahnma/msuchy  |  Now in Fedora  |
| rhnlib-2.5.10  |  GPLv2  |  [486937](https://bugzilla.redhat.com/show_bug.cgi?id=486937)  |  msuchy  |  Now in Fedora  |
| rhn-client-tools  |  GPLv2  |  [490438](https://bugzilla.redhat.com/show_bug.cgi?id=490438)  |  msuchy  |  Now in Fedora  |
| yum-rhn-plugin  |  GPLv2  | [525453](https://bugzilla.redhat.com/show_bug.cgi?id=525453)   |  msuchy  |  Now in Fedora  |
| rhnmd  |  GPLv2  |  [538057](https://bugzilla.redhat.com/show_bug.cgi?id=538057)  |  msuchy  |  Now in Fedora  |
| rhnsd  |  GPLv2  |  [524558](https://bugzilla.redhat.com/show_bug.cgi?id=524558)  |  msuchy  |  Now in Fedora  |

RPMS found in Satellite RHN Tools Channels where source/binary rpms are not available (AFAIK)
 * All of the auto-* packages 
 * rhn-applet-actions
 * rhns-config-libs
# Phase 3



We need to break dependancy on Oracle. - That is done now.


This packages need to be edited and then eventualy imported to Fedora:


|  name  |  licence  |  bugzilla  |  submiter  |
| --- | --- | --- | --- | --- | --- |
|  c3p0-0.9.0-2jpp.ep1.1.el5.src.rpm  |   |   |   |  Now in Fedora  |
|  cglib-2.1.3-2jpp.ep1.3.el5.1.src.rpm  |   |   |   |  Now in Fedora  |
|  eventReceivers-2.20.11-1  |  GPLv2  |  [487128](https://bugzilla.redhat.com/show_bug.cgi?id=487128)  |  msuchy  |  :( depends on DBD::Oracle  |
|  freemarker-2.3.6-2jpp_1rh.src.rpm  |   |   |   |  |
|  hadoop  |  Apache v2.0  |   |    |  |
|  hibernate3-3.2.4-1.SP1_CP01.0jpp.ep1.1.el5.1.src.rpm  |   |   |   |  |
|  jakarta-commons-configuration-1.2-1jpp_1rh.src.rpm  |   |   |   |  |
|  jfreechart-0.9.21-2jpp.ep1.1.el5.2.src.rpm  |   |   |   |  Now in Fedora  |
|  jpam-0.4-16.el5.src.rpm  |   |    |    |  |
|  MessageQueue-3.26.0-5.el5.src.rpm  |   |   |   |  |
|  NPalert-1.125.17-19.el5.src.rpm  |   |   |   |  |
|  ~~np-config-2.110.3-6.el5.src.rpm~~  |  GPLv2  |  [453111](https://bugzilla.redhat.com/show_bug.cgi?id=453111) - merging with NPuser into nocpulse-common  |  msuchy  |
|  ~~NPusers-1.17.11-6.el5.src.rpm~~ nocpulse-common  |  GPLv2  |  https://bugzilla.redhat.com/show_bug.cgi?id=453109  |  msuchy  |  Now in Fedora  |
|  nutch  |  Apache v2.0  |  GettingPackagesIntoFedora/Nutch  |   |  |
|  oscache-2.2-2jpp.ep1.1.el5.src.rpm  |   |   |   |  |
|  perl-Satcon-1.3-9.el5.src.rpm  |  GPLv2  |  https://bugzilla.redhat.com/show_bug.cgi?id=466777  |  msuchy  |  Now in Fedora  |
|  perl-NOCpulse-CLAC-1.9.4-10.el5.src.rpm  |  GPLv2  |  https://bugzilla.redhat.com/show_bug.cgi?id=485893  |  msuchy  |  Now in Fedora  |
|  perl-NOCpulse-Debug-1.23.4-6.el5.src.rpm  |  GPLv2  |  https://bugzilla.redhat.com/show_bug.cgi?id=483016  |  msuchy  |  Now in Fedora  |
|  perl-NOCpulse-Gritch-1.16.1-5.el5.src.rpm  |  GPLv2  |  https://bugzilla.redhat.com/show_bug.cgi?id=487115  |  msuchy  |  Now in Fedora  |
|  perl-NOCpulse-Object-1.26.4-7.el5.src.rpm  |  GPLv2  |  https://bugzilla.redhat.com/show_bug.cgi?id=485893  |  msuchy  |  Now in Fedora  |
|  nocpulse-db-perl  |  GPLv2  |  depends on spacewalk-base (/web directory)  |   |  |
|  perl-NOCpulse-Probe-1.183.1-21.el5.src.rpm  |   |   |   |  |
|  perl-NOCpulse-SetID-1.5.2-5.el5.src.rpm  |   |  https://bugzilla.redhat.com/show_bug.cgi?id=466906  |  msuchy  |  Now in Fedora  |
|  perl-NOCpulse-Utils-1.14.2-8.el5.src.rpm  |   |  https://bugzilla.redhat.com/show_bug.cgi?id=466953  |  msuchy  |  Now in Fedora  |
|  ProgAGoGo-1.11.0-5.el5.src.rpm  |   |   |   |  |
|  PyPAM-0.4.2-20.el5.src.rpm  |  LGPLv2  |  [612998](https://bugzilla.redhat.com/show_bug.cgi?id=612998)  |  msuchy  |  Now in Fedora  |
|  python-gzipstream-1.4.0-15.el5.src.rpm  |  Python and GPLv2  |  [657531](https://bugzilla.redhat.com/show_bug.cgi?id=657531)  |  msuchy  |  Now in Fedora  |
|  quartz-1.5.2-1jpp.ep1.5.el5.src.rpm  |   |  [738079](https://bugzilla.redhat.com/show_bug.cgi?id=738079)  |   |  Now in Fedora  |
|  redstone-xmlrpc-1.1_20071120-6.el5.src.rpm  |   |   |    |  |
|  spacewalk-java  |   |  Blocked by Nutch  |   |  |
|  rhn-satellite-admin-5.2.0-3.el5.src.rpm  |   |  [737972](https://bugzilla.redhat.com/show_bug.cgi?id=737972)  |   |  Now in Fedora  |
|  spacewalk-config  |  GPLv2  |  [491331](https://bugzilla.redhat.com/show_bug.cgi?id=491331)  |  msuchy  |  Now in Fedora  |
|  rhn-satellite-schema-5.1.0-27.src.rpm  |  requires spacewalk-setup  |   |   |  |
|  SatConfig-cluster-1.54.2-5.el5.src.rpm  |   |   |   |  |
|  servletapi4-4.0.4-3jpp_5rh.src.rpm  |   |   |   |  |
|  sitemesh-2.1-1jpp_1rh.src.rpm  |   |   |   |  |
|  spacewalk-0.1-7.src.rpm  |   |   |  mmccune  |
|  spacewalk-setup-0.01-3.el5.src.rpm  |  requires spacewalk-web  |   |   |  |
|  velocity-dvsl-0.45-5jpp_1rh.src.rpm  |   |   |   |  |
|  velocity-tools-1.2-1jpp_1rh.src.rpm  |   |   |   |  |
|  spacewalk-proxy-html  |  GPLv2  |  [494292](https://bugzilla.redhat.com/show_bug.cgi?id=494292)  |  msuchy  |  Now in Fedora  |
|  spacewalk-proxy-docs  |  Open Publication  |  [501655](https://bugzilla.redhat.com/show_bug.cgi?id=501655)  |  msuchy  |  Now in Fedora  |
|  spacewalk-backend  |  GPLv2  |  [612581](https://bugzilla.redhat.com/show_bug.cgi?id=612581)  |  msuchy  |  Now in Fedora  |
|  spacecmd  |  GPLv2  |  [616120](https://bugzilla.redhat.com/show_bug.cgi?id=616120)  |  msuchy  |  Now in Fedora  |
|  spacewalk-branding  |  GPLv2  |  [657366](https://bugzilla.redhat.com/show_bug.cgi?id=657366)  |  msuchy  |  Now in Fedora  |
|  spacewalk-web  |  GPLv2  |  [705363](https://bugzilla.redhat.com/show_bug.cgi?id=705363)  |  msuchy  |  Now in Fedora  |
|  spacewalk-pylint  |  GPLv2  |  [800899](https://bugzilla.redhat.com/show_bug.cgi?id=800899)  |  msuchy  |   |  |
|  spacewalk-setup-jabberd  |  GPLv2  |  [807479](https://bugzilla.redhat.com/show_bug.cgi?id=807479)  |  msuchy  |   |  |

There is also a set of jars currently in search-server which are not rpms which will probably need to be packaged up to be accepted into Fedora.
See GettingPackagesIntoFedora/Nutch

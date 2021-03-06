
    #!div class="important" style="border: 2pt solid; text-align: center" 
    '''''DEPRECATED, NO LONGER USED''''' 
## Roadmap

see [Roadmap](https://fedorahosted.org/spacewalk/roadmap) for up to date information. This page is now deprecated.


----

This page holds some feature ideas/themes for future releases of Spacewalk.  These may include RFEs currently received, ideas on the mailing list, and general directions of where we would like to go.  This can all change at any time, but should give you an idea of where we would like to go and where some of the future excitement is.
## Release - 0.1  (Completed: 6/19/08)
 


Initial open sourcing of Spacewalk project.

 * RHEL5 i386 server support
 * RHEL2/3/4/5 client support.
## Release - 0.2



Get packages into Fedora see RepoCleanup and GettingPackagesIntoFedora

Provide client support (via yum-rhn-plugin and associated packages) for
 * Fedora 9
 * CentOS 4
 * CentOS 5

Oracle 10g Database Support (previously was 9i)

Support for installation of Spacewalk on the following OS:
 * Fedora 9 (i386 & x86_64)
 * CentOS 4 & 5 (i386 & x86_64)

[API additions](ApiAdditions)

[Fully support multiple distributions within a single org](NevraIssue)

[Provide for Solaris sun4v client support](SolarisClientSupport)
## Release - 0.3




PostgreSQL DB support - availability << start to get this done

Start work on getting support needed for Fedora 10 changes, such as proposed:
 * DeltaRPM
 * SHA256 support in RPM
 * Improvements to yum-rhn-plugin and supporting components

[Cobbler/Koan Integration](CobblerKoanIntegration): Start work on Cobbler/Koan provisioning integration and replacement of current provisioning system

Command line Proxy Server installation process (deprecate the current WebUI based process)

[Inter Spacewalk Server sync](InterSpacewalkServerSync) - the ability to pull content from one Spacewalk server to the other via command line tools

Further API additions for supporting Multi-Org feature and other calls

[Improvements to the Search capabilities](Features_SearchImprovements) (Index Docs, Errata and System date using a Lucene based engine)

[Solaris Additions](SolarisAdditions)
 * Test and confirm Solaris client works on AMD64 based Solaris 10
 
Start on projects for 0.3 release:
 * Multi-Org Channel (packages & errata) Content Data sharing over multiple Organizations within Spacewalk
 * Ability to migrate/move registered client systems from one Organization to another Organization within the Spacewalk server
 * A support user role, who has read-only access to view data stored within Spacewalk
## Release - 0.4


## Future Ideas

These are not in any specific order


 * DeltaRpmSupport
 * [Support User](Features_SupportUser)
 * Allow for Spacewalk to use different databases, such as either PostgreSQL or MySQL
 * Look to re-factor the /var/satellite/ directory structure for large users of spacewalk who have more than 32,000 unique package names
 * Provide an SELinux policy for running Spacewalk within an Enforcing mode SELinux server.
 * Integrate further with FreeIPA
   * look at single sign on for users via kerberos tickets using mod_kerb (assumption) or some other solution
   * machine identity (beyond the systemid file) and authentication via FreeIPA
 * IPv6 support (not sure what changes, if any we would need to make)
 * Version Control system of Activations Keys & Kickstart Profiles
 * Pulp - what can we get from it, use/integrate with it as an alternative representation of how to get things done within Spacewalk UI
 * oVirt - Does Spacewalk provide management for it, communicate/represent data or something else?
 * Conga - Cluster and storage management - similar to oVirt - does Spacewalk look to provide an alternative management for these tools
 * Allow for integration with and communication/understanding of open source Jaspersoft reporting engine to mine data and reports out of Spacewalks database
 * Use puppet project as an alternative solution to replace our current Configuration File/Directory management tools - provide a richer usage than the basic capabilities we have today
 * Use smolt project as an alternative solution to replace our current hardware profile capturing system - provide a more complete hardware inventory system
 * Use func as an alternative solution to replace our current rhnpush technology (osad communication over jabber protocol servers)
 * [Multiple Organization Support Phase 2](Features_MultiOrg2) - incremental improvements upon the multiple organization support feature

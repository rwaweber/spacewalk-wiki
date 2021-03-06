![Alt](images/spacewalk_2.2.png?raw=True)
# __Spacewalk 2.2 Release Notes__



Hello everyone,

We are proudly announcing release of Spacewalk 2.2, a systems management solution.

The download locations are

  * http://yum.spacewalkproject.org/2.2/RHEL/5/
  * http://yum.spacewalkproject.org/2.2/RHEL/6/
  * http://yum.spacewalkproject.org/2.2/Fedora/19/
  * http://yum.spacewalkproject.org/2.2/Fedora/20/

with client repositories under

  * http://yum.spacewalkproject.org/2.2-client/RHEL/5/
  * http://yum.spacewalkproject.org/2.2-client/RHEL/6/
  * http://yum.spacewalkproject.org/2.2-client/RHEL/7/
  * http://yum.spacewalkproject.org/2.2-client/Fedora/19/
  * http://yum.spacewalkproject.org/2.2-client/Fedora/20/


SUSE Linux client packages can be found here

   * http://download.opensuse.org/repositories/systemsmanagement:/spacewalk:/2.2/SLE_11_SP3/
   * http://download.opensuse.org/repositories/systemsmanagement:/spacewalk:/2.2/openSUSE_12.3/
   * http://download.opensuse.org/repositories/systemsmanagement:/spacewalk:/2.2/openSUSE_13.1/
   * http://download.opensuse.org/repositories/systemsmanagement:/spacewalk:/2.2/openSUSE_Factory/

For fresh installations, please use steps from

  * https://fedorahosted.org/spacewalk/wiki/HowToInstall

If you plan to upgrade from older release, search no more -- the following page will guide you:

  * http://fedorahosted.org/spacewalk/wiki/HowToUpgrade
## Features & Enhancements in Spacewalk 2.2



  * Spacewalk supports RHEL 7 / CentOS 7 clients
  * Read-only API user
    * support for user able to retrieve data via API without the rights to make changes (e.g. for auditing purposes)
  * Action chaining
    * support for defining chain of actions executed on clients
    * http://wiki.novell.com/index.php/SUSE_Manager/ActionChaining
    * Video: http://turing.suse.de/%7Esmoioli/Action%20Chaining%20screencast.webm
  * Remote power management
    * https://fedorahosted.org/spacewalk/wiki/power-management
  * Support for FIPS 140-2
    * Ability to run on a FIPS enabled operating system (client and server)
    * https://fedorahosted.org/spacewalk/wiki/Spacewalk-FIPS
  * Spacewalk Proxy content pre-caching
    * https://fedorahosted.org/spacewalk/wiki/proxy-precache 
  * Further enhancements to the Spacewalk 2.1 Identity management (IPA) integration
    * ​https://fedorahosted.org/spacewalk/wiki/SpacewalkAndIPA 
    * mark roles that were assigned automatically and should thus be removed if not found upon subsequent logins
    * creating default system groups when creating new users
    * external groups to system groups mapping (newly created users may be created as administrators of selected system groups - according to their external group membership and configured mapping) 
  * Plenty of small enhancements and fixes
  * New API calls:
    * actionchain.addConfigurationDeployment
    * actionchain.addPackageInstall
    * actionchain.addPackageRemoval
    * actionchain.addPackageUpgrade
    * actionchain.addPackageVerify
    * actionchain.addScriptRun
    * actionchain.addSystemReboot
    * actionchain.createChain
    * actionchain.deleteChain
    * actionchain.listChainActions
    * actionchain.listChains
    * actionchain.removeAction
    * actionchain.renameChain
    * actionchain.scheduleChain
    * channel.software.syncRepo
    * kickstart.profile.software.getSoftwareDetails
    * kickstart.profile.software.setSoftwareDetails
    * systemgroup.listSystemsMinimal
    * system.listSystemEvents
    * system.provisioning.snapshot.rollbackToSnapshot
    * system.provisioning.snapshot.rollbackToTag
    * system.scheduleCertificateUpdate
    * system.schedulePackageInstall
    * user.external.createExternalGroupToRoleMap
    * user.external.createExternalGroupToSystemGroupMap
    * user.external.deleteExternalGroupToRoleMap
    * user.external.deleteExternalGroupToSystemGroupMap
    * user.external.getDefaultOrg
    * user.external.getExternalGroupToRoleMap
    * user.external.getExternalGroupToSystemGroupMap
    * user.external.getKeepTemporaryRoles
    * user.external.getUseOrgUnit
    * user.external.listExternalGroupToRoleMaps
    * user.external.listExternalGroupToSystemGroupMaps
    * user.external.setDefaultOrg
    * user.external.setExternalGroupRoles
    * user.external.setExternalGroupSystemGroups
    * user.external.setKeepTemporaryRoles
    * user.external.setUseOrgUnit
    * user.getCreateDefaultSystemGroup
    * user.setCreateDefaultSystemGroup
    * user.setReadOnly

The up-to-date API documentation can be found at http://www.spacewalkproject.org/documentation/api/
## Contributors



Our thanks go to the community members who contributed to this release:

* Avi Miller
* Bo Maryniuk
* Carsten Menzel
* Colin Coe
* Daniel Igel
* Dimitar Yordanov
* Duncan Mac-Vicar
* Flavio Castelli
* Gregor Gruener
* Hubert Mantel
* Jan Pazdziora
* Jeremy Davis
* Jiri Mikulka
* Johannes Renner
* Kumudini Shirsale
* Lukas Pramuk
* Marcelo Moreira de Mello
* Martin Seidl
* Michael Calmer
* Michele Baldessari
* Miroslav Suchý
* Neha Rawat
* Pierre Casenove
* Ron van der Wees
* Shannon Hughes
* Silvio Moioli
* Tasos Papaioannou
* Tobias D. Oestreicher


https://fedorahosted.org/spacewalk/wiki/ContributorList
## Some statistics



In Spacewalk 2.2, we've seen

    * 99 bugs fixed
    * 1308 changesets committed
    * 1760 commits done

Github repo for commits since Spacewalk 2.1

    * [Spacewalk 2.1 to 2.2](https://github.com/spacewalkproject/spacewalk/graphs/contributors?from=2014-03-04&to=2014-07-16&type=c)
## Spacewalk 2.2 on RHEL 5 (CentOS 5) and RHEL 7 (CentOS 7)



Due to missing package dependencies in EPEL 7 beta, we were not able to deliver Spacewalk 2.2 on RHEL 7 (CentOS 7). Spacewalk 2.2 therefore still
supports running on RHEL 5 and CentOS 5 as a base operating system. Support for RHEL 7 and CentOS 7 should be available in next Spacewalk release, at which
point support for RHEL 5 and CentOS 5 will be dropped. Note that RHEL 7 / CentOS 7 clients are supported with Spacewalk 2.2, just not running Spacewalk itself on those operating systems.
## Solaris and Monitoring Support - Deprecation Notice



The Spacewalk team is looking in future releases to drop support for Solaris clients and the Monitoring component of Spacewalk. They continue to be supported in their current state for the Spacewalk 2.2 release. Anyone currently using either of the capabilities may wish to consider alternatives for their needs. 
## User community, reporting issues



To reach the user community with questions and ideas, please use
mailing list spacewalk-list@redhat.com. On this list, you can of
course also discuss issues you might find when installing or using
Spacewalk, but please do not be surprised if we ask you to file a bug
at https://bugzilla.redhat.com/enter_bug.cgi?product=Spacewalk with more
details or full logs.

Thank you for using Spacewalk.

*_DEPRECATED, NO LONGER USED*_

*Things to do*:
    
    * Support sun4v to same extent as other hardware platforms we currently support, such as sun4u - Fixed and development side testing done. 
   
*Solaris Client Changes*:

    * Fix rhnclient/rhn/client/translate.py getArch() to support sun4v 

    * Fix rhnclient/rhn/client/rhnUtils.py getUnameArch() to support sun4v 

*Database Schema Changes*:

    * Fix eng/schema/rhnsat/tables/rhnCpuArch_data.sql, rhnPackageArch_data.sql, rhnServerArch_data.sql, rhnServerChannelArchCompat_data, rhnServerServerGroupArchCompat_data.sql, rhnServerPackageArchCompat_data.sql, rhnChannelPackageArchCompat_data.sql to support sun4v 

    * Fix eng/schema/rhnsat/tables/rhnServerPackageArchCompat_data.sql to add compatibility of "sun4u" with noarch packages and patches. 

*Customer Bugs/Request for this*:

    * Bug 238435: Satellite Support for Solaris Sun4v - https://bugzilla.redhat.com/show_bug.cgi?id=238435
    * Bug 431502: Architecture `sparc-sun4v-solaris' is not supported - https://bugzilla.redhat.com/show_bug.cgi?id=431502 

   
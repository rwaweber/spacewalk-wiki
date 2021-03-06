
    #!div class="important" style="border: 2pt solid; text-align: center"
    '''''DEPRECATED, NO LONGER USED'''''
## Questions



The following are notes from a discussion between jdob and dgoodwin regarding the definition of the support user role and its function.

There seem to be two things being discussed in relation to the support user role:
 * Read only access to the web UI and APIs
 * Auditing features (for instance, system searching across trusted orgs)

For the purposes of this discussion, all of the new auditing features (the new search interface and its results) are presumed to fall under a new UI tab (the Support User tab).

These two features lead to a number of questions:
 1. Does a user with the Support User role see all of the default tabs (Systems, Channels, etc.) or simply the Support User tab?
  * If the support user does see all tabs, we'll have to add checks on all of these pages to ensure that any writeable functionality is disabled.
   * Despite being a lot of legwork, we run into a conflict if a user is assigned the Support User role and another role (e.g. System Groups Admin). Do we assume that the read only nature of the Support User role is overridden by having another (admin) role assigned?
 1. Along the same lines, can the Support User role be assigned in conjunction with other roles?
  * This would be desirable if, for instance, an admin also wanted access to the new search/audit abilities on the new tab.
  * Again, we run into the question of how the read only aspect of the Support User role gets resolved with the change functionality provided by admin roles.
 1. Are the audit features and read only user necessarily the same thing? Should they be implemented as two distinct options, an Audit role and a read only flag?
  * This approach makes the Support User tab a simple addition if the role is present, similar to the Admin menu for admins.
  * Again, this requires reworking the entire UI with checking to disable writeable actions when the user is flagged as read only.
  * There may be a desire for access to certain admin menus with read only access, however this seems like a contradiction to have a read only admin. In fact, the way we have our roles set up, I think the read only flag would negate almost everything granted by the admin role.
## Proposed Solution



As a result of the above questions, the following is the usage of the support user role:
 1. The only read only functionality exists within the Support User tab. All existing pages retain their existing role permissions.
  * The pages inside of this tab should be implemented using snippets from existing pages without writeable actions (will likely require some refactoring of existing pages and potentially some porting from perl to java).
  * The Support User tab is read only no matter what other roles the user possesses. In other words, even the satellite admin will still not see writeable actions from within the Support User tab.
  * The Support User tab will be flushed out in the future with more search options. In this iteration, only system searches (across multiple trusted orgs) will be implemented.
 1. For users that only have the Support User role, the only tabs displayed are: Overview, Support User tab, and Help.
  * This is *different* than a no-role user, which also has access to Systems, Errata, Channels, and Schedule. A no-role user is able to access writeable actions on those tabs (e.g. delete system, package install).
  * The effect of this tab restriction is that users with only the Support User role effectively become read only users.
 1. For users that have roles in addition to the Support User role, the Systems, Errata, Channels, and Schedule tabs will appear in addition to the Support User tab.
  * This is a little wonky because the no-role user has so many default permissions rather than having none at all. One idea is to convert the no-role user into a User role, however that also carries some complications (e.g. what happens if you make a System Groups admin without the User role?).
  * To reiterate, no changes occur inside of the Support User tab if the user has other roles; it still does not offer any writeable actions. All other tabs continue to function as they do currently.
  * In other words, adding the Support User role to a user with other enhanced permissions (i.e. other roles) is simply to add the Support User tab to their interface.

This approach is (almost) consistent with our existing approach to roles. It is an option that increases functionality for a user in the sense that users given this role have access to the specific tab and search features. The difference (and potential confusion) is that for a user that is *only* a support user will have tabs removed (see above). This is a compromise between a fully hierarchical role structure (i.e. regular user is support user plus new features, admin is regular user plus new features) and our existing bolt-on functionality approach (i.e. users with a particular role get access to new features on top of the default base set that *all* users get).


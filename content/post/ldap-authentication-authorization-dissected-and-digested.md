---
title: "LDAP Authentication &amp; Authorization Dissected and Digested"
date: "2011-01-04"
categories: 
  - "development"
tags: 
  - "authentication"
  - "ldap"
---

LDAP is one of those things that I've integrated with a few times but never put enough energy into to really get the details or understand it much.  There's always been someone I can bounce questions off of and thankfully those people were available again as I started working out the details of performing LDAP authentication.

The steps below are general enough to be used by anyone and will hopefully shed some light into the steps performed in LDAP authentication.  The process below also includes some steps for authorization.

### Authentication

_1.  Get a connection to the LDAP server._ With the host and port for the LDAP server, create a connection to it.  This can be  a simple direct connection or a pooled connection.  If more than a basic test, it is best to use a pooled connection.  Log and fail if a connection cannot be created.

_2.  Bind as the application user._ Bind the connection to the application user.  This user should have enough permissions to search the area of LDAP where users are located.  This user may also have permissions to search for things outside of the users area (groups, authorization).  Log and fail if the application user cannot bind.

_3.  Search for the DN (distinguished name) of the user to be authenticated._ This is where we verify the username is valid.  This does not authenticate the user but simply makes sure the requested username exists in the system.  Log and fail if the user's DN is not found.

_4.  Bind as user to be authenticated using DN from step 3._ Now for the moment of truth.  Bind to the connection using the DN found in step 3 and the password supplied by the user.  Log and fail if unable to bind using the user's DN and password.

### Authorization

_5.  Re-bind as application user._ To check the authorization of a user, we need to read attributes from the user's account. To do this, we need to re-bind as the application user.

_6.  Search for user and require attributes._ A filter is used to search for the user like was done in step 3 but we'll add an extra check to the query to look for the attributes that show we're authorized.

### Example Code

The code shown here is using the JLDAP library from Novell.  Interesting includes are noted at the top. The utility class used for escaping the search filter can be found in the [Sakai Project's Nakamura codebase](https://github.com/thecarlhall/nakamura/blob/master/contrib/ldap/src/main/java/org/sakaiproject/nakamura/api/ldap/LdapUtil.java).

```java
import com.novell.ldap.LDAPAttribute;
import com.novell.ldap.LDAPConnection;
import com.novell.ldap.LDAPEntry;
import com.novell.ldap.LDAPException;
import com.novell.ldap.LDAPSearchResults;
// ...
String baseDn = "ou=People,o=MyOrg";
String userFilter = "uid = {}";
String authzFilter = "authzAttr=special:Entitlement:value";
// ...
public boolean authenticate(Credentials credentials) throws RepositoryException {
    boolean auth = false;
    if (credentials instanceof SimpleCredentials) {
        // get application user credentials
        String appUser = connMgr.getConfig().getLdapUser();
        String appPass = connMgr.getConfig().getLdapPassword();

        // get user credentials SimpleCredentials sc = (SimpleCredentials) credentials;

        long timeStart = System.currentTimeMillis();

        String userDn = LdapUtil.escapeLDAPSearchFilter(userFilter.replace("{}", sc.getUserID()));
        String userPass = new String(sc.getPassword());

        LDAPConnection conn = null;
        try {
            // 1) Get a connection to the server
            try {
                conn = connMgr.getConnection();
                log.debug("Connected to LDAP server");
            } catch (LDAPException e) {
                throw new IllegalStateException("Unable to connect to LDAP server \[" + connMgr.getConfig().getLdapHost() + "\]");
            }

            // 2) Bind as app user
            try {
                conn.bind(LDAPConnection.LDAP_V3, appUser, appPass.getBytes(UTF8));
                log.debug("Bound as application user");
            } catch (LDAPException e) {
                throw new IllegalArgumentException("Can't bind application user \[" + appUser + "\]", e);
            }

            // 3) Search for username (not authz).
            // If search fails, log/report invalid username or password.
            LDAPSearchResults results = conn.search(baseDn, LDAPConnection.SCOPE_SUB, userDn, null, true);
            if (results.hasMore()) {
                log.debug("Found user via search");
            } else {
                throw new IllegalArgumentException("Can't find user \[" + userDn + "\]");
            }

            // 4) Bind as user.
            // If bind fails, log/report invalid username or password.

            // value is set below. define here for use in authz check.
            String userEntryDn = null;
            try {
                LDAPEntry userEntry = results.next();
                LDAPAttribute objectClass = userEntry.getAttribute("objectClass");

                if ("aliasObject".equals(objectClass.getStringValue())) {
                    LDAPAttribute aliasDN = userEntry.getAttribute("aliasedObjectName");
                    userEntryDn = aliasDN.getStringValue();
                } else {
                    userEntryDn = userEntry.getDN();
                }

                conn.bind(LDAPConnection.LDAP_V3, userEntryDn, userPass.getBytes(UTF8));
                log.debug("Bound as user");
            } catch (LDAPException e) {
                log.warn("Can't bind user \[{}\]", userDn);
                throw e;
            }

            if (authzFilter.length() > 0) {
                // 5) Return to app user
                try {
                    conn.bind(LDAPConnection.LDAP_V3, appUser, appPass.getBytes(UTF8));
                    log.debug("Rebound as application user");
                } catch (LDAPException e) {
                    throw new IllegalArgumentException("Can't bind application user \[" + appUser + "\]");
                }

                // 6) Search user DN with authz filter
                // If search fails, log/report that user is not authorized
                String userAuthzFilter = "(&(" + userEntryDn + ")(" + authzFilter + "))";
                results = conn.search(baseDn, LDAPConnection.SCOPE_SUB, userAuthzFilter, null, true);
                if (results.hasMore()) {
                    log.debug("Found user + authz filter via search");
                } else {
                    throw new IllegalArgumentException("User not authorized \[" + userDn + "\]");
                }
            }

            // FINALLY!
            auth = true;
            log.info("User \[{}\] authenticated with LDAP in {}ms", userDn, System.currentTimeMillis() - timeStart);
        } catch (Exception e) {
            log.warn(e.getMessage(), e);
        } finally {
            connMgr.returnConnection(conn);
        }
    }
    return auth;
}
```

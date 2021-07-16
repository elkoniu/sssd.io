SSSD and shortnames
###################

In the simplest configuration SSSD can serve single domain.
In this case user / group query will be unambiguous as only one domain can provide the answer.
Multi domain setup is more complicated as the same user / group can be registrated in multiple domains.
To make query unique in this scenario fully qualified name need to be used:

.. note::

    Fully "qualified" user name is the situation when username is followed with domain name,
    for example bob@corp.abc.com instead of just "bob". On Windows systems it can be called
    as SAM account (Security Account Manager) and writted different way: domain_name\user_name.
    UPN (User Principal Name) is another alias for this naming convention.
    In case of Active Directory (AD) this type of entry may be called DN
    (Distinguished Name) where separated blocks will have value:
    CN=bob,OU=Users,DC=corp,DC=abc,DC=net

When fully qualified user name is used query will be send only to single domain
used as a part of the name.

Example:
Imaginate that sssd.conf has 3 domains: aaa.com, bbb.com and ccc.com
Then system administrator will trigger query: getent passwd bob
In this scenario SSSD will enumerate over domains in the order they are presented
in configuration file and "first" bob found will be returned.
If administrator want to specify which "bob" he is looking for fully qualified name
should be used (bob@aaa.com, bob@bbb.com, bob@ccc.com).

The problem is that this behaviour is not always wanted. In most cases users prefer
just short username to login into system. Another example is mapping between user
name and user directroy in the system. Administrator may want user directories
to be named in generic way, rather than including domain name in it.

SSSD supports short names, where domain name can be skipped.
To force specified domain resolution order sssd.conf has special configuration
option called "domain_resolution_order". This allows administrator to sort
set in what order domains needs to be queried.

Alternative way is to use "default_domain_suffix". This option allows administrator
to set "hiden default" domain name when it is not written explicite in query.


SSSD configuration options related to shortnames
************************************************

.. note::

    use_fully_qualified_names (bool)

        Use the full name and domain (as formatted by the domain's full_name_format) as the user's login name reported to NSS.

        If set to TRUE, all requests to this domain must use fully qualified names. For example, if used in LOCAL domain that contains a "test" user, getent passwd test wouldn't find the user while getent passwd test@LOCAL would.

        NOTE: This option has no effect on netgroup lookups due to their tendency to include nested netgroups without qualified names. For netgroups, all domains will be searched when an unqualified name is requested.

        Default: FALSE (TRUE if default_domain_suffix is used)

.. note::

    domain_resolution_order

        Comma separated list of domains and subdomains representing the lookup order that will be followed. The list doesn't have to include all possible domains as the missing domains will be looked up based on the order they're presented in the “domains” configuration option. The subdomains which are not listed as part of “lookup_order” will be looked up in a random order for each parent domain.

        Please, note that when this option is set the output format of all commands is always fully-qualified even when using short names for input. In case the administrator wants the output not fully-qualified, the full_name_format option can be used as shown below: “full_name_format=%1$s” However, keep in mind that during login, login applications often canonicalize the username by calling getpwnam(3) which, if a shortname is returned for a qualified input (while trying to reach a user which exists in multiple domains) might re-route the login attempt into the domain which users shortnames, making this workaround totally not recommended in cases where usernames may overlap between domains.

        Default: Not set

.. note::

    default_domain_suffix (string)

        This string will be used as a default domain name for all names without a domain name component. The main use case is environments where the primary domain is intended for managing host policies and all users are located in a trusted domain. The option allows those users to log in just with their user name without giving a domain name as well.

        Please note that if this option is set all users from the primary domain have to use their fully qualified name, e.g. user@domain.name, to log in. Setting this option changes default of use_fully_qualified_names to True. It is not allowed to use this option together with use_fully_qualified_names set to False.

        Default: not set

.. note::

    override_homedir (string)

        Override the user's home directory. You can either provide an absolute value or a template. In the template, the following sequences are substituted:

        %u
            login name

        %U
            UID number

        %d
            domain name

        %f
            fully qualified user name (user@domain)

        %l
            The first letter of the login name.

        %P
            UPN - User Principal Name (name@REALM)

        %o
            The original home directory retrieved from the identity provider.

        %H
            The value of configure option homedir_substring.

        %%
            a literal '%'

        This option can also be set per-domain.

        example:

            override_homedir = /home/%u

        Default: Not set (SSSD will use the value retrieved from LDAP)

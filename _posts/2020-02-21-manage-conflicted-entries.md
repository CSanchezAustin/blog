---
layout: post
title: "Fixing Directory Services conflicted entries"
subtitle:  "Or Mr. Fixit is conflicted..."
categories: [TechNote, identity, opendj, bash]
tags: [identity, opendj, bash]
readtime: 2 min read
author: Chris Sanchez
---
As a ForgeRock Directory Services owner/operator one has to regularly review logs to catch any number of operational problems that may surface. On problem that you may encounter are conflicted entries. 

This happens due to  conflicts during replication, meaning that a DS replica has already applied a change in it's database when it receives the replication event to update, delete or add the same entry. Directory Services will preserve the change by creating the conflicted entry that's distinguishable by it's DN (which now looks like: entryuuid=entryUUID-value+original-RDN,original-parent-DN) with the addition of an operational atribute `ds-sync-conflict` . ForgeRock has a KB article detailing [conflicted entries] and some [product documentation] that talks about the fix. 

To find conflicted entries you can use `ldapsearch` to find entries that have the `ds-sync-conflict` attribute. 

~~~~~
echo "#### Checking for conflicts"
/opt/opendj/bin/ldapsearch          \
--bindDN "cn=Directory Manager"     \
--bindPassword "password"           \
--hostname localhost                \
--port 1389                         \
--trustAll                          \
--baseDN "dc=zibernetics,dc=com"    \
'(ds-sync-conflict=*)' 1.1
~~~~~

Generates output that looks something like this:
~~~~~
dn: entryuuid=45e10549-78ab-4205-95cb-59fdf56ee59c+uid=851b60ca-4643-4789-aef6-b792e3fe680f,ou=People,dc=zibernetics,dc=com

dn: entryuuid=2cfa5520-8486-43fe-a416-85d85474f2fc+uid=d048ba75-025a-429f-828a-a037098423e8,ou=People,dc=zibernetics,dc=com
~~~~~
#### Fixing Conflicted Entries
Of course, each entry should be carefully examined per the [product documentation] to ensure accurate resolution. The following gist assumes that all changes that need to be applied to replicas in the cluster to achieve consistency have been performed and the conflicted entries can be deleted.

The key to getting this to work is on line 19 `'(ds-sync-conflict=*)' 1.1` which just returns the DN and the `awk` program on the next line `'$1 == "dn:" { print $0; print "changetype: delete"; print ""}'` which captures the DN and generates a little bit of ldif to delete the entry. Finally it's fed to `ldapmodify` to perform the changes using `--numConnections` option which is useful for opening multiple connections to the LDAP server for concurrent execution of the ldif.
 
{% gist 63d6c2165032c759cb419e0dc5547769 %}

I hope you find this useful. Feel free to submit a PR for this article if you have improvements.

[product documentation]: https://backstage.forgerock.com/docs/ds/6/admin-guide/#repl-conflict
[conflicted entries]: https://backstage.forgerock.com/knowledge/kb/article/a37856549

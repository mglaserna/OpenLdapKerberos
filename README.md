# OpenLdapKerberos


<!-- CSS work goes here for the time being -->
<!-- set a:link text-decoration to none -->
<!-- set a:hover text-decoration to underline -->
<!-- http://forums.markdownpad.com/discussion/143/include-pdf-pagebreak-instructions-in-markdown/p1 -->

---

## <center> <a name="cm_cdh_installation_section"/>Cloudera Manager & CDH Installation

* <a href="#install_methods">OpenLdap Installation</a>
* <a href="#parcels">Kerberos Installation</a>
* <a href="#db_setup">OpenLdap and Kerberos Integration</a>
* <a href="#cm_cdh_key_points">Connecting Cloudera Hadoop to OpenLdap+Kerberos</a>
---
<div style="page-break-after: always;"></div>

## <center> <a name="install_methods"/> OpenLdap Installation

 # <center> Steps for OpenLdap Installation
 
  1. Install openldap ( yum -y install   compat-openldap openldap-clients openldap-servers openldap-servers-sql ) 
  2. Create ldap password and store in a secure place (slappasswd > /tmp/secured.txt)
  3.  Create temporary directory to store ldif files (eg: mkdir /root/ldif)
  4. Download ldif files located at then put at temporary directory : https://github.com/mglaserna/OpenLdapKerberos/tree/master/ldif/ldif
  5. Edit all ldif files appropriately 
     * Edit object olcRootPW with encrypted password from slappaswd (monitor.ldif)
     * edit dn objects olcRootPW and olcRootDn to appropriate domain (monitor and other ldif with dn objects)
  6. Start slapd service if not yet started
  7. Execute and modify slapd configs
  		* ldapmodify -Y EXTERNAL  -H ldapi:/// -f  db.ldif and monitor.ldif
  		* add schemas to ldap `for a in cat list_ldif.lst; do   ldapadd -Y EXTERNAL -H ldapi:/// -f ${a} ; done`
  		* Add base items, Users and Groups `ldapadd -x -W -D "cn=ldapadm,dc=novare,dc=com,dc=hk" -f {base,users,groups}.ldif`
  		* Modify acl of ldap `ldapmodify -Y EXTERNAL  -H ldapi:/// -f  acl.ldif
`
        * Do an ldap search to check if users has been sync `ldapsearch -x cn=sales1 -b dc=novare,dc=com,dc=hk
`

     

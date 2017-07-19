<!-- CSS work goes here for the time being -->
<!-- set a:link text-decoration to none -->
<!-- set a:hover text-decoration to underline -->
<!-- http://forums.markdownpad.com/discussion/143/include-pdf-pagebreak-instructions-in-markdown/p1 -->

---

## <center> <a name="cm_cdh_installation_section"/>Cloudera Manager & CDH Installation

* <a href="#install_methods">OpenLdap Installation</a>
* <a href="#phpldapinstall">PhpLdapAdmin Installation</a>
* <a href="#kerbeldapintege">OpenLdap and Kerberos Integration and installation </a>
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

     

---
<div style="page-break-after: always;"></div>

## <center> <a name="phpldapinstall"/>Steps for phpldapinstallation
 
* For steps please look at http://www.itzgeek.com/how-tos/linux/centos-how-tos/install-configure-phpldapadmin-centos-7-ubuntu-16-04.html
---
<div style="page-break-after: always;"></div>

## <center> <a name="kerbeldapintege"/>Steps for Integrate Openldap to kerberos

1. Install Kerberos `yum install -y krb5-pkinit-openssl krb5-libs krb5-server-ldap krb5-server`
2. Configure schema for ldap
	* `ldapadd -Y EXTERNAL  -H ldapi:/// -f  kerberos.ldif` - add schema  
	*  Add `olcDbLinearIndex: FALSE` to `/etc/openldap/slapd.d/cn\=config/olcDatabase\=\{2\}hdb.ldif` then execute ` sed  "s/olcDbLinearIndex: FALSE/olcDbIndex: krbPrincipalName eq,pres,sub\nolcDbLinearIndex: FALSE/g" /etc/openldap/slapd.d/cn\=config/olcDatabase\=\{2\}hdb.ldif -i 
` then restart
   * `ldapadd  -D "cn=ldapadm,dc=novare,dc=com,dc=hk"   -H ldapi:/// -f entitiy_kerberos.ldif  -W` check if kerberos objects has been created `ldapsearch -LLLY EXTERNAL -H ldapi:/// -b ou=services,dc=novare,dc=com,dc=hk dn
`
3. Configure krb5 conf files (krb5 conf,acl and krb5kdc conf) 
	* Download conf files and edit appropriately 
4. Execute `kdb5_ldap_util -D "cn=$LDAP_ADMIN_USER,dc=novare,dc=com,dc=hk" create -subtrees "cn=kerberos,ou=services,dc=novare,dc=com,dc=hk" -r NOVARE.COM.HK -s -H ldapi:/// -w passw0rd` to create kerberos database and create stashpw ` kdb5_ldap_util -D "cn=$LDAP_ADMIN_USER,dc=novare,dc=com,dc=hk" create -subtrees "cn=kerberos,ou=services,dc=novare,dc=com,dc=hk" -r NOVARE.COM.HK -s -H ldapi:/// -w passw0rd
`



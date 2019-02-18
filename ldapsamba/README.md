# SAMBASERVER BACKEND LDAP

## Parveen ASIX M06 2018-2019


## Description:

Practica with ldap,pam_host and sambaserver backend .Main to config. of samba server. 

### important steps:


Make sambanet network to put togather from exemple name by (sambanet)
now we need add all docker in this network

By using this command

```
[root@localhost pam_ssh_ldap]# docker network create sambanet
8ff156696dccf766367d5fbb6814b46e2f4b286dd4ea73d980fbfd67a943671e
```

## 1 Ldap Server
	* Dockerfile need to edit some pacquetes  (procps openldap-clients openldap-servers).
	* Put database edt.org also new group and manager as admin
	* Copy samba.schema for add new data
	* Chown file data  and directoary conf.
	* Turn on server slpad


This one my server from use
	
```
docker run --rm --network sambanet --name ldap --hostname ldap -d parveen1992/ldapsamba
```

**check this by correct ip address my ip is 172.20.0.2**

```
[root@localhost ldapsamba]# docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS               NAMES
3e780ccc6320        parveen1992/ldapsamba   "/opt/docker/start..."   4 minutes ago       Up 4 minutes        389/tcp             ldap
[root@localhost ldapsamba]# ldapsearch -x -LLL -h 172.20.0.2 -b dc=edt,dc=org 'ou=grups'
dn: ou=grups,dc=edt,dc=org
ou: groups
ou: grups
description: Container per a grups
objectClass: organizationalunit

```

## 2 samba Server backend
	
	* Docker line need add more packets  (procps passwd samba samba-client openldap-clients nss-pam-ldapd authconfig pam_mount smbldap-tools).
	* Make some localuser and group late for check test.also need to give password.
	* Now we need cp file accordig to setting in (/opt/docker/auth.sh or /etc/nslcd.conf,/etc/openldap/ldap.conf,/etc/nsswitch.conf)to connect ldapsamba server
	* Now we need connection to ldap server befor we make (Ldapsamba server).
	* Start server to connect ldap server(nscd,nslcd).
	* After connecect make home directory for samba user.
	* mkdir for every user and import give own premission.
	* put some file samba public and private(/var/lib/samba).
	* copy smb.conf in (/etc/samba/smb.conf).
	* now need to tools in conf. for backend 
	* smbldap_bind.conf in /etc/samldap-tools/.
	* put sercet password.
	* now start server (smbd,nmbd).
	This one my server to user
	
```
docker run --rm --network sambanet --name sambasam --hostname sambasam -it parveen1992/sambasam
```

**check this**
	
```
[root@sambasam docker]# getent passwd pere
pere:*:5001:100:Pere Pou:/tmp/home/pere:

[root@sambasam docker]# pdbedit -L
pau:5000:Pau Pou
pere:5001:Pere Pou
anna:5002:Anna Pou
marta:5003:Marta Mas
jordi:5004:Jordi Mas
admin:10:Administrador Sistema
root:0:root
nobody:99:Nobody

[root@sambasam docker]# ldapsearch -x -LLL 'uid=pere'
dn: uid=pere,ou=usuaris,dc=edt,dc=org
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: sambaSamAccount
cn: Pere Pou
sn: Pou
homePhone: 555-222-2221
mail: pere@edt.org
description: Watch out for this guy
ou: Profes
uid: pere
uidNumber: 5001
gidNumber: 100
homeDirectory: /tmp/home/pere
sambaSID: S-1-5-21-2274168011-4190508070-1264377906-1001
displayName: Pere Pou
userPassword:: e1NTSEF9RisxTThtM3NMeEFHUTE3eXpndkxIZ1RiaFR5aXozNjM=
sambaNTPassword: 394248C9B811DB06DA043F2455B0A4BD
sambaPasswordHistory: 00000000000000000000000000000000000000000000000000000000
 00000000
sambaPwdLastSet: 1550446590
sambaAcctFlags: [U          ]
```


## 3 Pamhost or cliente
	
	* Docker line need add more packets (procps passwd openldap-clients nss-pam-ldapd authconfig pam_mount cifs-utils).
	* Make confgure file for ldapserver connection (nsswitch.conf).
	* ALso need pam system conf. for login user and mkhomedir (/etc/pam.d/system-auth.edt) .
	* file to copy with well conf.(/etc/nslcd.conf,/etc/openldap/ldap.conf,)
	* Auto mount pam file need /etc/security/pam_mount.conf.xml
	* xml file conf. (<volume user="*" fstype="cifs" server="samba" path="%(USER)"  mountpoint="~/%(USER)" />).
	* Start server to connect ldap server(nscd,nslcd).
	This my PamHost is
	
```
docker run --rm --network sambanet --privileged --name client  --hostname client -it parveen1992/hostmountsam
```

**check this** 
	
```
root@client docker]# su - local01
[local01@client ~]$ ll
total 0
[local01@client ~]$ pwd
/home/local01

[root@client docker]# su - anna
reenter password for pam_mount:
[anna@client ~]$ ll
total 0
drwxr-xr-x+ 2 anna alumnes 0 Feb 17 23:36 anna

[anna@client ~]$ mount -t cifs
//sambasam/anna on /tmp/home/anna/anna type cifs (rw,relatime,vers=1.0,cache=strict,username=anna,domain=,uid=5002,forceuid,gid=600,forcegid,addr=172.20.0.3,unix,posixpaths,serverino,mapposix,acl,rsize=1048576,wsize=65536,echo_interval=60,actimeo=1)

```	



### Github repo. with all of images file we need conf.(or your castom conf.). 

[Github Parveen](https://github.com/parveen1/ldapsambasam)


### Dockerhub repo. links

[Docker Parveen ldapsamba](https://hub.docker.com/r/parveen1992/ldapsamba)

[Docker parveen sambasam](https://hub.docker.com/r/parveen1992/sambasam)

[Docker Parveen hostmountsam](https://hub.docker.com/r/parveen1992/hostmountsam)

### All to start

```
docker run --rm --network sambanet --name ldap --hostname ldap -d parveeen1992/ldapsamba
docker run --rm --network sambanet --name sambasam --hostname sambasam -it parveen1992/sambasam
docker run --rm --network sambanet --privileged --name client  -hostname client -it parveen1992/hostmountsam
```

### Check this order after strat to verify

```
	[pere@client ~]$ su - anna
	pam_mount password:
	Creating directory '/tmp/home/anna'.

	[anna@client ~]$ pwd
	/tmp/home/anna
```

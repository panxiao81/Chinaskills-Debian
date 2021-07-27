介绍忽略不表，直接去看 Wikipedia

Windows Active Directory 是 LDAP 的一种实现，而 Linux 上通常使用 OpenLDAP

LDAP 在 Linux 上请部署 `slapd`

直接安装软件包即可。

如果需要的话，执行 `dpkg-reconfigure slapd` 进行部分设置

LDIF 文件太难写，比赛如果不提供 Web 界面，就使用 MigrationTools 将本地用户合并进 LDAP

`apt install migrationtools`

编辑 `/etc/migrationtools/migrate_common.ph` 加入域名，确认排除的用户范围

安装的所有脚本均存放在 `/usr/share/migrationtools/` 下面所有当前目录均为此目录。

一次性导入基本结构，并合并 `/etc/passwd` 与 `/etc/group` 

```sh
LDAPADD="/usr/bin/ldapadd -c" ETC_ALIASES=/dev/null ./migrate_all_online.sh
```

脚本实现有些问题，这个锅发行版背，至少当前用的最新版镜像还没修复。

需要把 `migrate_common.ph` 手动符号链接到 `/etc/perl`

该脚本会询问一些问题，例如 X.500 名称等：见下面表格：

根据实际情况修改填写

| Question | Answer |
| --- |  --- |
| X.500 naming context | `dc=falcot,dc=com` |
| LDAP server hostname | `localhost` |
| Manager DN | `cn=admin,dc=falcot,dc=com` |
| Bind credentials | the administrative password |
| Create DUAConfigProfile | no |

合并时会提示错误，这是正常的，因此需要额外指定 `ldapadd` 的 `-c` 参数，使其正常运行

---

配置 ldaps

生成证书后，确保保存位置允许 `openldap` 用户读

写文件，并配置进 LDAP

```sh
$ cat >ssl.ldif << EOF
dn: cn=config
changetype: modify
add: olcTLSCertificateFile
olcTLSCertificateFile: /etc/ssl/certs/ldap.falcot.com.pem
-
add: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/ssl/private/ldap.falcot.com.key
-
EOF
$ ldapmodify -Y EXTERNAL -H ldapi:/// -f ssl.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "cn=config"
```

或者用 ldapvi 直接修改：

```sh
$ ldapvi -Y EXTERNAL -h ldapi:/// -b cn=config
```

最后修改 `/etc/default/slapd`,修改 `SLAPD_SERVICES` 为仅允许 LDAPS 与 LDAPI ( Unix Socket )

```sh
SLAPD_SERVICES="ldaps:/// ldapi:///"
```


> https://www.debian.org/doc/manuals/debian-handbook/sect.ldap-directory.zh-cn.html
>
> https://wiki.debian.org/LDAP/MigrationTools
>
> https://www.openldap.org/doc/admin24/quickstart.html
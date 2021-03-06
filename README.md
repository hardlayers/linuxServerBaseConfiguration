# Hardlayers Servers Baseline Configuration

This repo contains the basic configurations that exist on hardlayers servers before coming online.

## linuxServerBaseConfiguration

Base configurations for linux servers

### Setup environment

#### Install Nano if it is not installed

```Shell
yum install nano
```

#### Symlink Pico

```Shell
ln -sv /usr/bin/nano /usr/bin/pico
```

#### Configure SSH

Change Port / Disable Root login / ...etc.

```Shell
pico /etc/ssh/sshd_config
#port 22
/etc/init.d/sshd reload
```

#### Configure Timezones

Server Configuration

- Basic cPanel/WHM Setup
- Update Config

#### Server Contacts

- Change System Mail Preferences and put `servers@hardlyers.net.sa`

#### Security

- `/scripts/apachelimits`
- `/scripts/mysqlpasswd`
- `/scripts/securemysql -q -F -a removeanon,removelockntmp,removeremoteroot`

#### Security Center

- PHP open_basedir Tweak (Enable)
- Apache mod_userdir Tweak (Enable)
- `/scripts/userdirctl on`
- Compilers Tweak (Disable)
- Shell Fork Bomb Protection (Enable)

#### FTP Configuration

- pure-ftpd
- Disable Anonymous

#### System Health

- Background Process Killer

#### Configure Hostname

```Shell
pico /etc/hosts
pico /etc/sysconfig/network
```

set
HOSTNAME=myserver.domain.com

```Shell
hostname hostname.domain.com
hostname #to check
/etc/init.d/network restart
```

#### Reboot

```Shell
reboot
```

### Install cPanel (if required)

```Shell
cd /home && curl -o latest -L https://securedownloads.cpanel.net/latest && sh latest
# Update License
/usr/local/cpanel/cpkeyclt
```

- Go through settings and updates
- Setup Apache as required
- Name server (Bind)
- Configure nameservers domains
- Configure Address Records for Nameservers & Hostname
- FTP Server PureFTPd
- Enable cPHulk
- Install Common Set of Perl Modules
- Enable
  ** DKIM
  ** SPF
  ** Thunderbird and Outlook autodiscover and autoconfig support
  ** Email delivery retry time (15 Minutes)
  ** Track email origin via X-Source email headers
  ** Notify admin or reseller when disk quota reaches “warn” state
- Disable Compilers
- Install ClamAV
- Apache Symlink Protection

### Secure Temp

```Shell
pico /etc/fstab
```

set
no exec on temp
or run

```Shell
/scripts/securetmp
```

### Configure MySQL

/etc/my.cnf  
install openVPN

### Install CSF

```Shell
cd /usr/src
rm -fv csf.tgz
wget https://download.configserver.com/csf.tgz
tar -xzf csf.tgz
cd csf
sh install.sh
sh /usr/local/csf/bin/remove_apf_bfd.sh
```

- Configure it as needed
- Enable Passive mode for pureFTPd and open required ports

### Apache Configureations

Before or during production, ensure handling of CGI and Symlinks is only allowed as per requirements.
Following settings disable CGI and Symlinks globally

#### 1. Create HL.conf file

```Shell
pico /usr/local/apache/conf/HL.conf
```

Place the following inside

```ApacheConf
<Directory "/">
	Options +Indexes -ExecCGI -FollowSymLinks +includes +IncludesNOEXEC -SymLinksIfOwnerMatch
	AllowOverride AuthConfig Indexes Limit FileInfo Options=IncludesNOEXEC,Indexes,Includes,MultiViews
</Directory>

<Directory "/usr/local/apache/htdocs">
	Options +Includes -Indexes -FollowSymLinks -SymLinksIfOwnerMatch
	AllowOverride None
	Order allow,deny
	Allow from all
</Directory>

RemoveHandler .cgi .pl .py .pyc .pyo .plx .ppl .perl
```

Close the file

#### 2. Include HL.conf in apache

```Shell
nano /usr/local/apache/conf/httpd.conf
```

and put the following at end of file

```ApacheConf
Include "/usr/local/apache/conf/HL.conf"
```

- **Note:** Ensure inclusion of HL.conf on each Apache update.

### Setup Server Backup

Configure Backup as per Policy

- Incremential daily, retain 2
- Weekly, retain 2
- monthly, retain 3
- Off-site monthly, retain 2

### Stop unnecessary services

```Shell
service cups stop
chkconfig cups off
service nfslock stop
chkconfig nfslock off
service rpcidmapd stop
chkconfig rpcidmapd off
service anacron stop
chkconfig anacron off
service xfs stop
chkconfig xfs off
service bluetooth stop
chkconfig bluetooth off
service avahi-daemon stop
chkconfig avahi-daemon off
service hidd stop
chkconfig hidd off
service pcscd stop
chkconfig pcscd off
```

### Update everything and restart

```Shell
/scripts/upcp
/scripts/fixeverything
```

### Install Monitoring Tools

#### Side Comment

- Don't Forget ioncube_encoder5_7.0.tar.gz

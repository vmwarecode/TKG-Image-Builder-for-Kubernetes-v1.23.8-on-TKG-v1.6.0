# ubuntu_20.04_CIS

Cookbook to automate CIS implementation for Ubuntu 20.04

## Cookbook variables:

Comnpensating controls may exist that satisfy the STIG requirement for a security measure.  
Variables definded in [vars.yml](vars/main.yml) allow skipping package insatallation where compensating controls exist.

|Name|Default|Description|
|----|:-------:|-----------|
|`install_aide`| `true`|`aide` is an open source host based file and directory integrity checker. `install_aide` can be set to `no` if any other integrity checker (e.g. Tripwire) is installed on the VM instead of being baked into the image|
|`install_clamav`| `false`|`clamav` is an open source antivirus software. `install_clamav` can be set to `no` if any other antivirus softwarer is installed on the VM instead of being baked into the image|
|`clamav_database_mirrors`| `[]`|Array of private clamav mirrors in order to suppoer air-gapped environments. If empty, clamav will download signatures from http://database.clamav.net. Requires `install_clamav` to be set to `true`  |
|`install_chrony`| `true`| `chrony` provides fast and accurate time synchronization. AMI uses `timesyncd` for time synchronization.`install_chrony` can be set to `true` to use `chrony` instead of `timesyncd`. **[WARNING]** Setting this to `false` will fail image-builder goss test.|
|`install_protect_kernel_defaults` | `true` | set kernel setting to enable `--protect-kernel-defaults` on `kubelet` |
|`syslog_host`| `""`|FQDN or IP of remote loghost. Both `syslog_host` and `syslog_port` should be set for syslog forwarding to be configured|
|`syslog_port`| `""`|port of remote loghost. Both `syslog_host` and `syslog_port` should be set for syslog forwarding to be configured|


## Cloud provider specific tasks:
This repo has been tested on AWS only. 

## Exceptions

|	Rule ID	|	Title	|	Impact	|	Exception	|
|---------|-------|---------|-----------|
|1.1.1.6| Ensure mounting of squashfs filesystems is disabled| high | Disabling squashfs will prevent the use of snap. Snap is a package manager for Linux for installing Snap packages. |
|	1.1.10	|	Ensure separate partition exists for /var	|	high	|	This setting may cause issues cloud-init filesystem resizing.	|
|	1.1.15	|	Ensure separate partition exists for /var/log	|	high	|	Audit files are offloaded to external log sink. max_log_file_action is set to rotate to prevent the disk space filling up	|
|	1.1.16	|	Ensure separate partition exists for /var/log/audit	|	high	|	Log are offloaded to external log sink. logrotate is configured to prevent the disk space filling up	|
|	1.1.17	|	Ensure separate partition exists for /home	|	high	|	Not applicable: system is not intended to support local users	|
|	1.1.18	|	Ensure /home partition includes the nodev option	|	moderate	|	Not applicable: system is not intended to support local users	|
|	1.4.4	|	Ensure authentication required for single user mode	|	moderate	|	User ubuntu uses certificate based authentication 	|
|	1.6.1.4	|	Ensure all AppArmor Profiles are enforcing	|	high	|	snap.amazon-ssm-agent.amazon-ssm-agent profile is a known issue. https://github.com/aws/amazon-ssm-agent/issues/108	|
|	2.1.1.2	|	Ensure systemd-timesyncd is configured	|	none	|	Not applicable: chrony is used for time synchronisation	|
|	2.1.1.4	|	Ensure ntp is configured	|	none	|	Not applicable: chrony is used for time synchronisation	|
|	2.3	|	Ensure nonessential services are removed or masked	|	moderate	|	Manual verification	|
|	3.2.2	|	Ensure IP forwarding is disabled	|	moderate	|	This setting will break for container networking	|
|	3.5.1.1	|	Ensure ufw is installed	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.1.2	|	Ensure iptables-persistent is not installed with ufw	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.1.3	|	Ensure ufw service is enabled	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.1.4	|	Ensure ufw loopback traffic is configured	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.1.5	|	Ensure ufw outbound connections are configured	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.1.6	|	Ensure ufw firewall rules exist for all open ports	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.1.7	|	Ensure ufw default deny firewall policy	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.2.1	|	Ensure nftables is installed	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.2.10	|	Ensure nftables rules are permanent	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.2.2	|	Ensure ufw is uninstalled or disabled with nftables	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.2.3	|	Ensure iptables are flushed with nftables	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.2.4	|	Ensure a nftables table exists	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.2.5	|	Ensure nftables base chains exist	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.2.6	|	Ensure nftables loopback traffic is configured	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.2.7	|	Ensure nftables outbound and established connections are configured	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.2.8	|	Ensure nftables default deny firewall policy	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.2.9	|	Ensure nftables service is enabled	|	none	|	Not applicable: iptables is used for host based firewall	|
|	3.5.3.2.1	|	Ensure iptables loopback traffic is configured	|	moderate	|	iptables are managed by kubernetes	|
|	3.5.3.2.2	|	Ensure iptables outbound and established connections are configured	|	moderate	|	iptables are managed by kubernetes	|
|	3.5.3.2.3	|	Ensure iptables default deny firewall policy	|	moderate	|	iptables are managed by kubernetes	|
|	3.5.3.2.4	|	Ensure iptables firewall rules exist for all open ports	|	moderate	|	iptables are managed by kubernetes	|
|	3.5.3.3.1	|	Ensure ip6tables loopback traffic is configured	|	moderate	|	iptables are managed by kubernetes	|
|	3.5.3.3.2	|	Ensure ip6tables outbound and established connections are configured	|	moderate	|	iptables are managed by kubernetes	|
|	3.5.3.3.3	|	Ensure ip6tables default deny firewall policy	|	moderate	|	iptables are managed by kubernetes	|
|	3.5.3.3.4	|	Ensure ip6tables firewall rules exist for all open ports	|	moderate	|	iptables are managed by kubernetes	|
|	4.1.2.2	|	Ensure audit logs are not automatically deleted	|	high	|	Audit files are offloaded to external log sink. TKG favours availability to halting.	|
|	4.1.2.3	|	Ensure system is disabled when audit logs are full	|	high	|	Audit files are offloaded to external log sink. TKG favours availability to halting.	|
|	4.2.1.3	|	Ensure logging is configured	|	moderate	|	Manual verification	|
|	4.2.1.5	|	Ensure rsyslog is configured to send logs to a remote log host	|	moderate	|	Manual verification	|
|	4.3	|	Ensure logrotate is configured	|	moderate	|	Manual verification	|
|	6.1.1	|	Audit system file permissions	|	high	|	Manual verification	|

# Bastille BSD 

## Initial configuration

### Create network interface

Create lo1 interface  as root

    sysrc cloned_interfaces+=lo1
    sysrc ifconfig_lo1_name="bastille0"
    /etc/rc.d/netif cloneup
    /etc/rc.d/netif/restart

Use network 10.0.0.0/8 for jails

### Configure firewall

Enable pf service

    sysrc pf_enable="YES"

Create firewall rule in /etc/pf.conf

	# ext_if is my host system interface
	ext_if="re0"
	 
	set block-policy return
	scrub in on $ext_if all fragment reassemble
	set skip on lo
	 
	table <jails> persist
	nat on $ext_if from <jails> to any -> ($ext_if:0)
	rdr-anchor "rdr/*"
     
	block in all
	pass out quick keep state
	antispoof for $ext_if inet
	# allow remote SSH (not needed for desktop station)
	#pass in inet proto tcp from any to any port ssh flags S/SA modulate state

Start  pf

    /etc/rc.d/pf start
    /etc/rc.d/pf status

## Create first jail

Bootstrap current freebsd version

    sudo bastille bootstrap 12.2-RELEASE

Create Python jail

    sudo bastille create stacosys 12.2-RELEASE 10.25.71.1 bastille0
    sudo bastille pkg stacosys install python39


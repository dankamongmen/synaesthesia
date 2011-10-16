.DELETE_ON_ERROR:

.PHONY: default networks swinder winxp debsnare ironnet ironnetlocal smtpserver clean mrproper killdhcp killtap killsmtpkvm killwinxpkvm killfenetkvm killswinderkvm debnetboot

SHELL=/bin/sh
.SUFFIXES:

MAKEFILE:=GNUmakefile

default: swinder debsnare

OUT:=.out
SSHOUT:=$(OUT)/ssh
DHCPOUT:=$(OUT)/dhcp
QEMUOUT:=$(OUT)/qemu

XMLBIN:=$(shell which xmlstarlet 2> /dev/null || which xml 2> /dev/null)
MD5:=md5sum -c -

CONFDIR:=conf

# Debian GNU/Linux "Lenny"
DEBMODS:=Debian-mods/lenny

UPSTREAM:=upstream
ISOOUT:=$(UPSTREAM)/iso
MD5SUMS:=$(UPSTREAM)/md5sums

# Debain GNU/Linux "Lenny" (direct install)
DEBOUT:=$(UPSTREAM)/debootstrap
DEBIANTARBALL:=$(DEBOUT)/debian-226.tgz

# Debain GNU/Linux "Unstable" (Sid) (ISO)
DEBCDMODS:=Debian-mods/sid
DEBOMATE:=scripts/automate-debsid-iso-fuse
DEBAUTOISO:=$(OUT)/debautoinstall.iso
DEBISO:=$(ISOOUT)/debian-testing-amd64-businesscard.iso

# SCUR images created by mahatma (IronNet for now)
FENETVER:=1.2.0-pre15-r2937-c2823
FENETISO:=$(ISOOUT)/ironnet-$(FENETVER).iso

# Windows XP Professional
SIDEWINDERISO:=$(ISOOUT)/Sidewinder70.iso

# RedHat Enterprise Linux Server Edition (i386) upgrade 9
RHEL3VER:=3-u9-i386-es
RHOUT:=$(UPSTREAM)/$(RHEL3VER)
RHEL3ISO:=$(addsuffix .iso,$(addprefix $(ISOOUT)/rhel-$(RHEL3VER)-disc,1 2 3 4))
AUTOMATE:=scripts/automate-rhel3-iso-fuse
AUTOMATECONF:=$(addprefix RedHat-mods/$(RHEL3VER)/,ks.cfg images/bootdisk.img/syslinux.cfg .discinfo)

# The pristine installation ISO's must be modified to allow a fully automatic
# installation. The actual installation step boots such a modified ISO.
AUTOISO:=$(OUT)/autoinstall.iso

# Additional software must be installed into the images.
RPMS:=$(addprefix $(RHOUT)/,$(addsuffix .rpm,subversion-1.4.4-1.i386 neon-0.24.7-1.i386 xmlstarlet-1.0.1-1.i586 httpd-2.0.46-61.1.ent.i386))

IMGSIZE:=80G

SMTPIMG:=$(QEMUOUT)/smtpserver.img
FENETIMG:=$(QEMUOUT)/fenet.img
SIDEWINDERIMG:=$(QEMUOUT)/swinder.img

SMTPHOSTKEY:=$(SSHOUT)/smtp_host_rsa
SMTPHOSTPUB:=$(SMTPHOSTKEY).pub

DEBPID:=$(QEMUOUT)/debsnare.pid
SMTPQEMUPID:=$(QEMUOUT)/smtp.pid
FENETPID:=$(QEMUOUT)/fenet.pid
SIDEWINDERPID:=$(QEMUOUT)/swinder.pid

DEBLINK:=$(QEMUOUT)/debsnare-installed.img
DEBIMG:=$(QEMUOUT)/debsnare.img

# Script executed by qemu when the associated network interface is initialized
# (by default /etc/qemu-if.up). The keyword "no" disables this feature.
TAPSCRIPT:=no
TAPDSCRIPT:=no

QEMU:=kvm
QIMG:=kvm-img

include $(MAKEFILE).network
include $(MAKEFILE).netboot
include $(MAKEFILE).winxp

DEBOPS:=-pidfile $(DEBPID) -no-reboot -m 1024 -daemonize

FENETTCP:=10026
FENETOPS:=-pidfile $(FENETPID) -no-reboot -m 1024 -daemonize
FENETOPS+=-net tap,ifname=$(FENETTAPDEV),script=$(TAPSCRIPT),downscript=$(TAPDSCRIPT),vlan=1
FENETOPS+=-net nic,model=ne2k_pci,macaddr=$(FENETMAC),vlan=1
FENETOPS+=-serial tcp:$(DMZIP):$(FENETTCP),server,nowait
FENETVNCOPS:=-vnc :4 -monitor null

SMTPTCP:=10025
SMTPKERNEL:=$(DEBOUT)/bzImage-2.6.22.1.i386
SMTPQEMUOPS:=-pidfile $(SMTPQEMUPID) -no-reboot --no-kvm
SMTPQEMUOPS+=-net tap,ifname=$(SMTPTAPDEV),script=$(TAPSCRIPT),downscript=$(TAPDSCRIPT)
SMTPQEMUOPS+=-net nic,model=rtl8139,macaddr=$(SMTPMAC)
SMTPQEMUOPS+=-vnc :1 -monitor null -serial tcp:$(DMZIP):$(SMTPTCP),server,nowait
SMTPQEMUOPS+=-daemonize -kernel $(SMTPKERNEL) -append 'root=/dev/hda ro console=ttyS0,57600'

SIDEWINDERTCP:=10027
SIDEWINDERQEMUOPS:=-pidfile $(SIDEWINDERPID) -no-reboot -m 1024
SIDEWINDERQEMUOPS+=-net tap,ifname=$(SIDEWINDERTAPDEV),script=$(TAPSCRIPT),downscript=$(TAPDSCRIPT),vlan=1
SIDEWINDERQEMUOPS+=-net nic,model=e1000,macaddr=$(SIDEWINDERMAC),vlan=1
SIDEWINDERQEMUOPS+=-net tap,ifname=$(SIDEWINDERTAPDEVB),script=$(TAPSCRIPT),downscript=$(TAPDSCRIPT),vlan=2
SIDEWINDERQEMUOPS+=-net nic,model=e1000,macaddr=$(SIDEWINDERMACB),vlan=2
SIDEWINDERQEMUOPS+=-serial tcp:$(DMZIP):$(SIDEWINDERTCP),server,nowait
SIDEWINDERQEMUVNCOPS:=-vnc :5 -monitor null

ADDUSER:=scripts/adduser
SSHIMAGE:=scripts/sshimage
SCPIMAGE:=scripts/scpimage
SSHCONF:=$(CONFDIR)/ssh_config
SSHKNOWNHOSTS:=$(SSHOUT)/known_hosts

COMMONSCRIPTS:=$(ADDUSER) $(SSHIMAGE) $(SCPIMAGE)

smtpserver: $(SMTPQEMUPID)
swinder: $(SIDEWINDERPID)

ironnet: $(FENETPID)

$(FENETPID): $(INNERDHCPPID) $(FENETIMG)
	@[ -r $@ ] || { echo "[Booting IronNet $(FENETVER)]" && \
	 $(QEMU) -hda $(FENETIMG) $(FENETOPS) $(FENETVNCOPS) ; }

ironnetlocal: $(INNERDHCPPID) $(FENETIMG)
	[ -r $(FENETPID) ] || { echo "[Booting IronNet $(FENETVER)]" && \
	 $(QEMU) -hda $(FENETIMG) $(FENETOPS) ; }

$(SIDEWINDERPID): $(SIDEWINDERIMG)
	@echo "[Booting Sidewinder 7.0]"
	$(QEMU) -hda $< -daemonize $(SIDEWINDERQEMUOPS) $(SIDEWINDERQEMUVNCOPS)

$(SIDEWINDERIMG): $(DHCPPID) $(INNERDHCPPID) $(SIDEWINDERISO)
	@[ -d $(dir $(SIDEWINDERPID)) ] || mkdir -p $(dir $(SIDEWINDERPID))
	@echo "[Preparing Sidewinder 7.0]"
	$(QIMG) create -f qcow2 $@ $(IMGSIZE)
	@echo "[Booting Sidewinder 7.0]"
	$(QEMU) -hda $@ -cdrom $(SIDEWINDERISO) $(SIDEWINDERQEMUOPS) -boot d

$(SMTPQEMUPID): $(INNERDHCPPID) $(SMTPIMG)
	@[ -r $@ ] || { echo "[Booting SMTP Server]" && \
	 $(QEMU) -hda $(SMTPIMG) $(SMTPQEMUOPS) && \
	 echo "[Waiting for connection from VM, please be patient]" && \
	 ( echo -e "HTTP/1.1 200 Ok\r\n\r\n\c" && cat $(SMTPHOSTPUB) ) | socat - tcp-listen:10080,bind=$(DMZIP),reuseaddr ; }
	@echo "You can now use $(SSHIMAGE) $(SMTPIP) to access the device."
	@echo "Access the serial port via socat - tcp:$(DMZIP):$(SMTPTCP). VNC is available."

debsnare: $(DEBLINK)
	[ -r $(DEBPID) ] || { echo "[Booting Debian Unstable]" && \
	 $(QEMU) -hda $< -boot c $(DEBOPS) ; }
	
$(DEBLINK): $(DEBIMG)
	@ln -sf $(<F) $@

$(DEBIMG): $(DEBAUTOISO) $(INNERDHCPPID) $(DHCPPID)
	@[ -d $(@D) ] || mkdir -p $(@D)
	@echo "[Preparing Debian GNU/Linux Unstable]"
	$(QIMG) create -f qcow2 $@ $(IMGSIZE)
	@echo "[Booting Linux and generating host key]"
	$(QEMU) -hda $@ -boot d -cdrom $< $(DEBOPS)

SMTPIMGGIGS:=2
DEBIANIZER:=scripts/makedebian
KEYRING:=$(CONFDIR)/trusted.gpg
$(SMTPIMG): $(SMTPHOSTKEY) $(SMTPHOSTPUB) $(KEYRING) $(DEBIANTARBALL) $(SMTPKERNEL) $(COMMONSCRIPTS)
	@[ -d $(@D) ] || mkdir -p $(@D)
	@[ -d $(dir $(SMTPQEMUPID)) ] || mkdir -p $(dir $(SMTPQEMUPID))
	@echo "[Preparing Debian GNU/Linux]"
	@$(DEBIANIZER) $@ $(KEYRING) $(SMTPIMGGIGS) $(DEBMODS) $(DEBIANTARBALL)
	@[ ! -r $(SSHKNOWNHOSTS) ] || ssh-keygen -f $(SSHKNOWNHOSTS) -R $(SMTPIP)
	@echo "[Booting Linux and generating host key]"
	$(QEMU) -hda $@ -boot c $(SMTPQEMUOPS)
	@echo "[Waiting for connection from VM, please be patient]"
	@( echo -e "HTTP/1.1 200 Ok\r\n\r\n\c" && cat $(SMTPHOSTPUB) ) | socat - tcp-listen:10080,bind=$(DMZIP),reuseaddr
	@echo "[Checking post-installation log]"
	$(SSHIMAGE) -n -oStrictHostKeyChecking=no $(SMTPIP) cat /root/postinstall.log
	@echo "[Setting default locale to UTF-8]"
	echo -e "129\n2" | $(SSHIMAGE) $(SMTPIP) dpkg-reconfigure locales
	@echo "[Installing $$USER as root mailalias]"
	$(ADDUSER) $(SMTPIP) $$USER sudo
	echo "root: $$USER" | $(SSHIMAGE) $(SMTPIP) tee /etc/aliases
	$(SSHIMAGE) -n $(SMTPIP) newaliases
	@echo "[Configuring mail server]"
	echo -e "2\n$$USER\n$(SMTPHOSTNAME).$(INNERDOMAIN)\n$(SMTPHOSTNAME).$(INNERDOMAIN),$(INNERDOMAIN),$(SMTPHOSTNAME),localhost,localhost.localdomain\nno\n\nyes\n0\n+\n1\n" | $(SSHIMAGE) $(SMTPIP) dpkg-reconfigure postfix
	@echo "[Installing extra packages via apt]"
	$(SSHIMAGE) -n $(SMTPIP) aptitude install --assume-yes mailx realpath mutt dnsutils snmpd
	$(SSHIMAGE) -n $(SMTPIP) shutdown -h now
	@echo "[Waiting for Debian GNU/Linux shutdown]"
	@socat - tcp:$(DMZIP):$(SMTPTCP) 2> /dev/null || true

$(FENETIMG): $(FENETISO) $(SSHIMAGE) $(INNERDHCPPID) $(DHCPPID)
	@[ -d $(@D) ] || mkdir -p $(@D)
	@[ -d $(dir $(FENETPID)) ] || mkdir -p $(dir $(FENETPID))
	@echo "[Preparing IronNet $(FENETVER)]"
	$(QIMG) create -f qcow2 $@ $(IMGSIZE)
	@echo "[Installing IronNet $(FENETVER)]"
	$(QEMU) -cdrom $(FENETISO) -hda $@ -boot d $(FENETOPS) $(FENETVNCOPS)
	@socat - tcp:$(DMZIP):$(FENETTCP)

$(AUTOISO): $(RHEL3ISO) $(AUTOMATE) $(AUTOMATECONF) $(MD5SUMS)
	@[ -d $(@D) ] || mkdir -p $(@D)
	@echo "[Preparing RedHat Enterprise Linux Installer]"
	@$(AUTOMATE) $(RHEL3ISO) $@ $(AUTOMATECONF)

$(DEBAUTOISO): $(DEBISO) $(DEBOMATE) $(DEBCDMODS)
	@[ -d $(@D) ] || mkdir -p $(@D)
	@echo "[Preparing Debian GNU/Linux Unstable Installer]"
	@$(DEBOMATE) $(DEBISO) $@ $(DEBCDMODS)

$(SMTPHOSTKEY):
	@[ -d $(@D) ] || mkdir -p $(@D)
	@ssh-keygen -t rsa -N "" -C "synaesthesia key: $@" -f $@

$(DEBISO):
	@[ -d $(@D) ] || mkdir -p $(@D)
	@wget http://cdimage.debian.org/cdimage/daily-builds/daily/arch-latest/amd64/iso-cd/$(@F) -O $@

$(SIDEWINDERISO) $(WINXPISO) $(SMTPKERNEL) $(DEBIANTARBALL) $(RPMS) $(RHEL3ISO):
	@[ -d $(@D) ] || mkdir -p $(@D)
	@wget http://svn.research.sys/synaesthesia/trunk/$(@F) -O $@
	@echo "[Checking MD5 sum for $(@F)]"
ifeq ($(shell uname),Linux)
	@grep $@ $(MD5SUMS) | $(MD5)
else
	@grep "`md5 -q $@` \*$@" $(MD5SUMS)
endif

$(FENETISO):
	@[ -d $(@D) ] || mkdir -p $(@D)
	@wget http://svn.research.sys/mahatma/$(@F) -O $@
	@echo "[Checking MD5 sum for $(@F)]"
ifeq ($(shell uname),Linux)
	@grep $@ $(MD5SUMS) | $(MD5)
else
	@grep "`md5 -q $@` \*$@" $(MD5SUMS)
endif

# Has its reciprocal in the "killtap" target
networks: $(DHCPPID) $(INNERDHCPPID)

# Doesn't remove .iso files
clean: killtap
	@svn --xml --no-ignore status | $(XMLBIN) sel -t -m //entry -i "wc-status[@item='ignored']" -v @path -n | grep -v -E upstream/ | xargs rm -rvf

mrproper: killtap
	@svn --xml --no-ignore status | $(XMLBIN) sel -t -m //entry -i "wc-status[@item='ignored']" -v @path -n | xargs rm -rvf

CC=gcc
CFLAGS=-Wall
TARGETS=bootparams disklist removable_disk disk_size \
    node-kstat/build/Release/kstat.node zoneevent \
    node-expat/build/Release/node-expat.node \
    node-syslog/build/Release/syslog.node
SMARTDC_TARGETS=has_hvx
DESTDIR=../proto
NODE_WAF=$(PWD)/../proto/usr/bin/node-waf

all: $(TARGETS) sysinfo $(SMARTDC_TARGETS)

install: $(TARGETS) sysinfo
	mkdir -p $(DESTDIR)/usr/bin
	cp -p $(TARGETS) sysinfo $(DESTDIR)/usr/bin
	mkdir -p $(DESTDIR)/smartdc/bin
	cp -p $(SMARTDC_TARGETS) $(DESTDIR)/smartdc/bin
	mkdir -m 0755 -p $(DESTDIR)/usr/node_modules
	find node_modules -type f -exec cp {} $(DESTDIR)/usr/node_modules/ \;
	cp node-kstat/build/Release/kstat.node $(DESTDIR)/usr/node_modules/
	cp node-syslog/build/Release/syslog.node $(DESTDIR)/usr/node_modules/syslog.node
	rm -rf $(DESTDIR)/usr/vm
	cp -PR vm $(DESTDIR)/usr/
	mkdir -m 0755 -p $(DESTDIR)/usr/vm/node_modules
	cp node-expat/build/Release/node-expat.node \
        $(DESTDIR)/usr/vm/node_modules/expat_binding.node
	cat node-expat/lib/node-expat.js | \
        sed -e "s|var expat.*;|var expat = require('expat_binding');|" > \
        $(DESTDIR)/usr/vm/node_modules/node-expat.js
	cp zoneevent $(DESTDIR)/usr/vm/sbin/zoneevent
	mkdir -p $(DESTDIR)/lib/svc/manifest/system
	cp vm/smf/system-vmadmd.xml $(DESTDIR)/lib/svc/manifest/system/system-vmadmd.xml
	mv $(DESTDIR)/usr/vm/sbin/vmadmd.js $(DESTDIR)/usr/vm/sbin/vmadmd
	mv $(DESTDIR)/usr/vm/sbin/vmadm.js $(DESTDIR)/usr/vm/sbin/vmadm
	mkdir -m 0755 -p $(DESTDIR)/var/db/dsadm
	cp -PR ds $(DESTDIR)/usr/

check:
	@tools/node-lint/bin/node-lint *.js \
        --config=tools/node-lint.json

bootparams: bootparams.c
	$(CC) $(CFLAGS) -o $@ $^ -ldevinfo

removable_disk: removable_disk.c
	$(CC) $(CFLAGS) -o $@ $^

disk_size: disk_size.c
	$(CC) $(CFLAGS) -o $@ $^

has_hvx: has_hvx.c
	$(CC) $(CFLAGS) -o $@ $^

node-kstat/build/Release/kstat.node: node-kstat/kstat.cc $(NODE_WAF)
	(cd node-kstat && $(NODE_WAF) configure && $(NODE_WAF) build)

node-expat/build/Release/node-expat.node: node-expat/node-expat.cc $(NODE_WAF)
	(cd node-expat && $(NODE_WAF) configure && $(NODE_WAF) build)

node-syslog/build/Release/syslog.node: node-syslog/syslog.cc $(NODE_WAF)
	(cd node-syslog && $(NODE_WAF) configure && $(NODE_WAF) build)

disklist:
	cp disklist.sh disklist
	chmod 0755 disklist

sysinfo:
	touch sysinfo

zoneevent: zoneevent.c
	gcc -Wall -W -o zoneevent zoneevent.c -lnvpair -lsysevent

clean:
	@rm -f $(TARGETS)
	(cd node-kstat && $(NODE_WAF) clean)
	(cd node-expat && $(NODE_WAF) clean)
	(cd node-syslog && $(NODE_WAF) clean)

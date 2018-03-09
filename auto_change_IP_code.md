#!/bin/bash
# This script test sending to hotmail and Change the IP if it's blocked
# and if sending fails, it changes
# the interface IP of exim randomly
# and sends email report
# v.0.3.0 Apr-2010
#
PATH=/usr/local/jdk/bin:/usr/kerberos/sbin:/usr/kerberos/bin:/usr/lib/courier-imap/sbin:/usr/lib/courier-imap/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin:/usr/X11R6/bin:/root/bin
function changeinterfaceip {
IFACE=$(awk '/ETHDEV/ {print $2}' /etc/wwwacct.conf)
IPs=($(ifconfig | grep -A1 "$IFACE"":[2-9]" | awk 'BEGIN {FS=":"}/inet addr:/{split($0,a,":");split(a[2],b," "); print b[1]}'))
IPnum=$(($RANDOM%$[${#IPs[@]}-1]))
if grep ' *interface *=' /etc/exim.conf | egrep -qo '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' ; then
IPs=($(echo "${IPs[@]}"| tr ' ' '\n' | egrep -v "$(grep ' *interface *=' /etc/exim.conf | egrep -o '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+'|  head -n 1)" ))
NEWIP=${IPs[$IPnum]}
sed -i 's/^ *interface *=.*$/ interface = '$NEWIP'/' /etc/exim.conf
/etc/init.d/exim restart

else
NEWIP=${IPs[$IPnum]}
sed -i 's/^ *interface *=.*$/ interface = '$NEWIP'/' /etc/exim.conf
/etc/init.d/exim restart
fi
}

echo 'Hotmail test' | exim -v yourmail@hotmail.com 2>&1 | egrep -q 'SMTP<< (421|451|450|550)' && ( changeinterfaceip )

echo 'Yahoo test' | exim -v yourmail@yahoo.com 2>&1 | egrep -q 'SMTP<< (421|451|450|550)' && ( changeinterfaceip )

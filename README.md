Diladele Web Safety 4.5 in Docker with Squid 3.15.19
=============================================

This project is a fork of https://github.com/diladele/docker-websafety  Web Safety ICAP web filter running in a docker container.

Modifications
-------------

The fork makes the following modifications to run the latest Squid version

 - Moved to use phusion/baseimage 0.9.15 for the docker container - remains on Ubuntu 14.04
 - Moved to use runit rather than supervisor

The following remains unmodified

 - Diladele Websafety 4.5 - direct from Diladele website
 - Squid 3.5.19 - compiled for Ubuntu 14.04 by Diladele
 - SQLite only in this image

Important - currently /opt/qlproxy/bin is setuid to run as the correct user
This might mean changes to instructions such as these http://docs.diladele.com/administrator_guide_4_5/traffic_monitoring/database/create.html


Details / Running changes
-------
The run scripts for the diladele processes in /contents are simple scripts.  exec is used to make sure that runit can control the processes properly.
Squid has some additional controls to ensure that the diladele processes are started first by using the result of the sv status command:

        qlpstatus=`sv status qlproxy |cut -d ' ' -f 1`
        wsmgrstatus=`sv status wsmgr |cut -d ' ' -f 1`
        if [[ "$qlpstatus" != "run:" ]]
		    then exit 1
	    fi
	    if [[ "$wsmgrstatus" != "run:" ]]
		    then exit 1
	    fi

This is a refinement to the suggestion here http://smarden.org/runit/faq.html#depends

apache2 is similarly delayed to start after squid has started.

The four key processes are controllable via the GUI, or from the command line using the sv start/stop/restart commands.
Note, that in the diladele restart script, a pause is used for squid which can take a short time to shutdown

    echo "Reloading Squid Proxy Server..."
    sv -w 15 restart squid

The latest modifications use a config container as described here : http://container-solutions.com/understanding-volumes-docker/ 


Use
---
After cloning the project, build with

    ./build.sh

Run with

    ./run.sh

Stop with

    docker stop websafety-runtime

Start again with

    docker start websafety-runtime

TODO:
Some instructions for getting running with mysql in a separate container


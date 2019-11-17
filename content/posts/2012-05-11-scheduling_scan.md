+++
title = "Spacewalk & OpenSCAP :: Scheduling First Security Scan"
type = ["posts","post"]
tags = [
    "OpenSCAP",
    "Satellite",
]
date = "2012-05-11"
categories = [
    "OpenSCAP",
    "Satellite",
]
series = ["OpenSCAP 101"]
[ author ]
  name = "Šimon Lukašík"
+++

  ----

This example shows how to schedule OpenSCAP scan through Spacewalk web user interface.

**Prerequisites**: For this example we will need to setup server/client environment. We will use server to schedule SCAP XCCDF scan remotely on the client machine. As a client may serve any Fedora based distro (it could be e.g RHEL5), but it *could* work even with (Open)SUSE or Debian. In the [previous post](/posts/2012/05/a-first-scap-content/) we have prepared a very simple SCAP content. Let's start with copying it to the client.

    wget http://isimluk.com/blog-data/2012-05-first_scap/first_xccdf.xml \
         http://isimluk.com/blog-data/2012-05-first_scap/first_oval.xml
    scp first_xccdf.xml first_oval.xml root@eva.example.com:/usr/share/scap-content/


On the server side, there will be Spacewalk with OpenSCAP support. At this point, OpenSCAP support is not yet present in any of released versions of Spacewalk, it will go to the next (post 1.7) version. Thus, we need to [install Spacewalk nightly build](https://github.com/spacewalkproject/spacewalk/wiki/HowToInstall) (it may take a while). And finally, to register your client system with Spacewalk please consult [documentation](https://github.com/spacewalkproject/spacewalk/wiki/RegisteringClients).

Once registered, client will appear on the Spacewalk web. As you can see on the picture, Spacewalk offers a huge variaty of management features and I suggest you to explore them.

![Spacewalk Web Interface](/blog-pics/2012-schedule/blog-01-registered-client.jpg)

The SCAP functionality can be found under Audit sub-tab. As the page suggests, the client machine needs to have spacewalk-oscap package installed. This package is an adapter between the /usr/bin/oscap tool and the Spacewalk. The package is available in [Spacewalk-nightly client repo](http://spacewalk.redhat.com/yum/nightly-client/). You can install it with yum, or using Spacewalk webui (in that case you need to [upload the package](https://fedorahosted.org/spacewalk/wiki/UploadContent) to Spacewalk first).

![System Audit page](/blog-pics/2012-schedule/blog-02-audit-tab.jpg)

Additionally, in case you have installed the spacewalk-oscap package using yum, you need to notify Spacewalk server that the OpenSCAP capability is available on the client. You can do it by running the /usr/bin/rhn_check command, or by waiting for rhnsd deamon to do it automatically. More info about the deamon you can find in downstream documentation: it could be either [rhnsd](http://docs.redhat.com/docs/en-US/Red_Hat_Network_Satellite/5.4/html/Reference_Guide/chap-Reference_Guide-RHND.html) or [osad](http://docs.redhat.com/docs/en-US/Red_Hat_Network_Satellite/5.4/html/Installation_Guide/sect-Installation_Guide-Maintenance-Enabling_Push_to_Clients.html).

Now, everything is plugged in and we can schedule the OpenSCAP scan. Since we have a very simple XCCDF file (without profiles), we only to need to specify path it.

![Scheduling the scan](/blog-pics/2012-schedule/blog-03-schedule.jpg)

After hitting the schedule button. the OpenSCAP action is pending and waits for client's pick-up. If you have osad daemon running, it is executed immediatelly, if you have rhnsd deamon, it is executed within next 4 hours. Alternativally, you can run it manually by rhn_checkk command.

![Scan is pending](/blog-pics/2012-schedule/blog-04-pending.jpg)

Wondering what happens underneath? Deamon runs the rhn_check. The rhn_check reads instructions from Spacewalk and issues OpenSCAP to evaluate the given document and to store the results in a secured tempfile. Then rhn_check cherry-picks a brief resume from the results and sends it back to Spacewalk as a response. After the resume is stored Spacewalk database, user can inspect it and take actions.

![Detailed results](/blog-pics/2012-schedule/blog-05-detailed_results.jpg)

As we can see, the client machine passed the scan. The only rule in our document no_hashes_outside_shadow is pass. Meaning, all the password hashes are safe in /etc/shadow. Yay!

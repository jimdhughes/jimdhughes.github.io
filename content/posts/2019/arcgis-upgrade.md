+++
author = "James D Hughes"
categories = ["ArcGIS"]
date = 2019-06-15T00:45:00Z
description = ""
draft = false
slug = "arcgis-upgrade"
tags = ["ArcGIS"]
title = "ArcGIS Upgrade Error 28809"

+++


I was upgrading ArcGIS Server (10.6.1 -> 10.7) the other day...

We encountered an issue with the account ArcGIS was using to run itself - the LocalSystem account. We don't have passwords for it, so the installer would not let us proceed. We couldn't change the user either which posed a bit of an issue so we decided to just cancel our install and proceed along our merry way.

So we turned our ArcGIS Server service back on and gave it a few minutes to spin back up however it never did! We hunted through our logs and didn't see much to indicate what might have happened.

Windows logs gave some odd messages about ArcGIS' inability to start up some files and suggested that our installation might be corrupt. That seemed logical since we had a failed upgrade attempt.  I went ahead and re-installed v 10.6.1 and waited patiently for it to finish and for the service to start back up.  Much to my dismay, we were still dead in the water.  I scrounged around to for a while and found a blog post at: [https://gdbgeek.wordpress.com/2015/10/12/arcgis-for-server-wont-start/](https://gdbgeek.wordpress.com/2015/10/12/arcgis-for-server-wont-start/). This pointed me in the right direction to my log files.  I dug  into the logs located at `C:\Program Files\ArcGIS\Server\framework\etc\service\logs` which shed some light on the situation.

Deep down in the depths of the log was a line that read a little like this:

`$ Found flag file D:\Program Files\ArcGIS\Server\framework\etc\ArcGISServerUG.txt. Starting in upgrade mode.`

Since the installer didn't actually start moving, I figured that it would have either not modified my filesystem or would have at least cleaned itself up. This does not seem to have been the case though!

To get ArcGIS Server running again, simply rename or delete that file (it's an empty .txt file). From there you should be good to go! At least we were. Our 10.6.1 environment is back up and running while we dig into how to get around our upgrade issues.

Hopefully this helps someone else! Was a bit of a head scratcher for me.






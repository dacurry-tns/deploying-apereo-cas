---
title: Problems encountered
last_updated: October 18, 2018
sidebar: main_sidebar
permalink: prod_problems-encountered.html
summary:
---

For the most part, CAS 5 in production has worked quite well. However, as we moved through the end of the Spring 2018 semester, through the summer, and then the start of the Fall semester, we did encounter a few problems that had to be addressed.

## Incorrect MongoDB connection pool size

At the beginning of the fall semester, we began to receive support desk calls from users who had successfully authenticated to CAS, but upon being redirected back to the calling service, were receiving an error that the service could not locate the service ticket. We had seen this sporadically before, and simply refreshing the browser was usually enough to cure the problem, but it was becoming a larger problem. We pretty quickly determined that the problem had something to do with tickets being requested from the MongoDB ticket registry before they had been written, but figuring out what was causing that to happen was much more difficult.

Eventually, we found the problem. It turns out that the CAS code that makes calls to the MongoDB Java driver has not kept up with driver developments on the MongoDB side; it is configuring the driver with deprecated property settings. This ultimately results in the driver being configured with a connection pool that is an order of magnitude too small&mdash;50 connections instead of 500 (the default, if non-deprecated property settings are used). The increased load on our servers as a result of the fall semester's start, combined with virtual server sizes that were barely adequate (see below) was just enough to cause the CAS servers to occasionally deplete the connection pool.

The solution to the problem was to add parameters to the MongoDB connection string to explicitly set the size of the connection pool back to the defaults. This was done by changing the value of the `mongo.opts` "pseudo-property" in `cas.properties` and `cas-management.properties` from

```properties
mongo.opts:                             &ssl=true
```

to

```properties
mongo.opts:                             &ssl=true&maxPoolSize=100&waitQueueMultiple=5
```

Since making this change (and increasing the size of the servers, see below) we have not seen any more occurrences of this problem.

## Limit MongoDB cache size

As one step in investigating the problem described above, we took a look at the MongoDB internal cache. By default, MongoDB will set the size of this cache to the larger of
* 50% of (RAM - 1GB), or
* 256MB

On our 8GB servers, this results in a cache size of 3.5GB ((8GB - 1GB) &times; 0.5). As it turns out, for our environment, this size performs pretty well, and it didn't make any sense to us to increase it. However, because we had separately made the decision to increase the CAS servers to 12GB of memory (see below), we realized that we needed a way to limit the size of MongoDB's cache, or it would start using 5.5GB ((12GB - 1GB) &times; 0.5) on the new servers.

To limit the size of MongoDB's cache, edit the `/etc/mongodb.conf` configuration file on the master build server for the environment and locate the `storage` section (around line 13) and change it from:

```yaml
storage:
  dbPath: /var/lib/mongo
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:
```

to

```yaml
storage:
  dbPath: /var/lib/mongo
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
      cacheSizeGB: 4
#  engine:
#  mmapv1:
```

This will limit the cache size to 4GB regardless of how much memory is on the system. Copy `/etc/mongod.conf` to all of the CAS servers in the environment and restart MongoDB.

## Server sizes in production

When we first went into production, the virtual servers were configured with 2 CPUs and 8GB of memory. This worked well until we entered the start of the Fall semester (one of our heaviest load periods), when we began to notice the servers slowing down because they were starved for resources. In part this was caused by the two MongoDB configuration issues described above, but it was also because things were running right on the edge already, and the additional load from the start of the semester was enough to push them over.

We have since decided to increase the size of the virtual servers to 4 CPUs and 12GB of memory. We have kept the Java heap size limited to 4GB (which is plenty), and have now also limited the MongoDB cache size to 4GB (see above). This results in two-thirds of the available memory on the system being devoted to CAS, and the remaining one-third being left for the kernel, other processes, etc.

Since making these changes, the servers have been performing very well.

## Cleaning up log files

In our environment, we don't want log files to grow without bound, nor do we want to accumulate log files "forever." In fact, we have configured CAS to [rotate its log file(s) every day][building_server_configure-logging-settings], and we really don't need to keep more than the last 30 days' worth of logs on line (we can go back further via backups, if necessary, plus we have everything in Graylog). Unfortunately, convincing Log4j2 to do this appears to be the next best thing to impossible, and getting Tomcat's JULI-based logging to do it is even worse. After trying numerous configurations without any luck, we eventually gave up and created `/etc/cron.daily/caslogs`:

```sh
#!/bin/sh
#
# Clean up CAS log files in /var/log/tomcat and /var/log/cas. These files do
# not necessarily get created every day, so deleting by age might end up
# deleting the current file. Instead, we keep the last $NLOGS files of
# each type.
#

PATH=/bin:/usr/bin; export PATH

TMP=/tmp/caslog$$
NLOGS=30

trap "rm -f $TMP" 0

DATEGLOB='[0-9][0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9]'

if [ -d /var/log/tomcat ]
then
    cd /var/log/tomcat

    for f in catalina host-manager localhost localhost_access_log manager
    do
        ls -1 ${f}.${DATEGLOB}.* | head -n -${NLOGS} > ${TMP}

        if [ -s ${TMP} ]
        then
            rm -f `cat ${TMP}`
        fi
    done
fi

if [ -d /var/log/cas ]
then
    cd /var/log/cas

    for f in cas cas-management cas_audit perfStats
    do
        ls -1 ${f}-${DATEGLOB}.log | head -n -${NLOGS} > ${TMP}

        if [ -s ${TMP} ]
        then
            rm -f `cat ${TMP}`
        fi
    done
fi

exit 0
```

This script is executed every night at (approximately) midnight by `cron`. It will keep the `NLOGS` most recent log files of each type in `/var/log/cas` and `/var/log/tomcat`, and delete any beyond that number. It relies on all the log files having names of the general format

```
[basename].YYYY-MM-DD.*
```

It does not make any assumptions about how often the log file is rotated; it will always keep the most recent `NLOGS` files regardless of how old they are. Although we have not seen the need to yet, the script could easily be modified to compress the remaining log files with `gzip` or something else.

## Fixing a bug in Duo's WebSDK

Once we started rolling out Duo MFA, we discovered that some users&mdash;those who were using Internet Explorer&mdash;were having difficulties using it because the "Duo box" was not appearing after they entered their username and password. Instead, a blank page would appear and the login process would essentially be "stuck" at that point. To make things even more interesting, the problem only seemed to occur when the users were accessing a SAML2-based service (SP); services that authenticated via CAS and required Duo MFA did not exhibit the problem. Eventually, we determined that the problem was due to a bug in the Duo Web SDK (written by Duo Security) used by the CAS server.

### How to fix the problem

Fixing the problem requires making a change to `Duo-Web-v2.js` and then installing the corrected file in the CAS server. The change to be made is:

```diff
*** Duo-Web-v2.js       2018-06-28 08:12:08.723891501 -0400
--- Duo-Web-v2-fix.js   2018-06-28 08:14:41.721450104 -0400
***************
*** 374,380 ****
          // point the iframe at Duo
          iframe.src = [
              'https://', host, '/frame/web/v1/auth?tx=', duoSig,
!             '&parent=', encodeURIComponent(document.location.href),
              '&v=2.6'
          ].join('');

--- 374,383 ----
          // point the iframe at Duo
          iframe.src = [
              'https://', host, '/frame/web/v1/auth?tx=', duoSig,
!             '&parent=',
!           (window.postMessage ?
!               encodeURIComponent(document.location.href.split('?')[0]) :
!               encodeURIComponent(document.location.href)),
              '&v=2.6'
          ].join('');
```

To apply the patch and include it in the CAS server:

1. Download the original source from [GitHub](https://raw.githubusercontent.com/duosecurity/duo_java/master/js/Duo-Web-v2.js).
2. Either make the change above manually, or save it to a file and run `patch -p0 < patch.txt`.
3. Feed the patched file to your favorite JavaScript minimizer (e.g., [Uglify](http://lisperator.net/uglifyjs/)) and save the result.
4. Copy the new minimized file to the Maven overlay at `src/main/resources/static/js/duo/Duo-Web-v2.js` or, if you have configured your own theme, at `src/main/resources/static/themes/yourtheme/js/duo/Duo-Web-v2.js`, and rebuild and install the application.

### Explanation of the patch

The problem is that as the web flow goes back and forth between CAS and the SP, the query parameters that CAS puts on the end of the URL get longer and longer. This seems to happen with all CAS/SAML2 services, but it's much worse when one or both sides of the SAML2 negotiation require signed and/or encrypted assertions; the URL with query parameters can grow in length to tens of thousands of characters.

Eventually things get to the point in the webflow where CAS sends `casDuoLoginView.html` to the user's browser. This is the page that includes the Duo Web SDK, a bunch of JavaScript that manipulates the `iframe` on the page that displays the Duo dialog. To populate the Duo `iframe`, the Web SDK constructs a source URL that points at the Duo back-end servers and includes as a query parameter the full URL of the CAS server including all of its query parameters. This is where the problem appears&mdash;because that URL is already really, really long, the resulting source URL calling back to the Duo back-end is even longer, and it exceeds the maximum length of a URL in Internet Explorer (2,083 characters) causing IE11 to silently fail, and so you end up with a blank `iframe`. (In our testing, Edge was also impacted, even though according to its documentation it should not have been.)

We [reported](https://github.com/duosecurity/duo_java/issues/4)  this to Duo back in May 2018 and provided a simple one-line fix, which was just to truncate the query parameters off the CAS URL before sending it to the Duo back-end. The response back from the Duo engineer was that by doing that we "may experience breakage with any users using IE7 or other browsers that don't have `window.postMessage` natively supported."

So they're apparently, for whatever reason, trying to maintain backward compatibility with some pretty ancient browsers&mdash;`window.postMessage` has been supported by all major browsers since 2009. But okay, to help preserve their backward compatibility we suggested a slightly "smarter" version of the patch to the Duo engineer&mdash;one that only truncates the URL if `window.postMessage` is supported by the browser (in which case it doesn't need the URL anyway). That's what the patch above does. But we never got a response back to that suggestion, so we ended up making the patch ourselves.

{% include reflinks.md %}
{% include links.html %}

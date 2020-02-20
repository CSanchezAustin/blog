---
layout: post
title:  "Don't Flood Splunk"
categories: identity opendjA
comments: true
---
I was recently preparing for an upcoming live TV event where participants can vote for their favorite artist. I do this before every voting season. My testing consists of running [JMeter] load testing on [BlazeMeter] to simulate the high volume spikes we get during show callouts, such as Save your Favorite Artist from being sent home **NOW!!** Long story short, I forgot to disable [Splunk] log forwarding during the test. Since it's a shared resource and has daily limits an hour or two of load testing can impact users and even shutdown logging in the case of repeated incidents. Akk.

That presented a problem because on of the targets, and LDAP directory [ForgeRock Directory Services] and one of the things I was trying to evaluate was the response times for LDAP calls, or `etimes` as it's known. Since I still have logs locally on an instance of Directory Services I can analyze them. It's still a bit of a pain because Splunk has some really nice built-in functions for [advanced statistics].

Since my analysis focused on `etimes` I needed typical stats such as median, percentiles and standard deviation. Again, simple in Splunk but now I have to calculate these stats on the fly. The solution I came up with would test my [StackOverflow] and Linux fu. I decided to keep it simple and stick to tools that I already had available on my Directory Services environment. The good news is Directory Services can be configured to log in json format. Cool.

My toolset would include:
- Bash
- jq
- awk
- Patience

StackOverflow taught me a bit for calculating [standard deviation using awk]. Here's the gist for the bash command I wrote that I'll walk you though in the comments. Also, I'd appreciate some feedback on ways to improve that `awk` command. 

{% gist 049670fde05c1b991c12e81821a76518 %}

This is an example of the report is produces:

~~~~~~
Protocol   Operation  Status             Tx       Time     Median     Min     Max      90      95      99     StdDev
--------   ---------  ------       --------   --------   --------   -----   -----    ----    ----    ----   --------
LDAP       ADD        SUCCESSFUL       8037       6617   0.823317       0       2       1       1       1   0.390109
LDAP       BIND       SUCCESSFUL       2250       7017    3.11867       0      74      14      14      23    6.11582
LDAP       CONNECT    SUCCESSFUL       2250          0          0       0       0       0       0       0          0
LDAP       DELETE     FAILED            337         68    0.20178       0       1       1       1       1   0.401329
LDAP       DELETE     SUCCESSFUL       7506       6361   0.847455       0       2       1       1       1   0.374788
LDAP       DISCONNECT SUCCESSFUL       2250          0          0       0       0       0       0       0          0
LDAP       EXTENDED   SUCCESSFUL       1800         98  0.0544444       0       1       0       1       1   0.226893
LDAP       MODIFY     SUCCESSFUL       5016       3028   0.603668       0       1       1       1       1   0.489135
LDAP       SEARCH     FAILED           6998       2101   0.300229       0       2       1       1       1   0.458669
LDAP       SEARCH     SUCCESSFUL    1694820     559283   0.329996       0     137       1       1       1   0.767667
LDAP       UNBIND     null             2250          0          0    null    null       0    null    null          0
LDAPS      BIND       SUCCESSFUL          6         83    13.8333      13      14      14      14      14   0.372678
LDAPS      CONNECT    SUCCESSFUL          6          0          0       0       0       0       0       0          0
LDAPS      DISCONNECT SUCCESSFUL          6          0          0       0       0       0       0       0          0
LDAPS      SEARCH     FAILED              3          1   0.333333       0       1       1       1       1   0.471405
LDAPS      SEARCH     SUCCESSFUL        162         45   0.277778       0       4       1       2       3   0.650261
LDAPS      UNBIND     null                6          0          0    null    null    null    null    null          0
internal   ADD        SUCCESSFUL      40805      34854    0.85416       0      82       1       1       1   0.677344
internal   DELETE     SUCCESSFUL      37786      26367   0.697798       0      65       1       1       1   0.568919
internal   MODIFY     SUCCESSFUL      51484      23351   0.453558       0      47       1       1       1   0.538942
~~~~~~

This approach is brute force and takes up a bit of system resources, so it's not advised to run this on a production server. However, you can simply copy the logs and perform the command elsewhere.

I hope you find this useful. Feel free to comment and 



[BlazeMeter]: https://www.blazemeter.com
[JMeter]: https://jmeter.apache.org/
[Splunk]: https://www.splunk.com
[ForgeRock Directory Services]: https://www.forgerock.com/platform/directory-services
[advanced statistics]: https://docs.splunk.com/Documentation/SplunkCloud/8.0.2001/Search/Aboutadvancedstatistics
[StackOverflow]: https://www.stackoverflow.com
[standard deviation using awk]:  https://stackoverflow.com/questions/15101343/standard-deviation-of-an-arbitrary-number-of-numbers-using-bc-or-other-standard

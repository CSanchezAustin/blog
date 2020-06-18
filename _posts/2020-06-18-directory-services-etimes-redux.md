---
layout: post
title: "Directory Services etime redux"
subtitle:  "Or you can always tweak it..."
tags: identity opendj
readtime: 2 min read
author: Chris Sanchez
---
I previously wrote about [Directory Services etime Analysis] where I showed how awk and jq can be powerful tools for working with [ForgeRock Directory Services] logs. In this blog post I take it a step further and improve the code to:
* Fully support Linux dates including millisecond precision
* Specify a start and end time for the analysis (down to the millis)
* Support for relative dates (see Linux: `info date`)
* Added a feature to filter raw logs
* Added a overall transaction summary to the report
* Made a standalone script that can be used as-is.
* Option to output a CSV file

~~~~
Usage: opendj-ops-metrics.sh -a [ auditReport | auditCSV | auditJSON | getLogs ] [ -s <startDate> ] [ -e <endDate> ] [ -f fileList ] [ -r relativeTime ]
script parameters
-a metricAction: what to do: auditReport (table format), auditCSV (csv outformat), filterLogs (json logs). Default: auditReport
-s startDate:  Date to start searching
-e endDate:    Date to end searching
-f fileList: list of files. If none provided, use all audit logs
-r relativeTime: generates a startDate and endDate relative to the current time if neither startDate or endDate is specified
                 if startDate is specified, then the endDate is calculated relative to this parameter
                 if endDate is specified, then the startDate is calculated relative to this parameter

Date (YYYY-MM-DDThh:mm:ss.uuu) YYYY - year, MM - month, DD - day, hh - hour, mm - minute, ss second, uuu millis
 e.g.
      1. get between dates:                      -s 2019-12-13T15:43:04.578 -e 2019-12-14T15:43:04.578
      2. get everything after:                   -s 2019-12-13T15:43:04.578
      3. get everything before:                  -e 2019-12-14T15:43:04.578
      4. get last 10 minutes from current time:  -r "10 min ago"
          Valid modifiers for past times are: -, ago, yesterday, last
      5. get 10 minutes after a start time:      -s 2019-12-13T15:43:04.000 -r "10 min"
      6. get 10 minutes before an end time:      -e 2019-12-14T15:43:04.000 -r "10 min ago"
 IMPORTANT NOTE: when specifying a startDate or endDate, do NOT include the Z timezone designation suffix
~~~~ 

Take note that if you are using startDate and/or endDate, you have to use a date without the trailing `Z`. This is because Linux date calculation is used generate comparable numbers used for filtering in the awk script. For example, if you pass `startDate`, the following code will generate comparable numbers. 
~~~~
        # relative to startTS
        startTS=$(date +%Y%m%d%H%M%S%3N --date "${startDate}")
        endTS=$(date +%Y%m%d%H%M%S%3N --date "${startDate} ${relativeTime}")
~~~~
Passing the date parameter with a `Z` will cause Linux to calculate the date in the current time zone, which may not match up with the logs. Omitting the `Z` will generate a date that is aligned with the log files.  

Here's the new gist for the bash command I wrote that I'll walk you through in the gist comments.
{% gist 73ceaf17e620546f32d9faa35dece344 %}

This is an example of the report produced:

~~~~~~
Protocol   Operation  Status             Tx       Time     Median     Min     Max      90      95      99     StdDev
--------   ---------  ----------   --------   --------   --------   -----   -----    ----    ----    ----   --------
LDAP       ADD        SUCCESSFUL      73892     472145    6.38966       1     325      11      17      45    12.1267
LDAP       BIND       FAILED           1910       7088    3.71099       0     169       4      15      76    15.0926
LDAP       BIND       SUCCESSFUL      11929      48188    4.03957       0     222       5      13      91    14.2503
LDAP       CONNECT    SUCCESSFUL       1887          0          0       0       0       0       0       0          0
LDAP       DISCONNECT SUCCESSFUL       1885          0          0       0       0       0       0       0          0
LDAP       EXTENDED   SUCCESSFUL        260         65       0.25       0      22       1       1       3    1.47381
LDAP       MODIFY     FAILED              8          8          1       0       6       6       6       6    1.93649
LDAP       MODIFY     SUCCESSFUL         99        146    1.47475       0      24       2       3      24    2.44678
LDAP       SEARCH     FAILED          58530     102169    1.74558       0     169       3       7      23     6.0059
LDAP       SEARCH     SUCCESSFUL     580032    1345135    2.31907       0     308       3       5      57     10.626
LDAP       UNBIND     null              325          0          0    null    null    null    null    null          0
LDAPS      BIND       SUCCESSFUL          1         14         14      14      14      14      14      14          0
LDAPS      CONNECT    SUCCESSFUL          1          0          0       0       0       0       0       0          0
LDAPS      SEARCH     SUCCESSFUL          1         34         34      34      34      34      34      34          0
internal   ADD        SUCCESSFUL     370997    1087062    2.93011       1     330       4       5       9    5.74374
internal   MODIFY     SUCCESSFUL     111995      54351   0.485298       0     223       1       1       2    2.44537
--------   ---------  ----------   --------   --------   --------   -----   -----    ----    ----    ----   --------
                          Total:    1213752    3116405    2.56758
~~~~~~

This approach is brute force and takes up a bit of system resources, so it's not advised to run this on a production server. However, you can simply copy the logs and perform the command elsewhere.

I hope you find this useful. Feel free to submit a PR for this article if you have improvements.

[Directory Services etime Analysis]: /directory-services-etimes-analysis

[BlazeMeter]: https://www.blazemeter.com
[JMeter]: https://jmeter.apache.org/
[Splunk]: https://www.splunk.com
[ForgeRock Directory Services]: https://www.forgerock.com/platform/directory-services
[advanced statistics]: https://docs.splunk.com/Documentation/SplunkCloud/8.0.2001/Search/Aboutadvancedstatistics
[StackOverflow]: https://www.stackoverflow.com
[standard deviation using awk]:  https://stackoverflow.com/questions/15101343/standard-deviation-of-an-arbitrary-number-of-numbers-using-bc-or-other-standard

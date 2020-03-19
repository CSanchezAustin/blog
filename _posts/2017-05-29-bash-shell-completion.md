---
layout: post
title: "Bash shell completion for OpenAM/OpenDJ"
subtitle:  "Or let the tab do the trick"
tags: identity opendj bash openam
readtime: 2 min read
author: Chris Sanchez
---
Ludovic Poitou, ForgeRock’s OpenDJ Product Manager, [blogs]{:target="_blank"} about about OpenDJ a bit. One of his posts shows a simple technique for adding a [bash shell completion]{:target="_blank"} for OpenDJ’s administrative tool, *dsconfig*. I liked it because it makes access to the CLI help for dsconfig more convenient. Since I’ve been doing a lot of work with OpenDJ and OpenAM recently, I thought I’d improve it.

One of my projects is called Identity Fabric. It is designed to make it easy to install distributed, production-grade deployments of Identity and Access Management Platforms such as ForgeRock’s OpenDJ and OpenAM. Using Ludo’s technique I create global startup scripts that include bash completions for OpenDJ and OpenAM.

Create `/etc/profile.d/openam.sh` (assuming root permissions, and you’ve already setup SSOTools)

~~~
OPENAM_SSOADM=/openam/openam

echo "export PATH=\${PATH}:${OPENAM_SSOADM}/bin" > /etc/profile.d/openam.sh
echo "complete -W \"$(${OPENAM_SSOADM}/bin/ssoadm --help 2>/dev/null | egrep '^ {4}[a-z]*-[a-z].*' |  sed 's/[ *]*//g')\" ssoadm" >> /etc/profile.d/openam.sh
chmod +x /etc/profile.d/openam.sh
~~~
{: .language-bash}

And logout and back in,
~~~
[ssouser@castor ~]$ ssoadm get-<tab>
get-attr-choicevals   get-auth-cfg-entr     get-identity          get-realm             get-recording-status  get-sub-cfg
get-attr-defs         get-auth-instance     get-identity-svcs     get-realm-svc-attrs   get-revision-number   get-svrcfg-xml
~~~
{: .language-bash}

Create `/etc/profile.d/opendj.sh`

~~~
OPENDJ_HOME_DIR=/opt/opendj

echo "export PATH=\${PATH}:${OPENDJ_HOME_DIR}/bin" > /etc/profile.d/opendj.sh

# sets up command completion
echo "complete -W \"$(${OPENDJ_HOME_DIR}/bin/dsconfig --help-all|grep '^[a-z].*' | tr '\n' ' ')\" dsconfig" >> /etc/profile.d/opendj.sh
echo "complete -W \"$(${OPENDJ_HOME_DIR}/bin/dsreplication --help|grep -o -w '^[a-z-]*$'|grep -v '^--'|tr '\n' ' ')\" dsreplication" >> /etc/profile.d/opendj.sh
chmod +x /etc/profile.d/opendj.sh
~~~
{: .language-bash}

And same as above,

~~~
[ssouser@castor ~]$ dsconfig get-backend-<tab>
get-backend-index-prop      get-backend-prop            get-backend-vlv-index-prop
~~~
{: .language-bash}

#### Quick notes

1. I chose to use /etc/profile.d so any users that log in will get the completion setup for OpenDJ andOpenAM.
2. I took a slightly different approach than Ludo for creating the bash startup file. I resolve the completion words when creating the file (see the echo statement), rather than adding the resolve directly to the bash startup file. On some virtual machines ssoadm and dsconfig can take a long time to return (even with the help subcommand). So each login will be significantly faster.
3. Scripts in /etc/profile.d have to be executable, so remember to chmod +x /etc/profile.d/{openam.sh,opendj.sh}
    
I hope you find this useful. Feel free to submit a PR for this article if you have improvements.

[blogs]: https://ludopoitou.com
[bash shell completion]: https://ludopoitou.com/2011/06/20/opendj-tip-auto-completion-of-dsconfig-command



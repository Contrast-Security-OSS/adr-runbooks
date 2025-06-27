---
layout: runbook
title: "Expression Language Injection"
description: "Guide for addressing server-side code injection vulnerabilities where attackers exploit expression language evaluation to compromise application data and functionality"
---

<!-- \ or two whitespaces used for line breaks -->
# Expression Language Injection Runbook

Expression Language Injection works by taking advantage of server-side code injection vulnerabilities which occur whenever an application incorporates user-controllable data into a string that is dynamically evaluated by a code interpreter. This can lead to complete compromise of the application's data and functionality, as well as the server that is hosting the application.


Example Event - Exploited outcome Expression Language Injection  
`Oct 22 2024 12:09:18.532-0600 192.168.12.70 CEF:0|Contrast Security|Contrast Agent Java|6.5.4|SECURITY|The parameter message had a value that successfully exploited expression-language-injection - T(java.lang.Runtime).getRuntime().exec("whoami")|WARN|pri=expression-language-injection src=0:0:0:0:0:0:0:1 spt=8080 request=/hello requestMethod=GET app=webapplication outcome=EXPLOITED`  
  
  

Example Event - Blocked outcome Expression Language Injection  
`Oct 22 2024 12:14:22.969-0600 192.168.12.70 CEF:0|Contrast Security|Contrast Agent Java|6.5.4|SECURITY|The parameter message had a value that successfully exploited expression-language-injection - T(java.lang.Runtime).getRuntime().exec("whoami")|WARN|pri=expression-language-injection src=0:0:0:0:0:0:0:1 spt=8080 request=/hello requestMethod=GET app=webapplication outcome=BLOCKED`
  
  


\
What is the “outcome” of the event you are triaging? (click to proceed)  

- [Exploited](#exploited)
- [Blocked](#blocked)

- [Ineffective](#ineffective)
- [Success](#success)


## Exploited

An "Exploited" outcome indicates that Contrast detected an input entering an application that resembled Expression Language Injection. It then confirmed that this input was utilized in the evaluation of the expression language.  

To verify this is a true positive, review the following attributes of the event for common indicators:  

- Does the payload contain references to Java classes or methods?
- Does the payload contain os commands?
- Does the payload contain template engine expressions? (e.g. ${...}, #{...}, *{...})
- Is the IP address from a pentester or known vulnerability scanner IP?
- Are there application logs with template engine related error messages around the same timestamp as the event?



\
Examples:

- `T(java.lang.Runtime).getRuntime().exec("whoami")`
- `${pageContext.request.getSession().setAttribute("admin",true)}`
- `*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec('id').getInputStream())}`  

\
Does the event appear to be a true positive? (click to proceed)  

- [No](#exploited-false-positive)  
- [Yes, or unsure](#exploited-true-positive)  



## Blocked

A "Blocked" outcome indicates that Contrast detected an input entering an application that resembled Expression Language Injection. It then confirmed that this input was going to be evaluated by the expression language and therefore blocked it.  

To verify this is a true positive, review the following attributes of the event:

- Does the payload contain references to Java classes or methods?
- Does the payload contain os commands?
- Does the payload contain template engine expressions? (e.g. ${...}, #{...}, *{...})
- Is the IP address from a pentester or known vulnerability scanner IP?
- Are there application logs with template engine related error messages around the same timestamp as the event?


\
Examples:

- `T(java.lang.Runtime).getRuntime().exec("whoami")`
- `${pageContext.request.getSession().setAttribute("admin",true)}`
- `*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec('id').getInputStream())}`  


\
Is the event a true positive? (click to proceed)

- [No](#blocked-false-positive)  
- [Yes, or unsure](#blocked-true-positive)  






## Ineffective

"Ineffective" means Contrast detected an input coming into an application that looked like Expression Language injection, but did not confirm that the input was used evaluated by the expression template engine. This is called a “Probe” within the Contrast UI. This event is a real attack attempt to exploit your application, but it was ineffective. Probes can indicate an attacker scanning or exploring for vulnerabilities.

- Does the probe event appear to be caused by legitimate traffic and numerous similar probe events are being generated, an [exclusion](https://docs.contrastsecurity.com/en/application-exclusions.html) can be configured to clean up Contrast data.  
- Is the probe originating from a specific ip[s] that is a real external IP address (not internal load balancer or network device) and not the public IP address for a large company network?   Consider…  
  - Block using network appliance
  - [Block using Contrast](https://docs.contrastsecurity.com/en/ip-management.html)
- Are all of the events originating from the same application user account  
  - Determine if the account is a legitimate account
  - If so, attempt to help them recover the account by contacting and authenticating the legitimate user, arranging to change their credentials, and recover from any damage.
  - If not,  consider the following options:
    - Ban the account
    - Disable the account
    - Delete the account

\
[Proceed to Post-Incident Activities](#post-incident-activities)  


## Success

“Success" means that Contrast's security measures functioned as intended, preventing unauthorized access or potentially malicious activity from reaching the application. This could be due to a [virtual patch](https://docs.contrastsecurity.com/en/virtual-patches.html), [IP block](https://docs.contrastsecurity.com/en/block-or-allow-ips.html), or [bot block rule](https://docs.contrastsecurity.com/en/server-configuration.html#:~:text=Bot%20blocking%20blocks%20traffic%20from,Events%2C%20use%20the%20filter%20options.) being triggered.  

Generally, these events don't necessitate action because they signify the system is working correctly.  

However, further investigation may be beneficial in specific scenarios to gain more insights or proactively enhance security:

- Should the event have been blocked?:
  - If the event is from an [IP block](https://docs.contrastsecurity.com/en/block-or-allow-ips.html):
    - Correlate the IP address with other events to identify any attempted malicious actions.
    - Look up the IP address's reputation and origin to determine if it's known for malicious activity.
    - Check if the IP is listed on any other denylists across your systems.
  - If the event is from a [Virtual Patch](https://docs.contrastsecurity.com/en/virtual-patches.html):
    - Correlate the event with any exploited or probed events.
    - Confirm if the virtual patch is protecting a known vulnerability in the application.
  - If the event is from a [Bot Block](https://docs.contrastsecurity.com/en/server-configuration.html#:~:text=Bot%20blocking%20blocks%20traffic%20from,Events%2C%20use%20the%20filter%20options.):
    - Analyze the user-agent header of the HTTP request. Only requests originating from known scanning, fuzzing, or malicious user-agents should be blocked.

\
If the event appears to be for legitimate traffic, an [exclusion](https://docs.contrastsecurity.com/en/application-exclusions.html) can be configured.  

\
[Proceed to Post-Incident Activities](#post-incident-activities)  


## Exploited True Positive  

It is possible that the event is a True Positive, but is benign. A Benign True Positive is when an application relies on vulnerable behavior that could potentially be exploited, but is currently necessary for operation. This determination will often require the assistance of the development or application security teams.  

If the event appears to be a Benign True Positive, click [here](#benign-true-positive).  

\
If it does not appear to be a Benign True Positive, the most immediate action to stop an "active" attack would be to block the current attacker of the exploited event, while further triage could result in a [virtual patch](https://docs.contrastsecurity.com/en/virtual-patches.html)/[enabling block mode](https://docs.contrastsecurity.com/en/set-protect-rules.html) for the rule:  

- Is the attack originating from a specific IP[s] that is a real external IP address (not internal load balancer or network device) and not the public IP address for a large company network?
  - Block using network appliance  
  - [Block using Contrast](https://docs.contrastsecurity.com/en/ip-management.html)  
- Are all of the events originating from the same application user account?
  - Determine if the account is a legitimate account  
  - If so, attempt to help them recover the account by contacting and authenticating the legitimate user, arranging to change their credentials, and recover from any damage.
  - If not,  consider the following options:
    - Ban the account
    - Disable the account
    - Delete the account

\
\
Once the current attack has been stopped, consider taking additional steps to prevent future exploitation.  

- If the only “Exploited” events for this rule are true positives, then the rule can be [switched to “Block” mode](https://docs.contrastsecurity.com/en/set-protect-rules.html) which will prevent future exploitation.  
- If there are other “Exploited” events that appear to be legitimate, benign traffic, then “Block” mode would block those events as well, which could have negative impact to the application.  
  - Before enabling “Block” mode for this situation, you must first exclude the legitimate, benign traffic being caught in the rule.  
  - Alternatively, you can set up a [Virtual Patch](https://docs.contrastsecurity.com/en/virtual-patches.html) that only allows the legitimate, benign traffic through and any non-matches will be blocked.

If none of the above options are satisfactory and it's perceived the application is at great risk, you can consider shutting down the application or removing network connectivity.  

\
\
Post Containment

- If confirmed this is a True Positive, it should be raised with the appsec/dev teams to get fixed. Useful information for those teams would be:  

  - Application name
  - Is app in production, development, staging, etc
  - Affected URL
  - Attack payload
  - Stack trace of the request
- To better understand the extent of the incident and to ensure the attack is no longer occurring, look for other IOCs:
  - Did the same IP Address Generate Other Alerts?
  - Is the vulnerability being exploited by other actors?
  - Spike in traffic or repeated access patterns to the vulnerable URL
  - Correlate exploited events with any "probed" or "blocked" events
  - If the attack was able to execute commands on the server, the server may need to be considered compromised and reviewed for persistence and other lateral movement.

\
\
[Proceed to Post-Incident Activities](#post-incident-activities)  



## Exploited False Positive  

If the event seems to be a False Positive, consider the following options:

- Ignore
- [Create Exclusion](https://docs.contrastsecurity.com/en/application-exclusions.html)

\
[Proceed to Post-Incident Activities](#post-incident-activities)  








## Blocked True Positive  

It is possible that the event is a True Positive, but benign. A Benign True Positive is when an application’s design relies on vulnerable behavior that could potentially be exploited, but is currently necessary for operation. This determination will often require the assistance of the development or application security teams.  

If the event appears to be a Benign True Positive, click [here](#benign-true-positive).

If it does not appear to be a Benign True Positive, consider the following options:

- If one IP address is generating a lot of blocked events, it's probably worthwhile to block it.  
- Notify Dev/Appsec team of Vulnerability. Useful information for those teams would be:  
  - Application name
  - Is app in production, development, staging, etc
  - Affected URL
  - payload
  - Stack trace of the request  
- Look for IOCs of further attacks in other parts/inputs of the application
  - Other blocked or probed events?  
  - Did anything show up as "exploited" indicating a different rule did not have blocking enabled?
- Ignore

[Proceed to Post-Incident Activities](#post-incident-activities)  



## Blocked False Positive  

If the event seems to be a False Positive, then Contrast may be blocking legitimate usage of the application, therefore negatively impacting it.

- Create an exclusion to allow the legitimate traffic through so that you can continue to be protected by “Block” mode without the negative impact.
- Alternatively, you can set up a Virtual Patch that only allows legitimate traffic through and any non-matches (attack traffic) will be blocked.  
- If neither of the above options are satisfactory and the negative impact of the application must be avoided, you can switch the rule to “Monitor” mode.

[Proceed to Post-Incident Activities](#post-incident-activities)  


## Benign True Positive

To review, a Benign True Positive occurs when an application relies on vulnerable behavior that could potentially be exploited, but is currently necessary for operation. Consider the following options:

- Ignore
- Create Exclusion  
- Work with the application developer on alternative implementations that do not pose such risk to the application, but meets the business needs.

## Post-Incident Activities

- **Documentation**
  - **Incident Report:** Document the incident, including findings, raw events and alerts, actions taken, assets impacted, and lessons learned.
  - **Update Documentation:** Keep security runbooks and documentation up to date.
- **Communication**
  - **Notify Stakeholders:** Inform relevant stakeholders about the incident and steps taken.
  - **User Notification:** Notify affected users if there was a data breach.
- **Review and Improve**
  - **Review Response:** Conduct a post-mortem to review the response and identify improvement areas.
  - **Enhance Security Posture:** Implement additional security measures and improve monitoring.  

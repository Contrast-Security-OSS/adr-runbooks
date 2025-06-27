---
layout: runbook
title: "Command Injection"
description: "Guide for detecting and responding to command injection attacks, where attackers exploit vulnerabilities to execute arbitrary operating system commands on the server"
---

<!-- \ or two whitespaces used for line breaks -->
# Command Injection Runbook

Command injection is a malicious technique where attackers exploit vulnerabilities in web applications to inject and execute arbitrary operating system (OS) commands on the server. By manipulating input data, attackers can manipulate the application's execution flow, tricking it into running unintended system commands.This can result in data breaches, system compromise, etc. Contrast uses various detection capabilities for Command Injection. Some track malicious input into commands being run, some detect sensitive file paths being accessed, and so forth. 


Example Event - Exploited outcome Command Injection  
`Oct 25 11:02:31 192.168.12.70 CEF:0|Contrast Security|Contrast Agent Java|6.9.0|SECURITY|The parameter cmd had a value that successfully exploited cmd-injection - whoami|WARN|pri=cmd-injection src=0:0:0:0:0:0:0:1 spt=8080 request=/cmd requestMethod=GET app=webapplication outcome=EXPLOITED`  
  
  

Example Event - Blocked outcome Command Injection  
`Oct 25 11:02:31 192.168.12.70 CEF:0|Contrast Security|Contrast Agent Java|6.9.0|SECURITY|The parameter cmd had a value that successfully exploited cmd-injection - whoami|WARN|pri=cmd-injection src=0:0:0:0:0:0:0:1 spt=8080 request=/cmd requestMethod=GET app=webapplication outcome=BLOCKED`
  
  



\
Is the event a "cmd-injection-process-hardening" event?   [Yes](#handling-process-hardening)


\
What is the “outcome” of the event you are triaging? (click to proceed)  

- [Exploited](#exploited)
- [Blocked](#blocked)

- [Ineffective](#ineffective)
- [Success](#success)


## Exploited

"Exploited" means Contrast detected an input coming into an application that looked like command injection and then confirmed the input performed a command injection during a call to the operating system by observing the execution of the injected command and its effects on the system.  

To verify this is a true positive, review the following attributes of the event for common indicators:  

- Does a system command get run in the application as part of normal usage?
- Are there command chaining operators present? (&& || ; & |)
- Are there command substitution characters present? ($(), ``)
- Do the commands seem to be accessing suspicious files? (/etc/password/)
- Is the IP address from a pentester or known vulnerability scanner IP?
- Are there unusual os/system related messages around the same timestamp as the event?
- Are there application logs with OS error messages around the same timestamp as the event?



\
Examples:

- `original_cmd_by_server $(cat /etc/passwd)`
- `original_cmd_by_server; command2`
- `original_cmd_by_server && command2`
- `original_cmd_by_server || command2`  

\
Does the event appear to be a true positive? (click to proceed)  

- [No](#exploited-false-positive)  
- [Yes, or unsure](#exploited-true-positive)  



## Blocked

"Blocked" means Contrast detected an input coming into an application that looked like command injection and then confirmed the input performed a command injection during a call to the operating system and therefore blocked the execution of it.  

To verify this is a true positive, review the following attributes of the event:

- Does a system command get run in the application as part of normal usage?
- Are there command chaining operators present? (&& || ; & |)
- Are there command substitution characters present? ($(), ``)
- Do the commands seem to be accessing suspicious files? (/etc/password/)
- Is the IP address from a pentester or known vulnerability scanner IP?
- Are there unusual os/system related messages around the same timestamp as the event?
- Are there application logs with OS error messages around the same timestamp as the event?


\
Examples:

- `original_cmd_by_server $(cat /etc/passwd)`
- `original_cmd_by_server; command2`
- `original_cmd_by_server && command2`
- `original_cmd_by_server || command2`  


\
Is the event a true positive? (click to proceed)

- [No](#blocked-false-positive)  
- [Yes, or unsure](#blocked-true-positive)  






## Ineffective

"Ineffective" means Contrast detected an input coming into an application that looked like command injection, but did not confirm the input performed a command injection during execution. This is called a “Probe” within the Contrast UI. This event is an unsuccessful attempt at an exploit. They can indicate an attack fuzzing and looking for vulnerabilities.

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
## Handling Process Hardening  

Process Hardening refers to a block that the agent applies to the application to be able to start external processes. Most web applications have no reason to launch external processes so this rule is intended to ensure that no attempts are made by an application to do so.

When triaging this event type the key is in understanding if the web application starts external processes under normal conditions.

Choose the Appropriate Action:  
If the application should NOT start external processes:
- Enable Block Mode: This will prevent the application from launching any external processes, ensuring maximum security.  

\
If the application DOES need to start external processes:

- Monitor Mode (Recommended): Start with monitor mode to observe the commands the application executes. This helps identify legitimate use cases and potential risks without disrupting normal functionality.
- Exclusions: If monitoring reveals legitimate needs, configure an exclusion to allow only specific, trusted external processes to run. Once all legitimate executions are accounted for, the rule can be placed into Block mode.
- Disable the Rule (Use with Caution): If the application requires external process execution and other options are not feasible consider disabling the rule.


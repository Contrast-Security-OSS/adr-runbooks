---
layout: runbook
title: "Cross-Site Scripting (XSS)"
description: "Guide for handling cross-site scripting vulnerabilities where attackers inject malicious JavaScript code into websites viewed by other users"
---

<!-- \ or two whitespaces used for line breaks -->
# Cross-Site Scripting (XSS) Runbook

Cross-site scripting (XSS) is a type of web security vulnerability that allows an attacker to inject malicious JavaScript code into websites viewed by other users. Instead of the website displaying trusted content, the attacker's code is executed, which can compromise user accounts, steal sensitive data, or even take control of the user's browser.

  
  

Example Event - Blocked outcome Cross-Site Scripting (XSS)  
`Oct 08 10:43:57 192.168.12.70 CEF:0|Contrast Security|Contrast Agent Java|6.9.0|SECURITY|The querystring QUERYSTRING had a value that successfully exploited reflected-xss - message=%3Cscript%3Ealert(document.domain(%3C/script%3E|WARN|pri=reflected-xss src=1.1.1.1 spt=8080 request=/error requestMethod=GET app=webapplication outcome=BLOCKED`
  

Example Event 1 - Suspicious outcome Cross-Site Scripting (XSS)  
`Oct 08 08:28:20 192.168.12.70 CEF:0|Contrast Security|Contrast Agent Java|6.9.0|SECURITY|The parameter message had a value that that was marked suspicious reflected-xss - <script>alert(document.domain)</script>|WARN|pri=reflected-xss src=1.1.1.1 spt=8080 request=/xss requestMethod=GET app=webapplication outcome=SUSPICIOUS`  
  


\
What is the “outcome” of the event you are triaging? (click to proceed)  


- [Blocked](#blocked)
- [Suspicious](#suspicious)

- [Success](#success)




## Blocked

"Blocked" means Contrast detected an input coming into an application that looked like a cross-site scripting attack and subsequently blocked it.   

To verify this is a true positive, review the following attributes of the event:

- Are HTML tags included in the payload? (<>,  </>)
- Are suspicious HTML attributes present in the payload? (onerror, onload, onfocus, etc)
- Look for any suspicious protocols within the payload, such as javascript: or data:.
- Are there application logs with relevant error messages?


\
Examples:

- `<script>alert('XSS')</script>`
- `<img src=x onerror=alert('XSS')>`
- `<a href='javascript:alert(1)'>Click me</a>`
- `javascript:alert(1)`
- `data:text/html,<script>alert(1)</script>`  


\
Is the event a true positive? (click to proceed)

- [No](#blocked-false-positive)  
- [Yes, or unsure](#blocked-true-positive)  



## Suspicious

"Suspicious" means Contrast detected an input coming into an application that looked like a cross-site scripting payload. Contrast reports suspicious for non-input tracing rules where Contrast is unable to verify that an attack occurred, and the rule is in monitor mode.  

To verify this is a true positive, review the following attributes of the event:

- Are HTML tags included in the payload? (<>,  </>)
- Are suspicious HTML attributes present in the payload? (onerror, onload, onfocus, etc)
- Look for any suspicious protocols within the payload, such as javascript: or data:.
- Are there application logs with relevant error messages?


\
Examples:

- `<script>alert('XSS')</script>`
- `<img src=x onerror=alert('XSS')>`
- `<a href='javascript:alert(1)'>Click me</a>`
- `javascript:alert(1)`
- `data:text/html,<script>alert(1)</script>`  


\
Is the event a true positive? (click to proceed)

- [No](#suspicious-false-positive)  
- [Yes, or unsure](#suspicious-true-positive) 




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






## Suspicious True Positive  

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

- If the only “Exploited” events for this rule are true positives, then the rule can be switched to “Block” mode which will prevent future exploitation.  
- If there are other “Exploited” events that appear to be legitimate, benign traffic, then “Block” mode would block those events as well, which could have negative impact to the application.  
  - Before enabling “Block” mode for this situation, you must first exclude the legitimate, benign traffic being caught in the rule.  
  - Alternatively, you can set up a Virtual Patch that only allows the legitimate, benign traffic through and any non-matches will be blocked.

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



## Suspicious False Positive

If the event seems to be a False Positive, consider the following options:
- Ignore
- Create Exclusion  

\
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

<!-- \ or two whitespaces used for line breaks -->
# Untrusted Deserialization Issue Runbook

Serialization refers to the process of converting an object into a format which can be saved to a file or a datastore. Deserialization reverses this process, transforming serialized data coming from a file, stream, or network socket into an object. Untrusted Deserialization is a web app/API vulnerability that enables attackers to send malicious serialized objects to an unprotected deserializer. The result is that attackers can cause arbitrary code to run on the web app/API server with full privilege. Attackers can steal data (including code), steal credentials, corrupt data (including code), plant backdoors, run malicious code (such asransomware), take any action the app or API could take, access backend systems, deny service, and launch further attacks.


## Triage

To verify this is a true positive, review the issue:

- [ ] Note the route and data source from the issue title
- [ ] If the untrusted data was captured from the HTTP request or other source, compare them to the data examples below to confirm serialization
  - Note that some serialized objects may come from many other sources besides an HTTP request
- [ ] The evidence should show that untrusted data traversed the application and was passed into a deserialization library
  - See the example traces below to get the idea
  - There may be many steps in the data flow
- [ ] Verify the absence of any defenses that would either
  - Ensure only safe classes are deserialized (allow list)
  - Prevent malicious classes from being deserialized (deny list)
- [ ] Check the details to see if this issue is currently under attack
- [ ] Check the details to see if the Contrast ADR rule for this issue is enabled

Example Trace (Java) - Read from untrusted input stream  
```
stream = request.getNIOInputStream() - service() @ Server.java:41
... additional stream processing events
stream = new ObjectInputStream(org.glassfish.grizzly....) - service() @ Server.java:41
obj = stream.readObject() - service() @ Server.java:43
```

Example Trace (.NET) - Read from untrusted querystring parameter  
```
serialized = query.get_Item("input") -> b1793f0f-76f5-467a-af03-db071100fb2e
stringReader = new StringReader(serialized)
textReader = new JsonTextReader(stringReader)
object = serializer.Deserialize(textReader)
```

Data Examples:

- Java serialized objects usually start with 'rO0'
- `Java - rO0ABXNyABFqYXZhL[...]D//////////3QACmdvb2GUuY29teA==`  
- .NET serialized objects usually start with 'AAEAAAD////'
- `.Net - AAEAAAD/////AQAAAAAAAAAMAgAAAF9TeXN0ZW0u[...]0KPC9PYmpzPgs=`

If your triage is complete and you aren't immediately fixing the issue, please update the status:
- "Confirmed" - you're confident this issue is a true positive
- "Not a problem" - you're confident that this issue is not an exploitable vulnerability
- "Suspicious" - the issue requires additional investigation

Does the event appear to be a true positive? (click to proceed)

- [Yes, or unsure](#remediation)
- [No](#false-positive)  


## Remediation

First, if this issue is serious, you should warn Operations that you've identified a real issue and that it will take some time to fix and get deployed.  Share the issue with them, so that they can start looking for attacks against this type of issue. Threat hunters may be able to search logs to confirm you haven't been attacked or exploited. Operations may also be able to establish some temporary defenses while the fix is being implemented.

There are several approaches to preventing untrusted deserialization, depending on the language and details of your application or API.

- [ ] The first, and safest option is to remove the deserializing of untrusted input completely. Although we believe the techniques below to be   effective, it's worth noting that attacks against serialization have been getting more effective for many years. The consensus amongst security researchers is that developers should be moving away from object serialization when possible.

- [ ] Add validation capabilities to ensure that either only safe classes are deserialized (allow list), or that no known exploitable classes (known as "gadgets") are deserialized (deny list). Either strategy can work effectively, although the allow list approach will require less future maintenance. To help Contrast recognize that the issue has been fixed, please add a "Security Control" in Contrast that indicates that your validation function protects against this attack.

- [ ] If you use Contrast ADR, consider enabling Contrast's ```{Block} mode for the untrusted deserialization rule, which will prevent any remaining vulnerabilities in the application from being exploited.  Before enabling “Block” mode in this situation, be sure your app/API is not relying on untrusted deserialization to perform its function.

In time, Contrast will attempt to confirm remediation automatically if the changes are straightforward. This will automatically set the status to "Remediated-Auto-Verified."  If you want to confirm the fix for yourself, you can exercise the relevant app/API functionality on a server with Contrast installed.  Your test does not have to attempt to exploit the vulnerability, just exercise the app/API in the same way captured in the event. Look at the HTTP request and code for guidance.  Generally, we recommend using an automated test that will exercise this functionality, but a manual test will work as well.  



## False Positive  

If the event seems to be a False Positive, consider the following options:

- [ ] Mark the issue as "Suspicious" if it needs additional investigation
- [ ] Mark the issue as "Not a Problem" and provide a reason if it's a confirmed false positive

Some common reasons that issues are marked "Not a Problem" include:
- This issue is defended by an external security control that prevents this vulnerability from being exploited.
- This issue was reported incorrectly. Please let Contrast Support know.
- This issue is defended by code inside the app/API that prevents this vulnerability from being exploited.
- This issue is only accessible by trusted power users.
- This issue only exists in specific environments, such as test.



## References


- OWASP Deserialization Cheatsheet (https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html)
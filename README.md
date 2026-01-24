# Vulnerability response in Microsoft Security tools (Defender XDR, Cloud & EASM)

New vulnerability announcements are a familiar pattern: a CVE drops late on a Friday or during the holidays, a proof of concept quickly follows, and soon after, the vulnerability is reported as actively exploited in the wild. For vulnerability response and security teams, this immediately triggers two critical questions:

Are we vulnerable, and where exactly do we have the affected software?

In this post, we’ll walk through practical ways to answer those questions using Microsoft Defender XDR, Microsoft Defender for Cloud, and Defender External Attack Surface Management (EASM).

As a real-world example, we’ll use CVE-2025–55182 (React2Shell), a vulnerability that is not trivial to detect and requires correlating information from multiple security data sources.

Case Study: React2Shell (CVE-2025–55182)
After the initial research phase, the first challenge is identifying whether the vulnerable components even exist in our environment. In this case, we are looking for specific versions of JavaScript frameworks:

React version: 19.0.0,19.1.0, 19.1.1, 19.2.0
Next.js version: 15.0.0–15.0.4, 15.1.0–15.1.8, 
15.2.0–15.2.5, 15.3.0–15.3.5, 15.4.0–15.4.7, 15.5.0–15.5.6, 
16.0.0–16.0.6, 14.3.0-canary.77 and later canary releases
Well, we barely know how many newest versions of Windows 11 we have in the organisation, the speed race to upgrade all Windows 10 just finished. Now we need to find some JavaScript frameworks and libraries versions? Not many IT security teams are tracking framework/packages/libraries/databases/software components versioning.

Let's start our deep dive.

1. Defender XDR — Vulnerability Profiles
Press enter or click to view image in full size

Overview of Vulnerability Profile
First place to check in case of a new “Celebrity” vulnerability. I mean high/critical vulnerability that is exploited in the wild, PoC was released, and half of the CyberSec world is talking about it (for a few days, till the next big thing happens).

Vulnerability profiles are curated by Microsoft Security teams and are useful material for learning about:

Technical vulnerability details
Press enter or click to view image in full size

Details about vulnerability and ongoing exploitation
Affected assets within your organisation
Press enter or click to view image in full size

Insigths into assets impact in our organisation
Existing Microsoft detections and mitigations
Press enter or click to view image in full size

Detections and remediation available in Microsoft security products
Additional hunting queries and investigation guidance
Press enter or click to view image in full size

Advanced Hunting queries for React2Shell
What matters most?

Change log. Vulnerability profiles are being updated over time. It’s common for new detection logic, impacted assets, or mitigations to appear days after the initial publication. Make a habit of revisiting the profile as it matures.

Press enter or click to view image in full size

Change log
Pro tip #1: Visit and configure Threat Analytics email notifications for Vulnerability profiles https://security.microsoft.com/securitysettings/defender/email_notifications

Press enter or click to view image in full size

Example of email updates in Vulnerability profile
Related incidents: here we can see if detections from antivirus and EDR associated with our CVE already triggered on monitored devices

Press enter or click to view image in full size

Related incidents from Vulnerability Profile
Detections in other Microsoft Security products. From there, we can pivot to other tools to see how they are helping with vulnerability response. It can be an Azure WAF rule example, Defender EASM signatures, AV and EDR detections, or a dedicated Defender for Endpoint CVE vulnerability page.

Press enter or click to view image in full size

Defender VM signature and Azure WAF policy
2. Microsoft Defender for Vulnerability Management (Defender VM)
Defender for Endpoint (software agent allowing the collection of telemetry for Vulnerability management) is the backbone of host-based vulnerability detection. Defender for Vulnerability Management continues to expand its coverage, including signals from Defender for Cloud.

When investigating a newly disclosed CVE, I suggest:

Remove filters such as “Affects my organization” to review the vulnerability details even before exposure is confirmed.
Open the CVE page directly to monitor updates and emerging context.
Press enter or click to view image in full size

CVE searching
Press enter or click to view image in full size

Removing filters
Press enter or click to view image in full size

Pivoting to CVE page
Press enter or click to view image in full size

CVE Page
Press enter or click to view image in full size

Vulnerable software in our organisation
Pro tip #2: As in Vulnerability Profile, we can create a notification for Vulnerability page creation and update.
https://security.microsoft.com/securitysettings/endpoints/email_notifications

Press enter or click to view image in full size

Notification rule for new Critical vulnerability
3. Defender for External Attack Surface Management (EASM)
Defender EASM may be underutilized, but it is extremely valuable for external attack surface monitoring and CVE hunting.

EASM allows us to search for exposed components and technologies associated with specific CVEs across our internet-facing assets.

Press enter or click to view image in full size

Search for CVE signatures
Press enter or click to view image in full size

Host with React2shell detections
While some findings may be marked as [Potential], they often provide strong leads during early-stage vulnerability response. For vulnerabilities like React2Shell, EASM can quickly surface externally visible applications running affected frameworks.

4. Defender for Cloud & Azure Resource Graph
In the cloud, where all kinds of resources have their own CVEs, we can leverage Defender for Cloud CSPM (Cloud Security Posture Management). Microsoft Defender CSPM provides advanced security posture capabilities, including agentless vulnerability scanning, but needs to be properly configured (and paid per resource):

Press enter or click to view image in full size

Defender CSPM settings
If enabled, we can look for vulnerability insights in Defender for Cloud itself:

Press enter or click to view image in full size

Defender for Cloud — Explorer search for CVE
Or we can query them by querying the Azure Resource Graph using the explorer as below:

Press enter or click to view image in full size

Azure Resource Graph CVE search
5. Defender XDR Advanced Hunting & Global search
If we have Defender for Endpoint telemetry and logs, we are able to hunt for artefacts related to vulnerable software.

For React2Shell, affected packages include:

react-server-dom-webpack
react-server-dom-parcel
react-server-dom-turbopack
Let's take the “react-server” string and hunt for affected packages using KQL to identify potential exposure:

union DeviceFileEvents, DeviceProcessEvents
| where TimeGenerated > ago (30d)
| where * contains "react-server"
Based on this query, we can find presense of “react-server” in our organisation and narrow down machines to be checked more deeply. In case of other vulnerabilities, e.g., MongoBleed, we can search for the “mongod.log” file presence.

We can also do a similar search from Defender XDR using Global search:

Press enter or click to view image in full size

This method can be used in the first minutes after CVE details disclosure. It does not rely on Microsoft signatures and content.

Pro tip #4: Generative AI tools can be surprisingly effective at identifying relevant artefacts based on vulnerability descriptions.

Summary
There are many ways to find information during Vulnerability response in Microsoft Security tools, and there is no single place that provides a complete picture. Signatures, Vulnerability profiles, and content created by Microsoft are not always present in the first hours after the release of vulnerability details.

Key takeaways:

Test these methods in your own environment before an incident occurs.
Enable Defender CSPM. It provides high value for relatively low cost, especially for agentless vulnerability scanning.
Assume that initial detection coverage may be incomplete and rely on hunting and telemetry correlation in the first hours of Vulnerability response.

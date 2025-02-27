---
title: SBC connectivity issues
description: Describes how to diagnose SIP OPTIONS or TLS certificate issues with SBC. 
ms.date: 2/5/2021
author: cloud-writer
ms.author: meerak
manager: dcscontentpm
audience: Admin
ms.topic: troubleshooting
ms.service: msteams
localization_priority: Normal
search.appverid:
- SPO160
- MET150
appliesto:
- Microsoft Teams
ms.custom: 
- CI 124780
- CSSTroubleshoot 
ms.reviewer: mikebis
---

# SBC connectivity issues

When you use Direct Routing, you might experience the following Session Border Controller (SBC) connectivity issues:

- **Session Initiation Protocol** (SIP) OPTIONS are not received.
- **Transport Layer Security** (TLS) connections problems occur.
- The SBC doesn’t respond.
- The SBC is marked as inactive in the Admin portal.

Such issues are most likely caused by either or both of the following conditions:

- A TLS certificate experiences problems.
- An SBC is not configured correctly for Direct Routing.

This article lists some common issues that are related to SIP options and TLS certificates, and provides resolutions that you can try.  

## Overview of the SIP OPTIONS process

- The SBC sends a TLS connection request that includes a TLS certificate to the SIP proxy server Fully Qualified Domain Name (FQDN) (for example, **sip.pstnhub.microsoft.com**).

- The SIP proxy checks the connection request.

  - If the request is not valid, the TLS connection is closed and the SIP proxy does not receive SIP OPTIONS from the SBC.
  - If the request is valid, the TLS connection is established, and the SBC uses it to send SIP OPTIONS to the SIP proxy.

- After it receives SIP OPTIONS, the SIP proxy checks the Record-Route to determine whether the SBC FQDN belongs to a known tenant. If the FQDN information is not detected there, the SIP proxy checks the Contact header.

- If the SBC FQDN is detected and recognized, the SIP proxy sends a **200 OK** message by using the same TLS connection.

- The SIP proxy sends SIP OPTIONS to the SBC FQDN that is listed in the Contact header of the SBC SIP OPTIONS.

- After receiving SIP OPTIONS from the SIP proxy, the SBC responds by sending a **200 OK** message. This step confirms that the SBC is healthy.

- As the final step, the SBC is marked as **Active** in the Teams Admin portal.

> [!NOTE]
> In a [hosted model](/microsoftteams/direct-routing-sbc-multiple-tenants), SIP OPTIONS should be sent from only the hosted SBC. The status of SBCs that are in a derived trunk model are based on the main SBC.

### SIP OPTIONS issues

After the TLS connection is successfully established, and the SBC is able to send and receive messages to and from the Teams SIP proxy, there might still be problems that affect the format or content of SIP OPTIONS.
<br><br>
<details>
<summary><b>SBC doesn't receive a "200 OK" response from SIP proxy</b></summary>

This situation might occur if you’re using an older version of TLS. To enforce stricter security, enable TLS 1.2.

Make sure that your SBC certificate is not self-signed and that you got it from a trusted Certificate Authority (CA).

If you’re using the minimum required version of TLS, and your SBC certificate is valid, then the issue might occur because the FQDN is misconfigured in your SIP profile and not recognized as belonging to any tenant. Check for the following conditions, and fix any errors that you find:

- The FQDN provided by the SBC in the Record-Route or Contact header is different from what is configured in Teams.
- The Contact header contains an IP address instead of the FQDN.
- The domain isn’t [fully validated](/microsoft-365/admin/setup/add-domain). If you add an FQDN that wasn’t validated previously, you must validate it now.
- After you register an SBC domain name, you must activate it by [adding at least one E3- or E5-licensed user](/microsoftteams/direct-routing-connect-the-sbc#connect-the-sbc-to-the-tenant).

</details>

<details>

<summary><b>SBC receives "200 OK" response but not SIP OPTIONS</b></summary>

The SBC receives the **200 OK** response from the SIP proxy but not the SIP OPTIONS that were sent. If this error occurs, make sure that the FQDN that's listed in the Record-Route or Contact header is correct and resolves to the correct IP address.

Another possible cause for this issue might be firewall rules that are preventing incoming traffic. Make sure that firewall rules are configured to allow incoming connections.

</details>

<details>
<summary><b>SBC status is intermittently inactive</b></summary>

This issue might occur if the SBC is configured to send SIP OPTIONS not to FQDNs but to the specific IP addresses that they resolve to. During maintenance or outages, these IP addresses might change to a different datacenter. Therefore, the SBC will be sending SIP OPTIONS to an inactive or unresponsive datacenter. Do the following:

- Make sure that the SBC is discoverable and configured to send SIP OPTIONS to only FQDNs.
- Make sure that all devices in the route, such as SBCs and firewalls, are configured to allow communication to and from all Microsoft-signaling FQDNs.
- To provide a failover option when the connection from an SBC is made to a datacenter that's experiencing an issue, the SBC must be configured to use all three SIP proxy FQDNs:

  - sip.pstnhub.microsoft.com
  - sip2.pstnhub.microsoft.com
  - sip3.pstnhub.microsoft.com

  > [!NOTE]
  > Devices that support DNS names can use sip-all.pstnhub.microsoft.com to resolve to all possible IP addresses.

For more information, see [SIP Signaling: FQDNS](/microsoftteams/direct-routing-plan#sip-signaling-fqdns).

</details>

<details>
<summary><b>FQDN doesn’t match the contents of CN or SAN certificates</b></summary>

This issue occurs if a wildcard doesn't match a lower-level subdomain. For example, the wildcard `\*\.contoso.com` would match sbc1.contoso.com, but not customer10.sbc1.contoso.com. You can't have multiple levels of subdomains under a wildcard. If the FQDN doesn’t match the contents of the Common Name (CN) certificate or Subject Alternate Name (SAN) certificate, request a certificate that matches your domains.

For more information about certificates, see the **Public trusted certificate for the SBC** section of [Plan Direct Routing](/MicrosoftTeams/direct-routing-plan#public-trusted-certificate-for-the-sbc).
</details>

<details>
<summary><b>Domain activation is not registered in the Microsoft 365 environment</b></summary>

To fully activate a domain for a tenant and distribute it over the Microsoft 365 environment, you must assign at least one licensed user to the subdomain that's used by the SBC. When all the requirements are met, it may take up to 24 hours for the domain to be activated.

For a list of the licenses that are required for Direct Routing, see the "Licensing and other requirements" section of [Plan Direct Routing](/MicrosoftTeams/direct-routing-plan#licensing-and-other-requirements).

For more information about this process, see the **Connect the SBC to the tenant** section of [Connect your Session Border Controller (SBC) to Direct Routing](/microsoftteams/direct-routing-connect-the-sbc#connect-the-sbc-to-the-tenant).
</details>

### TLS connection issues

If the TLS connection is closed right away and SIP OPTIONS are not received from the SBC, or if **200 OK** is not received from the SBC, then the problem might be with the TLS version. The TLS version configured on the SBC should be 1.2 or higher.
<br><br>
<details>

<summary><b>SBC certificate is self-signed or not from a trusted CA</b></summary>

If the SBC certificate is self-signed, it is not valid. Make sure that the SBC certificate is obtained from a trusted Certificate Authority (CA). The certificate must contain at least one FQDN that belongs to a Microsoft 365 tenant.

For a list of supported CAs, see the **Public trusted certificate for the SBC** section of [Plan Direct Routing](/MicrosoftTeams/direct-routing-plan#public-trusted-certificate-for-the-sbc).

</details>

<details>
<summary><b>SBC doesn’t trust SIP proxy certificate</b></summary>

If the SBC doesn't trust the SIP proxy certificate, download and install the Baltimore CyberTrust root certificate on the SBC. To download the certificate, see [Microsoft 365 encryption chains](/microsoft-365/compliance/encryption-office-365-certificate-chains).

For a list of supported CAs, see the **Public trusted certificate for the SBC** section of [Plan Direct Routing](/MicrosoftTeams/direct-routing-plan#public-trusted-certificate-for-the-sbc).

</details>

<details>
<summary><b>SBC certificate is invalid</b></summary>

If the [Health Dashboard for Direct Routing](/microsoftteams/direct-routing-health-dashboard) in the Teams admin center indicates that the SBC certificate is expired or revoked, request or renew the certificate from a trusted Certificate Authority (CA). Then, install it on the SBC.

For a list of supported CAs, see the **Public trusted certificate for the SBC** section of [Plan Direct Routing](/MicrosoftTeams/direct-routing-plan#public-trusted-certificate-for-the-sbc).

</details>

<details>
<summary><b>SBC certificate or intermediary certificates are missing in the SBC TLS "Hello" message</b></summary>

Check that a valid SBC certificate and all required intermediate certificates are installed correctly, and that the TLS connection settings on the SBC are correct.

Sometimes, even if everything looks correct, a closer examination of the packet capture might reveal that the TLS certificate is not provided to the Teams infrastructure.

</details>

<details>
<summary><b>SBC connection is interrupted</b></summary>

The TLS connection is interrupted or not set up even though the certificates and SBC settings experience no issues.

The TLS connection may have been closed by one of the intermediary devices (such as a firewall or a router) on the path between the SBC and the Microsoft network. Check for any connection issues within your managed network, and fix them.

</details>

## More information

Still need help? Go to [Microsoft Community](https://answers.microsoft.com/).

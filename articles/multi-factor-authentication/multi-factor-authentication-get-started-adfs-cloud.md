﻿---
title: Secure cloud resources with Azure MFA and AD FS
description: This is the Azure Multi-Factor authentication page that describes how to get started with Azure MFA and AD FS in the cloud.
services: multi-factor-authentication
documentationcenter: ''
author: kgremban
manager: femila
editor: yossib

ms.assetid: 0927fc67-8090-4fdd-913a-b3cfed3fbe77
ms.service: multi-factor-authentication
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 10/14/2016
ms.author: kgremban

---
# Securing cloud resources with Azure Multi-Factor Authentication and AD FS
If your organization is federated with Azure Active Directory, use Azure Multi-Factor Authentication or Active Directory Federation Services to secure resources that are accessed by Azure AD. Use the following procedures to secure Azure Active Directory resources with either Azure Multi-Factor Authentication or Active Directory Federation Services.

## Secure Azure AD resources using AD FS
To secure your cloud resource, first enable an account for users, then set up a claims rule. Follow this procedure to walk through the steps:

1. Use the steps outlined in [Turn-on multi-factor authentication for users](multi-factor-authentication-get-started-cloud.md#turn-on-two-step-verification-for-users) to enable an account.
2. Start the AD FS Management console.
   ![Cloud](./media/multi-factor-authentication-get-started-adfs-cloud/adfs1.png)
3. Navigate to **Relying Party Trusts** and right-click on the Relying Party Trust. Select **Edit Claim Rules…**
4. Click **Add Rule…**
5. From the drop-down, select **Send Claims Using a Custom Rule** and click **Next**.
6. Enter a name for the claim rule.
7. Under Custom rule: add the following text:

    ```
    => issue(Type = "http://schemas.microsoft.com/claims/authnmethodsreferences", Value = "http://schemas.microsoft.com/claims/multipleauthn");
    ```

    Corresponding claim:

    ```
    <saml:Attribute AttributeName="authnmethodsreferences" AttributeNamespace="http://schemas.microsoft.com/claims">
    <saml:AttributeValue>http://schemas.microsoft.com/claims/multipleauthn</saml:AttributeValue>
    </saml:Attribute>
    ```
8. Click **OK** then **Finish**. Close the AD FS Management console.

Users then can complete signing in using the on-premises method (such as smartcard).

## Trusted IPs for federated users
Trusted IPs allow administrators to by-pass two-step verification for specific IP addresses, or for federated users that have requests originating from within their own intranet. The following sections describe how to configure Azure Multi-Factor Authentication Trusted IPs with federated users and by-pass two-step verification when a request originates from within a federated users intranet. This is achieved by configuring AD FS to use a pass-through or filter an incoming claim template with the Inside Corporate Network claim type.

This example uses Office 365 for our Relying Party Trusts.

### Configure the AD FS claims rules
The first thing we need to do is to configure the AD FS claims. We will create two claims rules, one for the Inside Corporate Network claim type and an additional one for keeping our users signed in.

1. Open AD FS Management.
2. On the left, select **Relying Party Trusts**.
3. Right-click on **Microsoft Office 365 Identity Platform** and select **Edit Claim Rules…**
   ![Cloud](./media/multi-factor-authentication-get-started-adfs-cloud/trustedip1.png)
4. On Issuance Transform Rules, click **Add Rule.**
   ![Cloud](./media/multi-factor-authentication-get-started-adfs-cloud/trustedip2.png)
5. On the Add Transform Claim Rule Wizard, select **Pass Through or Filter an Incoming Claim** from the drop-down and click **Next**.
   ![Cloud](./media/multi-factor-authentication-get-started-adfs-cloud/trustedip3.png)
6. In the box next to Claim rule name, give your rule a name. For example: InsideCorpNet.
7. From the drop-down, next to Incoming claim type, select **Inside Corporate Network**.
   ![Cloud](./media/multi-factor-authentication-get-started-adfs-cloud/trustedip4.png)
8. Click **Finish**.
9. On Issuance Transform Rules, click **Add Rule**.
10. On the Add Transform Claim Rule Wizard, select **Send Claims Using a Custom Rule** from the drop-down and click **Next**.
11. In the box under Claim rule name: enter *Keep Users Signed In*.
12. In the Custom rule box, enter:

        c:[Type == "http://schemas.microsoft.com/2014/03/psso"]
            => issue(claim = c);
    ![Cloud](./media/multi-factor-authentication-get-started-adfs-cloud/trustedip5.png)
13. Click **Finish**.
14. Click **Apply**.
15. Click **Ok**.
16. Close AD FS Management.

### Configure Azure Multi-Factor Authentication Trusted IPs with Federated Users
Now that the claims are in place, we can configure trusted IPs.

1. Sign in to the [Azure classic portal](https://manage.windowsazure.com).
2. On the left, click **Active Directory**.
3. Under Directory, select the directory where you want to set up trusted IPs.
4. On the Directory you have selected, click **Configure**.
5. In the multi-factor authentication section, click **Manage service settings**.
6. On the Service Settings page, under trusted IPs, select **Skip multi-factor-authentication for requests from federated users on my intranet.**
   ![Cloud](./media/multi-factor-authentication-get-started-adfs-cloud/trustedip6.png)
7. Click **save**.
8. Once the updates have been applied, click **close**.

That’s it! At this point, federated Office 365 users should only have to use MFA when a claim originates from outside the corporate intranet.

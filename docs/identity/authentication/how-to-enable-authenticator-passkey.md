---
title: How to enable passkeys in Microsoft Authenticator for Microsoft Entra ID
description: Learn about how to enable passkeys in Microsoft Authenticator for Microsoft Entra ID.

ms.service: entra-id
ms.subservice: authentication
ms.topic: how-to
ms.date: 08/19/2024

ms.author: justinha
author: justinha
manager: amycolannino
ms.reviewer: mjsantani

# Customer intent: As a Microsoft Entra Administrator, I want to learn how to enable and enforce passkeys in Authenticator sign in for end users.
---
# Enable passkeys in Microsoft Authenticator

This article lists steps to enable and enforce use of passkeys in Authenticator for Microsoft Entra ID. First, you update the Authentication methods policy to allow end users to register and sign in with passkeys in Authenticator. Then you can use Conditional Access authentication strengths policies to enforce passkey sign-in when users access a sensitive resource.

## Requirements

- [Microsoft Entra multifactor authentication (MFA)](howto-mfa-getstarted.md)
- Android 14 and later or iOS 17 and later
- An active internet connection on any device that is part of the passkey registration/authentication process
- For cross-device registration and authentication, both devices must have Bluetooth enabled. For more information about how to allow Bluetooth usage only for passkey in Microsoft Authenticator, see [Restrict Bluetooth usage to passkeys in Authenticator](how-to-support-authenticator-passkey.md#restrict-bluetooth-usage-to-passkeys-in-authenticator).

> [!NOTE]
> Users need to install the latest version of Authenticator for Android or iOS to use a passkey. 

To learn more about where you can use passkeys in Authenticator to sign in, see [Support for FIDO2 authentication with Microsoft Entra ID](fido2-compatibility.md).

## Enable passkeys in Authenticator in the admin center

To enable passkeys in Authenticator, edit the **Passkey (FIDO2)** authentication method policy. The **Microsoft Authenticator** policy doesn't have an option to enable passkeys. 

1. Sign in to the [Microsoft Entra admin center](https://entra.microsoft.com) as at least an [Authentication Policy Administrator](~/identity/role-based-access-control/permissions-reference.md#authentication-policy-administrator).
1. Browse to **Protection** > **Authentication methods** > **Authentication method policy**.
1. Under the method **Passkey (FIDO2)**, select **All users** or **Add groups** to select specific groups. *Only security groups are supported*.
1. On the **Configure** tab, set:
   - **Allow self-service set up** to **Yes**
   - **Enforce attestation** to **No**
   - **Enforce key restrictions** to **Yes**
   - **Restrict specific keys** to **Allow**
   - Select **Microsoft Authenticator (Preview)** if the checkbox is displayed in the admin center. This setting automatically populates the Authenticator app AAGUIDs for you in the key restriction list. Otherwise, you can manually add the following AAGUIDs to enable the Authenticator passkey preview:

      - **Authenticator for Android:** de1e552d-db1d-4423-a619-566b625cdc84
      - **Authenticator for iOS:** 90a3ccdf-635c-4729-a248-9b709135078f

   :::image type="content" border="true" source="media/how-to-enable-authenticator-passkey/optional-settings.png" alt-text="Screenshot showing Microsoft Authenticator enabled for passkey."lightbox="media/how-to-enable-authenticator-passkey/optional-settings.png":::

  >[!WARNING]
  >Key restrictions set the usability of specific passkeys for both registration and authentication. If you change key restrictions and remove an AAGUID that you previously allowed, users who previously registered an allowed method can no longer use it for sign-in. If your organization doesn't currently enforce key restrictions and already has active passkey usage, you should collect the AAGUIDs of the keys being used today. Add them to the Allow list, along with the Authenticator AAGUIDs, to enable this preview. This task can be done with an automated script that analyzes logs such as registration details and sign-in logs.

## Optional settings

### Allow self-service set up

Set to **Yes** (default). If set to no, your users aren't able to register a passkey through MySecurityInfo, even if enabled by Authentication Methods policy.  

### Enforce attestation

Should be set to **No** for preview. Attestation support is planned for General Availability.

What kind of attestation is supported?

What are the implications for choosing attestation?

### Key restriction policy

Set **Enforce key restrictions** to **Yes** if your organization wants to only allow or disallow certain passkeys, which are identified by their Authenticator Attestation GUID (AAGUID). If you want, you can select **Microsoft Authenticator (Preview)** to add AAGUIDs to specifically restrict Authenticator on iOS and Android, or you can manually add the following AAGUIDs to enable the Authenticator passkey preview:

- **Authenticator for Android:** de1e552d-db1d-4423-a619-566b625cdc84
- **Authenticator for iOS:** 90a3ccdf-635c-4729-a248-9b709135078f

After you finish optional settings, select **Save**.

## Enable passkeys in Authenticator using Graph Explorer

In addition to using the Microsoft Entra admin center, you can also enable passkeys in Authenticator by using Graph Explorer. Those assigned at least the [Authentication Policy Administrator](../role-based-access-control/permissions-reference.md#authentication-policy-administrator) role can update the Authentication methods policy to allow the AAGUIDs for Authenticator. 

To configure the policy by using Graph Explorer:

1. Sign in to [Graph Explorer](https://aka.ms/ge) and consent to the **Policy.Read.All** and **Policy.ReadWrite.AuthenticationMethod** permissions.

1. Retrieve the Authentication methods policy: 

   ```json
   GET https://graph.microsoft.com/beta/authenticationMethodsPolicy/authenticationMethodConfigurations/FIDO2
   ```

1. To disable attestation enforcement and enforce key restrictions to only allow AAGUIDs for Microsoft Authenticator, perform a PATCH operation using the following request body:

   ```json
   PATCH https://graph.microsoft.com/beta/authenticationMethodsPolicy/authenticationMethodConfigurations/FIDO2
   
   Request Body:
   {
       "@odata.type": "#microsoft.graph.fido2AuthenticationMethodConfiguration",
       "isAttestationEnforced": false,
       "keyRestrictions": {
           "isEnforced": true,
           "enforcementType": "allow",
           "aaGuids": [
               "90a3ccdf-635c-4729-a248-9b709135078f",
               "de1e552d-db1d-4423-a619-566b625cdc84"
   
               <insert previous AAGUIDs here to keep them stored in policy>
           ]
       }
   }
   ```

1. Make sure that the passkey (FIDO2) policy is updated properly.

   ```json
   GET https://graph.microsoft.com/beta/authenticationMethodsPolicy/authenticationMethodConfigurations/FIDO2
   ```


## Restrict Bluetooth usage to passkeys in Authenticator

Some organizations restrict Bluetooth usage, which includes the use of passkeys. In such cases, organizations can allow passkeys by permitting Bluetooth pairing exclusively with passkey-enabled FIDO2 authenticators. for more information about how to configure Bluetooth usage only for passkeys, see [Passkeys in Bluetooth-restricted environments](https://review.learn.microsoft.com/en-us/windows/security/identity-protection/passkeys/?branch=pr-en-us-10051&tabs=windows#passkeys-in-bluetooth-restricted-environments).

## Delete a passkey 

If a user deletes a passkey in Authenticator, the passkey is also removed from the user's sign-in methods. An Authentication Policy Administrator can also follow these steps to delete a passkey from the user’s authentication methods, but it won't remove the passkey from Authenticator.

1. Sign in to the [Microsoft Entra admin center](https://entra.microsoft.com) and search for the user whose passkey needs to be removed.
1. Select **Authentication methods** > right-click **FIDO2 security key** and select **Delete**. 

    ![Screenshot of View Authentication Method details.](media/howto-authentication-passwordless-deployment/security-key-view-details.png)

> [!NOTE]
> Unless the user initiated the passkey deletion themselves in Authenticator, they need to also remove the passkey in Authenticator on their device.

## Enforce sign-in with passkeys in Authenticator 

To make users sign in with a passkey when they access a sensitive resource, use the built-in phishing-resistant authentication strength, or create a custom authentication strength by following these steps:

1. Sign in to the [Microsoft Entra admin center](https://entra.microsoft.com) as a Conditional Access Administrator.
1. Browse to **Protection** > **Authentication methods** > **Authentication strengths**.
1. Select **New authentication strength**.
1. Provide a descriptive **Name** for your new authentication strength.
1. Optionally provide a **Description**.
1. Select **Passkeys (FIDO2)** and then select **Advanced options**.
1. Add AAGUIDs for passkeys in Authenticator:

   - **Authenticator for Android:** de1e552d-db1d-4423-a619-566b625cdc84
   - **Authenticator for iOS:** 90a3ccdf-635c-4729-a248-9b709135078f

1. Choose **Next** and review the policy configuration.

## Next steps

[Support for passkey in Windows](/windows/security/identity-protection/passkeys)


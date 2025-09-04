---
title : "How a Broken Password Reset Workflow Led to Account Takeover"
date : 2025-09-04T08:00:39+01:00
description : "While testing the password reset functionality of an application, I discovered a critical logic flaw that allowed me to fully compromise user accounts without access to their email."
---


## **Overview**

Password reset mechanisms are among the most security-critical features in modern web applications. They are intended to provide users with a secure fallback in case of lost credentials, but design flaws in their implementation can convert them into direct vectors for account takeover. This research dissects a real-world vulnerability where the password reset token was incorrectly bound, enabling attackers to hijack arbitrary accounts through crafted API requests.

Password reset flows generally follow a standard model:

1. User requests a reset → system generates a one-time token.

2. Token is delivered to the user via a trusted channel (usually email or SMS).

3. User presents the token along with a new password → server validates both and updates credentials.

The assumption is that the reset token uniquely maps to the account that requested it. If this binding is broken, the entire mechanism collapses.


![https://x.com/descopeinc/status/1597644569348558848](https://i.imgur.com/LnhDNZO.png#center)


## **The vulnerability** 

1. **Initiate Reset with Attacker Account** :
The attacker begins by attempting to log in and selecting the “Forgot Password” option. They provide their own registered email address on the application, which triggers the system to send an OTP to their inbox.

2. **Intercept the Reset Request** :
Using a proxy tool such as Burp Suite, the attacker captures the final password reset request, which contains the following structure:

```
POST /api/v2/auth/forgot-password/reset
Content-Type: application/json

{
	"password":"newPassword123",
	"email":"attacker@example.com",
	"token":"256504"
}
```

> At this stage, the server expects a valid token and the new password.

3. **Manipulate the Email Parameter**
Instead of leaving the `email` field as their own, the attacker substitutes it with the target user’s email address:


```
{
  "password": "newPassword123",
  "email": "victim@example.com",
  "token": "256504"
}
```

> Crucially, the backend fails to validate that the token was originally generated for the supplied email address.

4. **Result: Unauthorized Account Takeover**
The server responds with a success message:

```
{"status":true,"message":"Password change was successful"}
```

> In reality, the victim’s account password has now been overwritten with the attacker’s chosen password, granting the attacker full access.

_This vulnerability arises because the reset token was only validated in isolation, without ensuring a strict binding to the original account for which it was issued. As a result, the attacker is able to weaponize their own valid reset token to reset the password of any arbitrary account._

## **Why This Was Possible:**

- Over-trust in client input → the backend trusted the `email` parameter from the request body instead of binding the reset process solely to the token.

- Improper state management → the token should have been uniquely associated with the user account at issuance, but the system treated **token** and **email** as independent variables.

- Confused deputy problem → the server acted on behalf of the attacker (who held a valid **token**) to reset the password of a different identity, effectively misusing its own authority.

## **Security Implications:**

This flaw allows full account takeover without access to the victim’s inbox. In real-world terms:

- Attacks scale easily — attackers can automate password resets against large user bases.

- Privileged accounts (admins, agents, support) are equally vulnerable.

- Secondary attacks such as data exfiltration, privilege escalation, or fraud could follow.

## **Mitigation Strategies:**

- Token-Account Binding → When a reset token is generated, store it server-side with a strict reference to the account. The client should never need to submit the email parameter during reset.

- Single Source of Truth → The reset token alone should determine which account is updated.

- Minimal Client Input → Reduce parameters; unnecessary fields like email in sensitive flows should be removed.

- Audit & Test Reset Flows → Include logic flaw testing in QA, not just brute-force or expiry testing.

## **Conclusion:**

This case illustrates how subtle logic oversights in authentication workflows can escalate into systemic vulnerabilities. Security in password reset flows does not only depend on cryptographic strength (like token randomness) but also on sound state management and parameter validation. The failure to enforce a strong binding between token and identity allowed a trivial yet devastating exploit: turning a self-reset into universal account takeover.
---
title: Einrichten einer direkten Anmeldung mit Azure Active Directory B2C | Microsoft-Dokumentation
description: Erfahren Sie, wie der Anmeldename aufgefüllt oder direkt zu einem sozialen Netzwerk als Identitätsanbieter umgeleitet wird.
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 06/18/2018
ms.author: davidmu
ms.subservice: B2C
ms.openlocfilehash: 308fb8ea7f3429ce61b0872da9b1c10648b3f44b
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/28/2019
ms.locfileid: "64695316"
---
# <a name="set-up-direct-sign-in-using-azure-active-directory-b2c"></a>Einrichten einer direkten Anmeldung mit Azure Active Directory B2C

Beim Einrichten der Anmeldung für Ihre Anwendung mithilfe von Azure Active Directory (AD) B2C können Sie den Anmeldenamen auffüllen oder sich direkt bei einem bestimmten sozialen Netzwerk als Identitätsanbieter anmelden, wie z.B. bei einem Facebook-, LinkedIn- oder Microsoft-Konto. 

## <a name="prepopulate-the-sign-in-name"></a>Auffüllen des Anmeldenamens

Während einer User Journey zur Anmeldung kann eine Anwendung der vertrauenden Seite auf einen bestimmten Benutzer oder Domänennamen ausgerichtet werden. Bei der Ausrichtung auf einen Benutzer kann eine Anwendung in der Autorisierungsanforderung den Abfrageparameter `login_hint` mit dem Anmeldenamen des Benutzers angeben. Azure AD B2C füllt den Anmeldenamen automatisch auf, während der Benutzer nur das Kennwort angeben muss.

![Verwenden eines Anmeldehinweises](./media/direct-signin/login-hint.png) 

Der Benutzer kann den Wert im Textfeld für die Anmeldung ändern.

Wenn Sie eine benutzerdefinierte Richtlinie verwenden, überschreiben Sie das technische Profil von `SelfAsserted-LocalAccountSignin-Email`. Legen Sie im Abschnitt `<InputClaims>` den „DefaultValue“ des „signInName“-Anspruchs auf `{OIDC:LoginHint}` fest. Die Variable `{OIDC:LoginHint}` enthält den Wert des Parameters `login_hint`. Azure AD B2C liest den Wert des „signInName“-Anspruchs und füllt das Textfeld „signInName“ auf.

```xml
<ClaimsProvider>
  <DisplayName>Local Account</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="SelfAsserted-LocalAccountSignin-Email">
      <InputClaims>
        <!-- Add the login hint value to the sign-in names claim type -->
        <InputClaim ClaimTypeReferenceId="signInName" DefaultValue="{OIDC:LoginHint}" />
      </InputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="redirect-sign-in-to-a-social-provider"></a>Umleiten einer Anmeldung zu einem Anbieter sozialer Netzwerke

Wenn Sie die User Journey für die Anmeldung bei Ihrer Anwendung so konfiguriert haben, dass Konten für soziale Netzwerke inbegriffen sind, wie z.B. Facebook, LinkedIn oder Google, können Sie den Parameter `domain_hint` angeben. Dieser Abfrageparameter enthält einen Hinweis für Azure AD B2C zu dem sozialen Netzwerk als Identitätsanbieter, das für die Anmeldung verwendet werden sollte. Wenn in der Anwendung beispielsweise `domain_hint=facebook.com` angegeben ist,erfolgt die Anmeldung direkt auf der Anmeldeseite von Facebook.

![Verwenden eines Domänenhinweises](./media/direct-signin/domain-hint.png) 

Wenn Sie eine benutzerdefinierte Richtlinie verwenden, können Sie den Domänennamen mit dem XML-Element `<Domain>domain name</Domain>` von `<ClaimsProvider>` konfigurieren. 

```xml
<ClaimsProvider>
    <!-- Add the domain hint value to the claims provider -->
    <Domain>facebook.com</Domain>
    <DisplayName>Facebook</DisplayName>
    <TechnicalProfiles>
    ...
```



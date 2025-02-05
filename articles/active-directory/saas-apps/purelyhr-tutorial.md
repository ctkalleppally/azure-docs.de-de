---
title: 'Tutorial: Integration des einmaligen Anmeldens (Single Sign-On, SSO) von Azure Active Directory mit PurelyHR | Microsoft-Dokumentation'
description: Erfahren Sie, wie Sie das einmalige Anmelden zwischen Azure Active Directory und PurelyHR konfigurieren.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/14/2019
ms.author: jeedes
ms.openlocfilehash: 344e0c142b905558bffbf339c8e7a29c92113c23
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/29/2021
ms.locfileid: "92515184"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-purelyhr"></a>Tutorial: Integration des einmaligen Anmeldens (Single Sign-On, SSO) von Azure Active Directory mit PurelyHR

In diesem Tutorial erfahren Sie, wie Sie PurelyHR in Azure Active Directory (Azure AD) integrieren. Die Integration von PurelyHR in Azure AD ermöglicht Folgendes:

* Steuern Sie in Azure AD, wer Zugriff auf PurelyHR hat.
* Ermöglichen Sie es Ihren Benutzern, sich mit ihren Azure AD-Konten automatisch bei PurelyHR anzumelden.
* Verwalten Sie Ihre Konten zentral im Azure-Portal.

Weitere Informationen zur Integration von SaaS-Apps in Azure AD finden Sie unter [Was bedeuten Anwendungszugriff und einmaliges Anmelden mit Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>Voraussetzungen

Für die ersten Schritte benötigen Sie Folgendes:

* Ein Azure AD-Abonnement Falls Sie über kein Abonnement verfügen, können Sie ein [kostenloses Azure-Konto](https://azure.microsoft.com/free/) verwenden.
* Ein PurelyHR-Abonnement, für das einmaliges Anmelden (Single Sign-On, SSO) aktiviert ist

## <a name="scenario-description"></a>Beschreibung des Szenarios

In diesem Tutorial konfigurieren und testen Sie das einmalige Anmelden von Azure AD in einer Testumgebung.

* PurelyHR unterstützt **SP- und IDP-initiiertes** einmaliges Anmelden.
* PurelyHR unterstützt die **Just-in-Time**-Benutzerbereitstellung.

## <a name="adding-purelyhr-from-the-gallery"></a>Hinzufügen von PurelyHR über den Katalog

Zum Konfigurieren der Integration von PurelyHR in Azure AD müssen Sie PurelyHR aus dem Katalog der Liste der verwalteten SaaS-Apps hinzufügen.

1. Melden Sie sich mit einem Geschäfts-, Schul- oder Unikonto oder mit einem persönlichen Microsoft-Konto beim [Azure-Portal](https://portal.azure.com) an.
1. Wählen Sie im linken Navigationsbereich den Dienst **Azure Active Directory** aus.
1. Navigieren Sie zu **Unternehmensanwendungen**, und wählen Sie dann **Alle Anwendungen** aus.
1. Wählen Sie zum Hinzufügen einer neuen Anwendung **Neue Anwendung** aus.
1. Geben Sie im Abschnitt **Aus Katalog hinzufügen** den Suchbegriff **PurelyHR** in das Suchfeld ein.
1. Wählen Sie im Ergebnisbereich **PurelyHR** aus, und fügen Sie dann die App hinzu. Warten Sie einige Sekunden, während die App Ihrem Mandanten hinzugefügt wird.

## <a name="configure-and-test-azure-ad-single-sign-on-for-purelyhr"></a>Konfigurieren und Testen des einmaligen Anmeldens von Azure AD für PurelyHR

Konfigurieren und testen Sie das einmalige Anmelden von Azure AD mit PurelyHR mithilfe eines Testbenutzers mit dem Namen **B. Simon**. Damit einmaliges Anmelden funktioniert, muss eine Linkbeziehung zwischen einem Azure AD-Benutzer und dem entsprechenden Benutzer in PurelyHR eingerichtet werden.

Führen Sie zum Konfigurieren und Testen des einmaligen Anmeldens von Azure AD mit PurelyHR die folgenden Schritte aus:

1. **[Konfigurieren des einmaligen Anmeldens von Azure AD](#configure-azure-ad-sso)** , um Ihren Benutzern die Verwendung dieses Features zu ermöglichen.
    * **[Erstellen eines Azure AD-Testbenutzers](#create-an-azure-ad-test-user)** , um das einmalige Anmelden von Azure AD mit dem Testbenutzer B. Simon zu testen.
    * **[Zuweisen des Azure AD-Testbenutzers](#assign-the-azure-ad-test-user)** , um B. Simon die Verwendung des einmaligen Anmeldens von Azure AD zu ermöglichen.
1. **[Konfigurieren des einmaligen Anmeldens für PurelyHR](#configure-purelyhr-sso)**, um die Einstellungen für einmaliges Anmelden auf der Anwendungsseite zu konfigurieren
    * **[Erstellen eines PurelyHR-Testbenutzers](#create-purelyhr-test-user)**, um eine Entsprechung von B. Simon in PurelyHR zu erhalten, die mit ihrer Darstellung in Azure AD verknüpft ist
1. **[Testen des einmaligen Anmeldens](#test-sso)** , um zu überprüfen, ob die Konfiguration funktioniert

## <a name="configure-azure-ad-sso"></a>Konfigurieren des einmaligen Anmeldens (Single Sign-On, SSO) von Azure AD

Gehen Sie wie folgt vor, um das einmalige Anmelden von Azure AD im Azure-Portal zu aktivieren.

1. Navigieren Sie im [Azure-Portal](https://portal.azure.com/) auf der Anwendungsintegrationsseite für **PurelyHR** zum Abschnitt **Verwalten**, und wählen Sie **Einmaliges Anmelden** aus.
1. Wählen Sie auf der Seite **SSO-Methode auswählen** die Methode **SAML** aus.
1. Klicken Sie auf der Seite **Einmaliges Anmelden (SSO) mit SAML einrichten** auf das Bearbeitungs- bzw. Stiftsymbol für **Grundlegende SAML-Konfiguration**, um die Einstellungen zu bearbeiten.

   ![Bearbeiten der SAML-Basiskonfiguration](common/edit-urls.png)

1. Geben Sie im Abschnitt **Grundlegende SAML-Konfiguration** die Werte in die folgenden Felder ein, wenn Sie die Anwendung im **IDP**-initiierten Modus konfigurieren möchten:

    Geben Sie im Textfeld **Antwort-URL** eine URL im folgenden Format ein: `https://<companyID>.purelyhr.com/sso-consume`.

1. Klicken Sie auf **Zusätzliche URLs festlegen**, und führen Sie den folgenden Schritt aus, wenn Sie die Anwendung im **SP-initiierten Modus** konfigurieren möchten:

    Geben Sie im Textfeld **Anmelde-URL** eine URL im folgenden Format ein: `https://<companyID>.purelyhr.com/sso-initiate`

    > [!NOTE]
    > Hierbei handelt es sich um Beispielwerte. Die Werte müssen durch die tatsächliche Antwort-URL und die tatsächliche Anmelde-URL ersetzt werden. Wenden Sie sich an das [Supportteam von PurelyHR](https://support.purelyhr.com/), um diese Werte zu erhalten. Sie können sich auch die Muster im Abschnitt **Grundlegende SAML-Konfiguration** im Azure-Portal ansehen.

1. Navigieren Sie auf der Seite **Einmaliges Anmelden (SSO) mit SAML einrichten** im Abschnitt **SAML-Signaturzertifikat** zum Eintrag **Zertifikat (Base64)** . Wählen Sie **Herunterladen** aus, um das Zertifikat herunterzuladen, und speichern Sie es auf Ihrem Computer.

    ![Downloadlink für das Zertifikat](common/certificatebase64.png)

1. Kopieren Sie im Abschnitt **PurelyHR einrichten** die entsprechenden URLs gemäß Ihren Anforderungen.

    ![Kopieren der Konfiguration-URLs](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Erstellen eines Azure AD-Testbenutzers

In diesem Abschnitt erstellen Sie im Azure-Portal einen Testbenutzer mit dem Namen B. Simon.

1. Wählen Sie im linken Bereich des Microsoft Azure-Portals **Azure Active Directory** > **Benutzer** > **Alle Benutzer** aus.
1. Wählen Sie oben im Bildschirm die Option **Neuer Benutzer** aus.
1. Führen Sie unter den Eigenschaften für **Benutzer** die folgenden Schritte aus:
   1. Geben Sie im Feld **Name** die Zeichenfolge `B.Simon` ein.  
   1. Geben Sie im Feld **Benutzername** die Zeichenfolge username@companydomain.extension ein. Beispiel: `B.Simon@contoso.com`.
   1. Aktivieren Sie das Kontrollkästchen **Kennwort anzeigen**, und notieren Sie sich den Wert aus dem Feld **Kennwort**.
   1. Klicken Sie auf **Erstellen**.

### <a name="assign-the-azure-ad-test-user"></a>Zuweisen des Azure AD-Testbenutzers

In diesem Abschnitt ermöglichen Sie B. Simon die Verwendung des einmaligen Anmeldens von Azure, indem Sie ihr Zugriff auf PurelyHR gewähren.

1. Wählen Sie im Azure-Portal **Unternehmensanwendungen** > **Alle Anwendungen** aus.
1. Wählen Sie in der Anwendungsliste **PurelyHR** aus.
1. Navigieren Sie auf der Übersichtsseite der App zum Abschnitt **Verwalten**, und wählen Sie **Benutzer und Gruppen** aus.

   ![Link „Benutzer und Gruppen“](common/users-groups-blade.png)

1. Wählen Sie **Benutzer hinzufügen** und anschließend im Dialogfeld **Zuweisung hinzufügen** die Option **Benutzer und Gruppen** aus.

    ![Link „Benutzer hinzufügen“](common/add-assign-user.png)

1. Wählen Sie im Dialogfeld **Benutzer und Gruppen** in der Liste „Benutzer“ den Eintrag **B. Simon** aus, und klicken Sie dann unten auf dem Bildschirm auf die Schaltfläche **Auswählen**.
1. Wenn Sie einen beliebigen Rollenwert in der SAML-Assertion erwarten, wählen Sie im Dialogfeld **Rolle auswählen** die entsprechende Rolle für den Benutzer in der Liste aus, und klicken Sie dann im unteren Bildschirmbereich auf die Schaltfläche **Auswählen**.
1. Klicken Sie im Dialogfeld **Zuweisung hinzufügen** auf die Schaltfläche **Zuweisen**.

## <a name="configure-purelyhr-sso"></a>Konfigurieren des einmaligen Anmeldens für PurelyHR

1. Wenn Sie die Konfiguration in PurelyHR automatisieren möchten, müssen Sie die **Browsererweiterung „Meine Apps“ für die sichere Anmeldung** installieren, indem Sie auf **Erweiterung installieren** klicken.

    ![Erweiterung „Meine Apps“](common/install-myappssecure-extension.png)

1. Klicken Sie nach dem Hinzufügen der Erweiterung zum Browser auf **PurelyHR einrichten**, um zur Anwendung PurelyHR weitergeleitet zu werden. Geben Sie dort die Administratoranmeldeinformationen ein, um sich bei PurelyHR anzumelden. Die Browsererweiterung konfiguriert die Anwendung automatisch für Sie und automatisiert die Schritte 3 bis 5.

    ![Einrichtungskonfiguration](common/setup-sso.png)

1. Wenn Sie PurelyHR manuell einrichten möchten, melden Sie sich in einem Webbrowserfenster bei der PurelyHR-Unternehmenswebsite als Administrator an, und führen Sie die folgenden Schritte aus:

1. Öffnen Sie das **Dashboard** aus den Optionen auf der Symbolleiste, und klicken Sie auf **SSO-Einstellungen**.

1. Fügen Sie die Werte wie unten beschrieben in die Felder ein.

    ![Einmaliges Anmelden konfigurieren](./media/purelyhr-tutorial/purelyhr-dashboard-sso-settings.png)  

    a. Öffnen Sie das **Zertifikat (Base64)**, das Sie aus dem Azure-Portal heruntergeladen haben, in Editor, und kopieren Sie den Zertifikatwert. Fügen Sie den kopierten Wert in das Feld für das **X.509-Zertifikat** ein.

    b. Fügen Sie im Feld **Idp Issuer URL** (Idp-Aussteller-URL) den **Azure AD-Bezeichner** ein, den Sie aus dem Azure-Portal kopiert haben.

    c. Fügen Sie im Feld **Idp Endpoint URL** (IdP-Endpunkt-URL) die **Anmelde-URL** ein, die Sie aus dem Azure-Portal kopiert haben. 

    d. Aktivieren Sie das Kontrollkästchen **Auto-Create Users** (Benutzer automatisch erstellen), um die automatische Benutzerbereitstellung in PurelyHR zu aktivieren.

    e. Klicken Sie auf **Änderungen speichern**, um die Einstellungen zu speichern.

### <a name="create-purelyhr-test-user"></a>Erstellen eines PurelyHR-Testbenutzers

Dieser Schritt ist in der Regel nicht erforderlich, da die Anwendung die Just-in-Time-Benutzerbereitstellung unterstützt. Wenn die automatische Benutzerbereitstellung nicht aktiviert ist, kann die manuelle Erstellung von Benutzern wie unten beschrieben durchgeführt werden.

Melden Sie sich als Administrator bei Ihrer Velpic SAML-Unternehmenswebsite an, und führen Sie die folgenden Schritte aus:

1. Klicken Sie auf die Registerkarte „Verwalten“, und wechseln Sie zum Abschnitt „Benutzer“. Klicken Sie dann auf die Schaltfläche „Neu“, um Benutzer hinzuzufügen.

    ![Benutzer hinzufügen](./media/velpicsaml-tutorial/velpic_7.png)

2. Führen Sie im Dialogfeld **Neuen Benutzer erstellen** die folgenden Schritte aus.

    ![user](./media/velpicsaml-tutorial/velpic_8.png)

    a. Geben Sie im Textfeld **First Name** (Vorname) den Vornamen ein, z. B. „B“.

    b. Geben Sie im Textfeld **Last Name** (Nachname) den Nachnamen ein, z. B. Simon.

    c. Geben Sie im Textfeld **User Name** (Benutzername) den Benutzernamen „B. Simon“ ein.

    d. Geben Sie im Textfeld **Email** (E-Mail) die E-Mail-Adresse des Kontos ein, z. B. B.Simon@contoso.com.

    e. Die übrigen Informationen sind optional und können nach Bedarf ausgefüllt werden.

    f. Klicken Sie auf **SPEICHERN**.

## <a name="test-sso"></a>Testen des einmaligen Anmeldens 

In diesem Abschnitt testen Sie die Azure AD-Konfiguration für einmaliges Anmelden über den Zugriffsbereich.

Wenn Sie im Zugriffsbereich auf die Kachel „PurelyHR“ klicken, sollten Sie automatisch bei der PurelyHR-Instanz angemeldet werden, für die Sie einmaliges Anmelden eingerichtet haben. Weitere Informationen zum Zugriffsbereich finden Sie unter [Einführung in den Zugriffsbereich](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Zusätzliche Ressourcen

- [Liste mit den Tutorials zur Integration von SaaS-Apps in Azure Active Directory](./tutorial-list.md)

- [Was bedeuten Anwendungszugriff und einmaliges Anmelden mit Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

- [Was ist der bedingte Zugriff in Azure Active Directory?](../conditional-access/overview.md)

- [PurelyHR mit Azure AD ausprobieren](https://aad.portal.azure.com/)
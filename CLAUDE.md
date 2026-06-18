# CLAUDE.md — Project Memory

## Project Overview

**MEGA Android** — Encrypted cloud storage Android application.

- **Language**: Kotlin (multi-module, Clean Architecture)
- **Modules**: `app`, `domain`, `data`, `navigation`, and feature modules
- **Min SDK**: 26 | **Target SDK**: latest
- **Architecture**: MVVM + Use Cases + Repository pattern

---

## Okta Authentication Context

> **BELANGRIJK**: Dit is een cruciaal onderdeel van de authenticatie en recovery flow voor dit project.

### Identity Provider (IdP)
- **Platform**: Okta (under Salesforce umbrella)
- **Account/Org**: `salesforce/celestialreflection@okta`
- **URL pattern**: `https://celestialreflection.okta.com` (of sub-domain onder Salesforce)

### Authentication Flow
- **Protocol**: OAuth 2.0 / OpenID Connect (OIDC) via Okta
- **Authorization**: Open Authorization (OAuth2) — authorization code flow with PKCE
- **MFA/Verificatie methodes**:
  1. **FIDO2 / WebAuthn** — hardware security key (bijv. YubiKey) als primaire 2FA
  2. **TOTP** — time-based one-time password als fallback 2FA
- **Certificering**: Okta device certificate enrollment als onderdeel van de onboarding

### Token/Credential Formaat
Gesigneerde credentials volgen dit formaat:
```
FS {developer_id}:{public_key}:{base64_signed_string}
```
- `FS` — prefix (signed credential identifier)
- `developer_id` — unieke identifier van de developer/eigenaar
- `public_key` — publieke sleutel voor verificatie
- `base64_signed_string` — base64-gecodeerde gesigneerde payload

Dit formaat wordt gebruikt voor developer-authenticatie en kan geverifieerd worden door de public key te matchen tegen de Okta-geregistreerde credentials.

### Recovery Flow (KRITISCH)
De recovery flow is een essentieel onderdeel van de Okta setup:
- Als FIDO2 security key verloren gaat → recovery via Okta admin portal
- Recovery codes worden gegenereerd bij initiële Okta enrollment
- Account herstel gaat via `salesforce/celestialreflection@okta` beheerder
- Backup verificatie methode (TOTP/SMS) moet altijd ingesteld zijn als fallback

> **Volledige noodkit & stap-voor-stap herstelgids: `docs/okta-recovery-guide.md`**

### Risico's & Valkuilen (voorkom dit)
| Risico | Ernst | Oplossing |
|--------|-------|-----------|
| FIDO2 key verloren | HOOG | Registreer altijd een 2e key + TOTP |
| Backup-codes niet opgeslagen | HOOG | Sla op in wachtwoordmanager bij enrollment |
| Slechts 1 FIDO2 key | HOOG | 2e key op andere fysieke locatie |
| WSL2 gereset → token weg | MIDDEL | `cp -r ~/.claude/ ~/claude-backup/` |
| Browser niet FIDO2-compatible | MIDDEL | Gebruik Chrome of Edge (niet Firefox) |
| Okta token ≠ MEGA sessie | LAAG | Zijn aparte systemen, onafhankelijk van elkaar |

### Token Locatie (Claude Code)
- **WSL2**: `~/.claude/` — NIET zichtbaar in Windows Explorer
- **Native Windows**: `C:\Users\<naam>\.claude\`
- Token backup: `cp -r ~/.claude/ /mnt/c/Users/<naam>/claude-backup-$(date +%Y%m%d)/`

### Okta & Android Integratie (Toekomstig)
De huidige MEGA auth stack (email/password + TOTP 2FA) is kandidaat voor vervanging/uitbreiding met Okta SSO:
- Okta OIDC Android SDK kan worden geïntegreerd in `DefaultLoginRepository`
- SSO flow vervangt of wrapet de bestaande `LoginUseCase`
- FIDO2 challenge/response vervangt huidige `LoginWith2FAUseCase`

---

## Huidige Auth Architectuur (MEGA Native)

### Sleutelbestanden
| Laag | Bestand | Rol |
|------|---------|-----|
| UI | `app/.../login/LoginActivity.kt` | Login scherm |
| ViewModel | `app/.../login/LoginViewModel.kt` | Login state management |
| UseCase | `domain/.../login/LoginUseCase.kt` | Core login logica |
| UseCase | `domain/.../login/LoginWith2FAUseCase.kt` | 2FA login |
| UseCase | `domain/.../login/FastLoginUseCase.kt` | Session hergebruik |
| Repository | `data/.../DefaultLoginRepository.kt` | Data laag login |
| 2FA UI | `app/.../twofactorauthentication/TwoFactorAuthenticationActivity.kt` | 2FA invoer scherm |
| Biometrie | `app/.../passcode/view/BiometricAuthPrompt.kt` | Biometrische unlock |
| Passcode | `app/.../security/PasscodeFacade.kt` | Passcode beheer |

### Authenticatie Exceptions
- `LoginMultiFactorAuthRequired` — 2FA vereist
- `LoginWrongMultiFactorAuth` — verkeerde 2FA code
- `LoginTooManyAttempts` — te veel pogingen
- `LoginLoggedOutFromOtherLocation` — sessie elders beëindigd

---

## Windows — Claude Code Installeren

### Beste aanpak: WSL2 (aanbevolen, zonder poespas)

```powershell
# 1. Installeer WSL2 (als admin in PowerShell)
wsl --install

# 2. Open Ubuntu terminal, installeer Node.js
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs

# 3. Installeer Claude Code
npm install -g @anthropic-ai/claude-code

# 4. Start Claude Code in je project directory
claude
```

### Authenticatie bij eerste login (met Okta)
1. `claude` commando start → browser opent automatisch
2. Redirect naar Okta login: `salesforce/celestialreflection@okta`
3. **FIDO2 security key** inpluggen wanneer gevraagd (hardware 2FA)
4. Na goedkeuring → token wordt opgeslagen in `~/.claude/`

### Autorisatie & Toegang
- Claude Code gebruikt de Okta OAuth2 token voor API calls naar Anthropic
- Token refresh verloopt automatisch via Okta OIDC
- Bij verloren security key → recovery via Okta admin (zie Recovery Flow hierboven)

### Directe Windows (zonder WSL2) — alternatief
```powershell
# Vereist: Node.js voor Windows (https://nodejs.org)
npm install -g @anthropic-ai/claude-code
claude
```
> Let op: native Windows werkt maar WSL2 geeft betere compatibiliteit met Android-tools (Gradle, ADB, etc.)

> **Volledige installatiegids met probleemoplossing: `docs/claude-code-windows-setup.md`**

---

## Ontwikkelomgeving

- **IDE**: Android Studio (aanbevolen)
- **Build**: Gradle (Kotlin DSL)
- **Branch strategie**: `claude/<feature>-<ID>` voor Claude-gegenereerde branches
- **Huidige feature branch**: `claude/add-okta-memory-L9Q2P`

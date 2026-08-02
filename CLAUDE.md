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

## Genspark AI — Rol in het Ecosysteem

- **Tool**: Genspark AI (`genspark.ai`) — gebruikt voor uitlijnen, schemata, en agent-workflows
- **Genspark Terminal**: wordt geïnstalleerd op Windows desktop (gepland) — lokale agent runner
- **Bestanden**: Genspark exporteert naar `genspark.ai/api/files/s/<id>` — staan NIET lokaal in Notion
- **Notion integratie**: 4 "Genspark Integration" pagina's aanwezig, meest recente child-pagina's zijn leeg
- **Verband met Hermes**: Genspark terminal + Hermes (Cloudflare Worker) + fgm-memory (D1) vormen samen de agent-stack
- **Caveka dossier**: Genspark heeft `Caveka_Payment_Structure.md` gegenereerd met salarisoverzicht en back-pay claim

### Genspark Terminal Installatie (gepland)
Na installatie op Windows koppelt de terminal aan de bestaande agent-stack:
```
Genspark Terminal (Windows)
    ↕
Hermes routing agent (Cloudflare Worker)
    ↕
fgm-memory D1 database (WEUR/Amsterdam)
    ↕
Notion / Claude Code context
```

---

## Communicatie-ecosysteem

Meerdere platforms, elk met eigen kanalen — Claude reageert soms op PC terwijl gesprek op telefoon begon:

| Platform | Kanalen | Opmerking |
|----------|---------|-----------|
| Telefoon (SIM) | Bellen, SMS | Primair — maar helft gemist als niet bij telefoon |
| WhatsApp | Telefoon + Desktop | 2 kanalen, gesprekken syncen |
| Telegram | Telefoon + Desktop | 2 kanalen, beter voor automation/bots |
| Windows PC | Bluetooth / schermverlenging van telefoon | Berichten via PC zonder apart oproepen |
| Claude Code | Desktop (WSL2) | Reageert op PC-sessie, niet op telefoon-sessie |

**Risico**: reacties op verkeerd platform → Claude ziet bericht niet, of reageert te laat.

---

## Identiteiten & Accounts (NIET wijzigen zonder expliciete instructie)

> Alles hieronder is bewust opgezet. Niets afraden of als "verdacht" bestempelen tenzij de gebruiker dat zelf aangeeft.

### Hoofd e-mailadressen
| Adres | Rol |
|-------|-----|
| `djthamo@gmail.com` | Primair Google account — meeste developer ingangen |
| `celestialreflection@outlook.com` | Enterprise hoofdaccount — GitHub repos, Azure DevOps, NordPass |
| `easio@proton.me` | ProtonMail — via CanaryMail.io Lifetime Access gekoppeld |

### Gekoppelde systemen aan `celestialreflection@outlook.com`
- NordPass / NordVPN (enterprise)
- GitHub repositories
- Azure DevOps
- Okta (`salesforce/celestialreflection@okta`)

### CanaryMail notitie
`easio@proton.me` had wachtwoord van `djthamo@gmail.com` opgeslagen via CanaryMail integratie — bewust of via synchronisatie, nog te beoordelen door gebruiker.

---

## Wallet & Web3 Infrastructuur

> Deze infrastructuur is bewust gebouwd over ~2,5 jaar. Niets verwijderen zonder expliciete instructie.

### Hoofd wallets
| Wallet | Adres / ENS | Rol |
|--------|-------------|-----|
| FU Wallet | `0x7BFEe91193d9Df2Ac0bFe90191D40F23c773C060` / `7bfee.eth` | Primair — Rabby browser extensie |
| Nord Wallet | `0xA7cF...BFBF` | Gekoppeld aan nordaccount.com + Rainbow |
| Google Pay Wallet | `0xbbbb...ffcb` | Gekoppeld aan payments.google.com |
| BAYC Wallet | `0x3e87...d90c` | Bored Ape Yacht Club holdings |

### Wallet relay-verbindingen (uit foto 25-03-2026)
| Site | Wallet |
|------|--------|
| `coinstats.app` | `7bfee.eth` |
| `hyperevmscan.io` | `7bfee.eth` |
| `nordaccount.com` | `0xA7cF...BFBF` |
| `payments.google.com` | `0xbbbb...ffcb` |
| `rainbowdotme.typeform.com` | `0xA7cF...BFBF` |
| Bored Ape Yacht Club | `0x3e87...d90c` |

### Wallet tools & Snaps
- **Rainbow** — primaire wallet interface, beheert relay-verbindingen
- **MetaMask** — basis wallet
- **Purple Dragon** — MetaMask Snap (ZK / identiteit laag)
- **Ninja Wallet** — MetaMask Snap
- **web3auth.io** — Web3 login integratie
- **CoinStats** — portfolio tracker, gekoppeld aan `7bfee.eth`

### 84 relay-foto's
Gebruiker heeft 84 foto's gemaakt van alle relay-verbindingen (bewust en onbewust opgeslagen). Nog niet volledig in memory verwerkt — **prioriteit om te ontvangen en te documenteren**.

---

## Ontwikkelomgeving

- **IDE**: Android Studio (aanbevolen)
- **Build**: Gradle (Kotlin DSL)
- **Branch strategie**: `claude/<feature>-<ID>` voor Claude-gegenereerde branches
- **Huidige feature branch**: `claude/add-okta-memory-L9Q2P`

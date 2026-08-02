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

## LOOP Consolidatie Plan (8 april 2026)

> **Doel**: Alle infrastructuur inkapselen naar 1 Business ID (KVK Nederland)
> **Startpunt**: Desktop via Ethernet (directe verbinding) = MOEDERSCHIP

### Apparaat Hiërarchie
```
DESKTOP (Ethernet - MOEDERSCHIP)
├── Brave Browser [INGEKAPSELD] → wachtwoorden + identiteiten
├── Firefox → Nightly Wallet + Mozilla Relay (MASTER INDEX)
└── sync ↓
LAPTOP (WiFi)
├── Opera GX / Opera normaal
└── Yandex Browser [KRITIEK] → wachtwoordyd1@yandex.ru (apart houden)
    └── sync ↓
TELEFOON (WiFi/Bluetooth)
├── Claude app (djthamo@gmail.com)
├── Google Files (blauw mapje)
├── Files (geel mapje)
└── TeraBox app
```

### Doel Eindstructuur
```
1 BUSINESS ID (KVK Nederland)
├── Email: djthamo@gmail.com (PRIMAIR)
├── Banking: ABN AMRO zakelijk / Revolut Business / PayPal (onechapterahead.com)
├── Crypto: Nederlandse exchanges (compliant) + D'CENT hardware wallet
└── Websites: celestialreflection.com / tessle.nl / tesla-import.nl

APART HOUDEN (Privacy — NIET samenvoegen):
├── Russische infra (Yandex, VK) — bewust gescheiden
├── Agent wallets (tot geverifieerd)
└── Proton (easio@proton.me)
```

---

## Identiteiten & Accounts (NIET wijzigen zonder expliciete instructie)

> Alles hieronder is bewust opgezet. Niets afraden of als "verdacht" bestempelen tenzij de gebruiker dat zelf aangeeft.
> **Yandex en VK zijn BEWUST apart gehouden — NIET adviseren te verwijderen.**

### E-mail identiteiten
| Adres | Functie | Status |
|-------|---------|--------|
| `djthamo@gmail.com` | **PRIMAIR** — consolideer hier naartoe | Actief |
| `djpeterstreets@gmail.com` | Persoonlijk — nooit zakelijk gebruikt | Persoonlijk |
| `djpeterstreets@hotmail.com` | Variant van persoonlijk adres | Persoonlijk |
| `djpeterstreets@outlook.com` | Variant van persoonlijk adres | Persoonlijk |
| `celestialreflection@outlook.com` | Microsoft/Okta SSO, Azure DevOps, NordPass | Forward → primair |
| `celestialreflection@hotmail.com` | OneDrive / Revolut | Forward → primair |
| `info@celestialreflection.com` | Enige echte zakelijke e-mail (website), maar nooit volledig opgezet | Forward → primair |
| `easio@proton.me` | Privacy/Proton — CanaryMail Lifetime Access | Apart houden |
| `fgmvanstraaten@yandex.ru` | Russische infra / Yandex Disk backup | Apart houden |
| `pvanstraaten@vk.com` | VKontakte (Russisch social network) | Apart houden |
| `wachtwoordyd1@yandex.ru` | Yandex browser sync op laptop | Apart houden |
| `peterstreets@live.nl` | Onbekende herkomst — niet bevestigd of gekoppeld | Onderzoeken |

### GitHub & SSH
- **GitHub gebruikersnaam**: `PeterStreets` (geen e-mail, alleen username)
- **SSH sleutel**: RSA 4096-bit aangemaakt met comment `celestialreflection@outlook.com`
- **Salesforce instantie**: `velocity-customer-4341.my.salesforce.com`

### Keybase
- Account met `djpeterstreets@proton.me` verscheen in configuratie ~2024
- Nooit bewust aangemaakt door gebruiker
- Functie: waarschijnlijk backup/recovery voor wallet of wachtwoordmanager
- Niet kritiek maar wel relevant als hersteloptie

### Mozilla Relay (Master Index — NIET wijzigen)
| Relay adres | Gekoppeld aan |
|-------------|--------------|
| `pq9yOcoo5@mozmail.com` | Crypto.com |
| `inuo59d19@mozmail.com` | "Innuo 59" |
| `makwim34r@mozmail.com` | ? (nog te identificeren) |
| `et4g8iSb8@mozmail.com` | MetaMask WebAuth |
| `nljl6z2tq@mozmail.com` | Nieuw ontdekt |

### CanaryMail notitie
`easio@proton.me` had wachtwoord van `djthamo@gmail.com` opgeslagen via CanaryMail integratie — bewust of via synchronisatie, nog te beoordelen door gebruiker.

---

## Wallet & Web3 Infrastructuur

> Bewust gebouwd over ~2,5 jaar. Niets verwijderen zonder expliciete instructie.

### ENS Domeinen (Ethereum)
| ENS | Adres | Waarde |
|-----|-------|--------|
| `7bfee.eth` | `0x7BFEe91193d9Df2Ac0bFe90191D40F23c773C060` | ~$27M+ |
| `chickengenius.eth` | `0xeb2Eb5C681562500C368914761bB8F1208d56AcD01` | ~$1.71M (10 chains) |
| `titanbuilder.eth` | Te verifiëren | ? |
| `robots.eth` | Te verifiëren | ? |

### Unstoppable Domains (via djthamo@gmail.com)
.crypto / .x / .wallet / .nft domeinen — exact welke nog te verifiëren

### Hardware & Software Wallets
| Wallet | Type | Status |
|--------|------|--------|
| D'CENT | Hardware (70+ blockchains) | Actief |
| Rabby | Software | Confirmed connected |
| MetaMask Flask | Software + Snaps | Actief |
| Rainbow | Software | Relay-beheer |
| Phantom | Software | Actief |
| Anchor | WAX/EOS | Actief |
| Solflare | Software | 2FA actief |
| Nightly | Root HD wallet | Actief |
| OKX Web3 | Software | 450+ sub-accounts |

### Wallet relay-verbindingen (foto 25-03-2026)
| Site | Wallet |
|------|--------|
| `coinstats.app` | `7bfee.eth` |
| `hyperevmscan.io` | `7bfee.eth` |
| `nordaccount.com` | `0xA7cF...BFBF` |
| `payments.google.com` | `0xbbbb...ffcb` |
| `rainbowdotme.typeform.com` | `0xA7cF...BFBF` |
| Bored Ape Yacht Club | `0x3e87...d90c` |

### Wallet tools & Snaps
- **Purple Dragon** — MetaMask Snap (ZK / identiteit laag)
- **Ninja Wallet** — MetaMask Snap
- **web3auth.io** — Web3 login integratie
- **CoinStats** — portfolio tracker, gekoppeld aan `7bfee.eth`

### Cloud Storage (~10TB totaal)
| Service | Login | Status |
|---------|-------|--------|
| Google Drive | djthamo@gmail.com | Primair |
| TeraBox | djthamo@gmail.com | Betaald 4 jaar ✅ |
| TeraBox | info@celestialreflection.com | **VERWIJDEREN** ❌ |
| Yandex Disk | fgmvanstraaten@yandex.ru | Russische backup (apart) |
| OneDrive | celestialreflection@hotmail.com | Microsoft/Revolut link |

### 84 relay-foto's
Gebruiker heeft 84 foto's van alle relay-verbindingen (bewust/onbewust). Deels verwerkt — meer foto's volgen.

### Openstaande acties (uit consolidatieplan)
- [ ] Brave Browser: exporteer wachtwoordenlijst
- [ ] Firefox Nightly wallet config checken
- [ ] TeraBox: verwijder info@ koppeling, behoud djthamo@ abonnement
- [ ] chickengenius.eth eigendom verifiëren (ENS app)
- [ ] titanbuilder.eth + robots.eth verifiëren
- [ ] Unstoppable Domains inloggen — exacte domeinen noteren
- [ ] celestialreflection.com DNS/hosting status checken
- [ ] onechapterahead.com — actief?
- [ ] hyperventure.xyz — eigenaar?
- [ ] KVK: Huddle B.V. status checken

### Wat NIET synchroniseren
| Item | Reden |
|------|-------|
| TeraBox China versie | Apart houden |
| Russische accounts (Yandex, VK) | Privacy, los van NL activiteiten |
| Yandex browser data | Alleen op laptop houden |

---

## Ontwikkelomgeving

- **IDE**: Android Studio (aanbevolen)
- **Build**: Gradle (Kotlin DSL)
- **Branch strategie**: `claude/<feature>-<ID>` voor Claude-gegenereerde branches
- **Huidige feature branch**: `claude/add-okta-memory-L9Q2P`

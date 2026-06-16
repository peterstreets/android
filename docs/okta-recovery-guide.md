# Okta Recovery Guide — Noodplan bij Authenticatieproblemen

> **Bewaar dit document ook offline (USB, afdruk, wachtwoordmanager).**
> Dit is je reddingslijn als je nergens meer bij kunt.

---

## 1. Situatie: FIDO2 Security Key kwijt of kapot

### Stap 1 — Controleer of je een backup-methode hebt
- Open `https://celestialreflection.okta.com`
- Probeer in te loggen met email + wachtwoord
- Kies bij 2FA-prompt: **"Use a different authenticator"**
- Opties in volgorde:
  1. Tweede FIDO2 key (als ingesteld)
  2. TOTP app (Google Authenticator / Authy)
  3. Recovery code (backup codes)

### Stap 2 — Gebruik een backup recovery code
Recovery codes zijn gegenereerd bij de initiële Okta enrollment.
- Locatie: je wachtwoordmanager, beveiligde USB, of afgedrukte lijst
- Elke code is eenmalig bruikbaar
- Na gebruik: direct een nieuwe reserve-code aanmaken

### Stap 3 — Neem contact op met Okta Admin
Als ALLE methodes falen:
- **Org**: `salesforce/celestialreflection@okta`
- **Actie**: admin kan MFA tijdelijk resetten zodat je opnieuw kunt enrollen
- Bereid je voor: bewijs van identiteit kan gevraagd worden

### Stap 4 — Na herstel: direct preventief handelen
1. Registreer een **tweede FIDO2 key** (zie sectie 4)
2. Verifieer dat TOTP nog werkt
3. Genereer nieuwe backup recovery codes en sla ze op

---

## 2. Situatie: Okta account geblokkeerd / te veel pogingen

- Wacht 15-30 minuten (automatische unlock bij de meeste Okta-configuraties)
- Als dat niet helpt: neem contact op met `salesforce/celestialreflection@okta` admin
- Gebruik NOOIT een script of bot om opnieuw te proberen — dit verlengt de blokkade

---

## 3. Situatie: Claude Code token verlopen of weg (WSL2)

Claude Code slaat zijn authenticatietoken op in:
```
~/.claude/           ← WSL2 home directory
~/.claude/.credentials
```

**Tokens zijn NIET zichtbaar vanuit Windows Explorer** — dit is de WSL2 home, niet `C:\Users\<naam>`.

### Token kwijt / WSL2 gereset
1. Start Claude Code opnieuw: `claude`
2. Browser opent automatisch → redirect naar Okta login
3. Log in met FIDO2 key
4. Token wordt automatisch opnieuw opgeslagen

### Token backup (proactief)
```bash
# Maak backup van Claude tokens (uitvoeren in WSL2 terminal)
cp -r ~/.claude/ ~/claude-token-backup-$(date +%Y%m%d)/
```
> Bewaar dit op een externe locatie (OneDrive, externe schijf). **Deel dit NOOIT.**

---

## 4. Proactieve Setup: Tweede FIDO2 Key registreren

**Dit is de belangrijkste preventieve maatregel.**

1. Ga naar `https://celestialreflection.okta.com`
2. Instellingen → Security → Extra Authenticator toevoegen
3. Kies: **Security Key or Biometric** (FIDO2/WebAuthn)
4. Volg de browser-prompt
5. Bewaar de tweede key op een andere fysieke locatie (thuis vs. kantoor)

---

## 5. Proactieve Setup: TOTP als Fallback

1. Ga naar `https://celestialreflection.okta.com`
2. Instellingen → Security → Authenticator toevoegen
3. Kies: **Google Authenticator** of **Authy**
4. Scan de QR-code met je authenticator-app
5. Sla de **secret key** ook op in je wachtwoordmanager (voor herstel van de TOTP app)

---

## 6. Checklist Noodkit (bewaar dit klaar)

| Item | Opgeslagen? | Locatie |
|------|-------------|---------|
| Okta recovery codes (backup codes) | [ ] | Wachtwoordmanager |
| TOTP secret key | [ ] | Wachtwoordmanager |
| Tweede FIDO2 key geregistreerd | [ ] | Andere fysieke locatie |
| Okta admin contactgegevens | [ ] | `salesforce/celestialreflection@okta` |
| Claude token backup | [ ] | Externe schijf / OneDrive |
| Dit document offline | [ ] | USB / afdruk |

---

## 7. MEGA Android — Sessie vs. Okta Token

> Belangrijk onderscheid om verwarring te voorkomen:

| | MEGA Sessie | Okta Token |
|---|---|---|
| **Waarvoor** | Toegang tot MEGA cloud storage | Toegang tot Claude Code / developer tools |
| **Opgeslagen in** | Android `CredentialsPreferencesDataStore` | `~/.claude/` (WSL2) |
| **Herstellen** | Via MEGA recovery key (link + sleutel) | Via Okta login opnieuw |
| **Verlopen** | Bij uitloggen of sessie elders | Automatisch, vernieuwd via OIDC |

Als de MEGA app je uitlogt (`LoginLoggedOutFromOtherLocation`): dit staat los van je Okta/Claude status.

---

## 8. Contacten & Links

| Resource | URL / Info |
|----------|-----------|
| Okta portal | `https://celestialreflection.okta.com` |
| Okta admin org | `salesforce/celestialreflection@okta` |
| Okta support | `https://support.okta.com` |
| FIDO2 info | `https://fidoalliance.org/fido2/` |
| Claude Code docs | `https://docs.anthropic.com/claude-code` |

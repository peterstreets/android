# Claude Code — Windows Installatie & Setup

> Doel: Claude Code installeren op Windows desktop met Okta-authenticatie,
> zodat je toegang hebt tot lokale bestanden, repositories en code.

---

## Aanbevolen Aanpak: WSL2 (zonder poespas)

WSL2 geeft de beste compatibiliteit met Android-tools (Gradle, ADB, Git, etc.).

### Stap 1 — WSL2 Installeren

Open **PowerShell als Administrator**:

```powershell
wsl --install
```

Herstart je computer. Ubuntu wordt automatisch geïnstalleerd.

### Stap 2 — Node.js Installeren (in WSL2/Ubuntu terminal)

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
node --version   # controleer: moet v18+ zijn
```

### Stap 3 — Claude Code Installeren

```bash
npm install -g @anthropic-ai/claude-code
claude --version  # controleer installatie
```

### Stap 4 — Eerste Login (Okta)

```bash
claude
```

1. Browser opent automatisch
2. Redirect naar Okta: `https://celestialreflection.okta.com`
3. Log in met je email + wachtwoord
4. Plug je **FIDO2 security key** in wanneer gevraagd
5. Druk op de knop van de key (of raak de sensor aan)
6. Browser bevestigt → Claude Code is geautoriseerd
7. Token wordt opgeslagen in `~/.claude/`

---

## Alternatief: Native Windows (zonder WSL2)

```powershell
# Vereiste: installeer Node.js van https://nodejs.org (LTS versie)
npm install -g @anthropic-ai/claude-code
claude
```

**Voordelen**: simpeler, geen WSL2 nodig
**Nadelen**: Gradle/ADB werkt minder goed, pad-problemen met Android SDK

---

## Je Android Project openen met Claude Code

### Via WSL2

```bash
# Navigeer naar je project (Windows schijf is gemount als /mnt/c/)
cd /mnt/c/Users/<jouw-naam>/AndroidStudioProjects/android
claude
```

Of zet je project direct in de WSL2 home voor snelheid:
```bash
cd ~/projects/android
claude
```

### Via native Windows

```powershell
cd C:\Users\<jouw-naam>\AndroidStudioProjects\android
claude
```

---

## Token Locatie & Backup

### WSL2
```
~/.claude/                 # token directory
~/.claude/.credentials     # Okta OAuth token
```

**Zichtbaar in Windows Explorer?** Nee. Gebruik de WSL2 terminal om erbij te komen:
```bash
ls -la ~/.claude/
```

**Backup maken** (aanbevolen, doe dit na eerste login):
```bash
cp -r ~/.claude/ /mnt/c/Users/<jouw-naam>/claude-backup-$(date +%Y%m%d)/
```

### Native Windows
```
C:\Users\<jouw-naam>\.claude\
```

---

## Toegang tot Bestaande Repositories

Claude Code kan alle bestanden lezen die toegankelijk zijn via het bestandssysteem.

### WSL2 — toegang tot Windows bestanden
```bash
ls /mnt/c/Users/<jouw-naam>/       # jouw Windows home
ls /mnt/c/                          # hele C-schijf
```

### Repositories klonen in WSL2
```bash
cd ~/projects
git clone <repo-url>
cd <project-naam>
claude
```

Claude kan dan alle bestanden in de repository lezen, doorzoeken en aanpassen.

---

## Probleemoplossing

### Browser opent niet bij `claude`
```bash
# Kopieer de URL handmatig uit de terminal output en open in Chrome/Edge
```

### FIDO2 key wordt niet herkend
- Gebruik **Chrome of Edge** (niet Firefox op Windows — soms problemen met WebAuthn)
- Zorg dat de key in een USB-poort zit die direct op het moederbord zit (niet via USB-hub)
- Probeer een andere USB-poort

### Token verlopen
```bash
claude  # start opnieuw → automatische re-authenticatie via Okta
```

### WSL2 start niet
```powershell
# Als admin in PowerShell:
wsl --update
wsl --shutdown
wsl
```

### Node.js niet gevonden na WSL2 herstart
```bash
node --version
# Als niet gevonden: installeer opnieuw via nodesource (zie Stap 2)
```

---

## Verificatie: Alles werkt?

```bash
# In WSL2 terminal:
claude --version          # Claude Code versie zichtbaar
node --version            # Node.js v18+
git --version             # Git beschikbaar
claude                    # Start sessie zonder login-prompt = token geldig
```

---

## Volgende Stap: Recovery Setup

Als de installatie werkt, zorg dan direct voor je backup-scenario:
→ Zie `docs/okta-recovery-guide.md` voor de volledige noodkit-checklist.

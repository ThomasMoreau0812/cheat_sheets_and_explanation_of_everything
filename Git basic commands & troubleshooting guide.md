# 🐙 Git Cheat Sheet & Stappenplan: Veilig Werken met GitHub

Een overzichtelijke handleiding voor de dagelijkse Git-workflow, inclusief oplossingen voor veelvoorkomende fouten en tips om dataverlies te voorkomen.

---

## 🛡️ De 3 Gouden Regels van Git

1. **Gebruik NOOIT `--force` op `main` (of `master`)**  
   Als Git een push weigert (`rejected`), is dat een veiligheidsgordel die aangeeft dat er bestanden op GitHub staan die jij lokaal nog niet hebt. Gebruik `git pull` in plaats van `--force`.
2. **Bestaat een repository al op GitHub? Altijd eerst `git clone`**  
   Maak geen handmatige map aan met `git init` als de repo al op GitHub bestaat. `git clone` stelt alles (URL, branch, origin) meteen 100% goed in.
3. **Altijd `git pull` vóórdat je begint te werken**  
   Zorg dat je lokale map gelijkloopt met GitHub voordat je nieuwe wijzigingen toevoegt.

---

## ⚙️ Eenmalige Instelling (Aanbevolen)

Voer dit één keer uit op je machine zodat nieuwe branches altijd `main` heten (net zoals op GitHub) in plaats van het verouderde `master`:

```bash
git config --global init.defaultBranch main
```

---

## 📋 Stappenplan 1: Bestanden Toevoegen aan een Bestaande Repo (Dagelijkse Workflow)

Dit is de veiligste en meest gebruikte manier om documenten toe te voegen of te bewerken:

```bash
# 1. Navigeer naar je lokale map
cd ~/Documents/github

# 2. Haal de laatste versie op van GitHub
git pull

# 3. Voeg je nieuwe of bewerkte bestand(en) toe
#    (kopieer of bewerk je bestanden in deze map)

# 4. Controleer wat er gewijzigd is
git status

# 5. Voeg de gewenste bestanden toe aan de staging-area
git add .
# Of specifiek één bestand:
# git add "mijn_bestand.md"

# 6. Maak een commit met een duidelijke beschrijving
git commit -m "docs: voeg nieuwe cheat sheets toe"

# 7. Push veilig naar GitHub
git push
```

---

## 📋 Stappenplan 2: Een Compleet Nieuwe Repo Beginnen

Wanneer je een gloednieuw project wilt maken en op GitHub wilt zetten:

### Manier A: Starten via GitHub (Aanbevolen & Makkelijkst)
1. Ga naar [github.com/new](https://github.com/new).
2. Geef je repo een naam en vink eventueel *Add a README file* aan.
3. Klik op **Create repository**.
4. Open je terminal en kloon hem direct naar je computer:
   ```bash
   cd ~/Documents
   git clone https://github.com/<gebruikersnaam>/<nieuwe-repo>.git
   cd <nieuwe-repo>
   ```
5. Voeg je bestanden toe, commit en push:
   ```bash
   git add .
   git commit -m "Initial commit"
   git push
   ```

### Manier B: Starten vanuit een Bestaande Lokale Map
```bash
cd ~/pad/naar/jouw/map

# Initialiseer Git
git init
git branch -M main

# Voeg bestanden toe en commit
git add .
git commit -m "Initial commit"

# Koppel aan GitHub (maak eerst een lege repo aan op GitHub)
git remote add origin https://github.com/<gebruikersnaam>/<repo-naam>.git

# Eerste keer pushen
git push -u origin main
```

---

## 🚨 Troubleshooting: Veelvoorkomende Foutmeldingen & Oplossingen

### 1. `error: src refspec main does not match any`
* **Oorzaak:** Je hebt nog geen commit gemaakt, óf je branch heet lokaal `master` in plaats van `main`.
* **Oplossing:**
  ```bash
  # Controleer de branchnaam
  git branch
  
  # Hernoem naar main en push
  git branch -M main
  git push -u origin main
  ```

---

### 2. `fatal: 'origin' does not appear to be a git repository`
* **Oorzaak:** Git weet niet naar welke GitHub-link hij moet sturen (`origin` ontbreekt).
* **Oplossing:**
  ```bash
  # Controleer of origin al bestaat
  git remote -v
  
  # Koppel je GitHub link
  git remote add origin https://github.com/<gebruikersnaam>/<repo>.git
  ```

---

### 3. `Updates were rejected because the remote contains work that you do not have locally`
* **Oorzaak:** Er staan wijzigingen of bestanden op GitHub die jij nog niet op je computer hebt.
* **Oplossing:**
  ```bash
  # Haal de wijzigingen binnen en voeg samen
  git pull origin main --rebase
  
  # Push opnieuw
  git push origin main
  ```

---

### 4. `bash: /pad/naar/bestand.md: Permission denied`
* **Oorzaak:** Je hebt per ongeluk alleen het bestandspad in de terminal geplakt en op Enter gedrukt. Bash probeert het tekstbestand dan als programma uit te voeren.
* **Oplossing:** Zet altijd een commando ervoor, zoals `cp` (kopiëren), `cat` (bekijken) of `nano` (bewerken):
  ```bash
  cp /pad/naar/bronbestand.md ~/Documents/github/
  ```

---

### 5. Bestanden per ongeluk overschreven met `--force`? Geen paniek!
Git gooit commits op GitHub nooit meteen weg. Ze blijven via hun commit-hash bewaard:
1. Zoek de oude commit-hash op via GitHub Events/Commits of je lokale `git reflog`.
2. Haal de oude commit terug en merge hem:
   ```bash
   git fetch origin <commit-hash>
   git merge FETCH_HEAD --allow-unrelated-histories
   git push origin main
   ```

---

## 📌 Samenvatting Snelle Commando's

| Commando | Wat doet het? |
| :--- | :--- |
| `git status` | Bekijk gewijzigde en niet-toegevoegde bestanden |
| `git pull` | Download de nieuwste versie vanaf GitHub |
| `git add .` | Zet alle gewijzigde bestanden klaar voor commit |
| `git commit -m "boodschap"` | Leg de wijzigingen lokaal vast met een beschrijving |
| `git push` | Upload je commits naar GitHub |
| `git branch -a` | Toon alle lokale en online branches |
| `git log --oneline` | Bekijk compacte geschiedenis van recente commits |

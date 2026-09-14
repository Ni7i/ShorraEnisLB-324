# LB 324

## Aufgabe 2
Die Datei `.pre-commit-config.yaml` definiert zwei Hooks:

| Hook | Stage | Wirkung |
|---|---|---|
| `black` | `pre-commit` | formatiert bei jedem `git commit` die geänderten Python-Dateien |
| `pytest` | `pre-push` | führt bei jedem `git push` alle Tests aus; schlägt ein Test fehl, wird der push abgebrochen |

Einmalig nach dem Klonen im Projektverzeichnis ausführen:

```
pip install -r requirements.txt
pip install pre-commit
pre-commit install
```

`pre-commit install` installiert dank `default_install_hook_types` beide Hooks (`.git/hooks/pre-commit` und `.git/hooks/pre-push`). Ohne diese Einstellung wären beide Typen einzeln zu installieren:

```
pre-commit install --hook-type pre-commit
pre-commit install --hook-type pre-push
```

Formatiert `black` beim commit eine Datei um, bricht der commit ab. Die umformatierten Dateien erneut mit `git add` hinzufügen und nochmals `git commit` ausführen.

Hooks manuell auf alle Dateien anwenden:

```
pre-commit run --all-files
pre-commit run --all-files --hook-stage pre-push
```

## Aufgabe 4
Laufende Applikation: https://shorraenis-lb324.azurewebsites.net

### Passwort aus der `.env` auf Azure übertragen
Die `.env` steht in der `.gitignore` und gelangt nie auf GitHub und damit auch nicht auf Azure. `load_dotenv()` findet auf Azure also keine Datei, `os.getenv("PASSWORD")` liest den Wert stattdessen aus den Umgebungsvariablen. Diese werden auf Azure als App-Einstellung gesetzt. Für die Auslieferung lautet das Passwort gleich wie der GitHub-Benutzername (`Ni7i`).

Im Azure-Portal:
1. *App Services* → `shorraenis-lb324` öffnen
2. *Settings* → *Environment variables* → Reiter *App settings* → *+ Add*
3. Name `PASSWORD`, Value `Ni7i` → *Apply*, danach unten nochmals *Apply* und *Confirm* (die App wird neu gestartet)

Alternativ mit der Azure CLI:

```
az webapp config appsettings set -g rg-lb324 -n shorraenis-lb324 --settings PASSWORD="Ni7i"
```

### Automatische Auslieferung
`.github/workflows/deploy.yml` läuft bei jedem push auf `main`, also bei jedem merge in den `main`-Ast. Der Workflow installiert die Abhängigkeiten, führt `pytest` aus und liefert nur bei erfolgreichen Tests mit `azure/webapps-deploy` auf Azure aus.

Die Anmeldung von GitHub bei Azure läuft über OpenID Connect ohne gespeichertes Passwort:
- Verwaltete Identität `id-lb324-github` mit der Rolle *Website Contributor* auf der Web App
- Federated Credential für `repo:Ni7i/ShorraEnisLB-324:ref:refs/heads/main`, dadurch darf nur der `main`-Ast ausliefern
- GitHub-Secrets `AZURE_CLIENT_ID`, `AZURE_TENANT_ID` und `AZURE_SUBSCRIPTION_ID` (*Settings* → *Secrets and variables* → *Actions*)

Weitere Einstellungen der Web App:
- Laufzeit `PYTHON|3.12`
- Startbefehl `gunicorn --bind=0.0.0.0 --timeout 600 app:app`
- `SCM_DO_BUILD_DURING_DEPLOYMENT=true`, damit Azure beim Ausliefern die `requirements.txt` installiert
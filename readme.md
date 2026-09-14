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
Erklären Sie hier, wie Sie das Passwort aus Ihrer lokalen `.env` auf Azure übertragen.
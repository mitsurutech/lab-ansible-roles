# Ansible-Rollen-Lab für den Unterricht: eine Rolle, Handler und Vault auf einer VM

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md) | [Deutsch](README.de.md)

[![check](https://github.com/mitsurutech/lab-ansible-roles/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-ansible-roles/actions/workflows/check.yml)

Ein fertiges **Ansible-Lab zu Rollen** für den Unterricht in Infrastructure as Code, von der [MitsuruTech Learning Platform](https://mitsurutech.co.uk/de/learning), auf der Schulen und Hochschulen jedem Lernenden eigene Maschinen und ein Terminal im Browser geben. Es ist die zweite Stufe nach [lab-ansible-basics](https://github.com/mitsurutech/lab-ansible-basics). Jede Person bekommt eine Ubuntu-**VM** mit installiertem **ansible-core** und einem kleinen Projekt und baut eine **Rolle** von Grund auf: ihre Variablen, eine Jinja2-Vorlage, Tasks mit Tags, einen **Handler**, der nur einmal läuft, und ein Token, das in **Ansible Vault** liegt. Die Maschine selbst ist das Ziel, es wird also nichts weiter gebraucht und auf dem Rechner der Lernenden wird nichts installiert.

## Was die Lernenden tun

Das Lab hinterlässt ein Projekt in `~/roles-lab` und eine `README.md` im Home-Verzeichnis, die in sieben Schritten sagt, was zu tun ist, und keine Lösung verrät:

```
ansible.cfg          zeigt auf inventory.ini und den Rollenordner
inventory.ini        zwei Hosts, web1 und spare1, beide diese Maschine
site.yml             ein Play auf web, das die Rolle appconf nutzt
vault-password.txt   das Vault-Passwort für dieses Lab
roles/               hier kommt die Rolle hinein
```

1. Das Playbook starten und sehen, wie es stehen bleibt, weil die Rolle `appconf` noch nicht existiert.
2. Die Rolle mit `ansible-galaxy role init` anlegen und ihre `meta/main.yml` ausfüllen.
3. Variablen in `defaults/main.yml` und `vars/main.yml` setzen. `site.yml` setzt `app_env` und `app_dir` schon im Play, die Lernenden müssen also herausfinden, welche Variable gewinnt.
4. Die Tasks schreiben, mit dem Tag `config` oder `user`: einen Ordner, eine Datei aus einer Vorlage mit Schleife und Bedingung, eine Datei je Funktion, eine Token-Datei, einen Systembenutzer und zwei kleine Dateien, von denen eine nur entsteht, wenn der Benutzer wirklich angelegt wurde.
5. Einen Handler, den drei Tasks benachrichtigen, nur einmal laufen lassen und die Rolle ihre Handler mitten im Ablauf ausführen lassen.
6. `app_api_token` in einer mit `ansible-vault` erstellten Datei halten, die mit der Passwortdatei geöffnet wird und sonst nirgends im Klartext steht.
7. Es zweimal ausführen (der zweite Lauf darf nichts ändern) und dann `--check --diff`, `--tags` und `--limit` ausprobieren.

## So wird bewertet

Das Lab hat eine **Prüfung**, ein Skript, das eine abgegebene VM bewertet, ohne dass die Lehrkraft sie lesen muss. Es läuft auf einer Kopie der gespeicherten Maschine nach einem frischen Start. Zuerst liest es die Rolle: die Standardordner, eine ausgefüllte `meta/main.yml`, mindestens sechs Tasks in der Rolle und nicht im Playbook, einen Handler, eine Schleife, Tags und eine Vorlage mit `for`-Schleife und `if`. Es weist `shell`, `command`, `raw` und `script` zurück (und ein paar andere Wege an den Ansible-Modulen vorbei) sowie einen `pipe`-Lookup, der ein Programm startet. Es öffnet die Vault-Datei mit dem Passwort des Labs und weist ein im Projekt im Klartext geschriebenes Token zurück.

Danach **führt es die `site.yml` der Lernenden siebenmal mit Werten aus, die niemand kennt**: auf einer sauberen Maschine, noch einmal ohne etwas zu ändern, als Probelauf (Dry Run), der die Änderung zeigen und nichts schreiben muss, mit anderen Werten, mit jedem Tag und mit `--limit` auf den Host, den das Play nicht benutzt. Es zählt die Handler-Läufe in der echten Ausgabe, ein Handler, der dreimal läuft, wird also zurückgewiesen, und liest die entstandenen Dateien. Die erste Zeile der Ausgabe sagt, was nicht stimmt.

Was es nicht tut: Es startet nie einen echten Dienst neu, weil eine Prüfung das nicht beweisen kann, und wer auf der eigenen VM root ist, könnte Ansible selbst noch manipulieren. Es ist eine Prüfung für den Unterricht, kein Beweis.

## Was in der Datei steht

`ansible-roles.lab.yaml` ist eine vollständige **VM-Lab-Definition**, genau das, was die Plattform ausführt:

| Schlüssel | Was es ist |
| --- | --- |
| `cpus`, `memory`, `disk` | die Größe der VM jedes Lernenden: 1 CPU, 1024 MB und 10 GB |
| `open` | die Ports, die sich im Browser öffnen: keine, das Lab ist das Terminal |
| `setup` | ein Skript, das einmal als root läuft, wenn die VM jedes Lernenden zum ersten Mal startet: Es installiert `ansible-core` aus dem Ubuntu-Archiv und schreibt das Projekt, das Playbook, ein falsches Vault-Passwort und die README |
| `check` | das Bewertungsskript, als root auf einer Kopie der gespeicherten Platte nach einem frischen Start ausgeführt; sein Exit-Status ist das Ergebnis und `0` heißt bestanden |

Das Vault-Passwort im Projekt ist ein **Testwert**: Es sagt selbst, dass es kein Geheimnis ist, und das Token, das die Lernenden verschlüsseln, ist Demodaten, nie ein echtes Geheimnis.

## Auf der Plattform nutzen

**So geben Sie es einer Klasse als Aufgabe:**

1. Fügen Sie in Ihrem Modul eine Aufgabe hinzu und wählen Sie die Karte **Ubuntu-VM-Lab**.
2. Drücken Sie auf das Ansible-Rollen-Beispiel oder fügen Sie den Inhalt von `ansible-roles.lab.yaml` ein. Ein Beispiel ersetzt den Text, und Rückgängig bringt Ihren zurück.
3. Veröffentlichen Sie und probieren Sie zuerst Ihre eigene Kopie. Jeder Lernende hat eine eigene VM, die zwischen den Sitzungen erhalten bleibt.
4. Weisen Sie es wie jede Aufgabe zu. Wenn es abgegeben ist, starten Sie die Prüfung in der Zeile eines Lernenden in der Klassenliste: Sie startet eine Kopie seiner gespeicherten Maschine, die nicht zurückgespeichert wird, und zeigt, ob sie bestanden hat und was sie ausgegeben hat.

Eine Basics-Lektion der Plattform lässt sich in diesem Lab bearbeiten. Die [Seite zu diesem Lab](https://mitsurutech.co.uk/de/help/lab-ansible-roles) im Handbuch, die [Anleitung zu Ubuntu-VM-Labs](https://mitsurutech.co.uk/de/help/labs) und die [Seite mit den Beispiel-Labs](https://mitsurutech.co.uk/de/help/sample-labs) sagen den Rest.

Noch kein Konto für Ihre Einrichtung? [Das erste Jahr ist kostenlos](https://mitsurutech.co.uk/de/pricing) für bis zu 50 Lernende, und die [Anmeldung](https://mitsurutech.co.uk/de/signup) braucht nur eine E-Mail-Adresse.

## Die Datei prüfen

Bei jedem Push prüft `.github/workflows/check.yml`, ob die Datei gültiges YAML mit den obigen Schlüsseln ist und ob `setup` und `check` jeweils innerhalb des 16-KB-Limits der Plattform liegen. Es startet keine VM: Das geschieht auf den eigenen Maschinen der Plattform.

## Was in einem Lab nicht funktioniert

Die VM erreicht das Paketarchiv von Ubuntu, PyPI, npm und GitHub und sonst nichts. Das Lab nutzt nur das Ubuntu-Archiv, deshalb wird jede Aufgabe mit den eigenen Modulen von Ansible an Dateien, Ordnern und einem Systembenutzer erledigt. Es gibt keinen Dienst, der neu gestartet wird: Der Handler zeigt sich an dem, was eine Prüfung sehen kann, wie oft er lief und welche Datei er schreibt. Das Ziel ist die VM selbst, über eine lokale Verbindung.

## Über uns

Erstellt von [Mitsuru Technologies](https://mitsurutech.co.uk/de) für die [MitsuruTech Learning Platform](https://mitsurutech.co.uk/de/learning): Python, Java, Go, Node, SQL und Linux im Browser für Schulen und Hochschulen, mit Container-Labs, VM-Labs, aus GitHub bereitgestellten Web-Apps und Quizzen, die sich selbst bewerten. Code ohne Konto testen Sie im [Playground](https://mitsurutech.co.uk/de/playground).

Weitere Labs vom selben Anbieter: [lab-netlab](https://github.com/mitsurutech/lab-netlab), [lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop), [lab-dvwa](https://github.com/mitsurutech/lab-dvwa), [lab-webgoat](https://github.com/mitsurutech/lab-webgoat), [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin), [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea), [lab-sample](https://github.com/mitsurutech/lab-sample), [lab-sdn-lab](https://github.com/mitsurutech/lab-sdn-lab), [lab-reverse-lab](https://github.com/mitsurutech/lab-reverse-lab), [lab-formal-lab](https://github.com/mitsurutech/lab-formal-lab), [lab-hdl-lab](https://github.com/mitsurutech/lab-hdl-lab), [lab-k3s-lab](https://github.com/mitsurutech/lab-k3s-lab), [lab-tofu-lab](https://github.com/mitsurutech/lab-tofu-lab) und [lab-ansible-basics](https://github.com/mitsurutech/lab-ansible-basics).

## Lizenz

Dieses Werk steht unter der Lizenz [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.de). Sie dürfen es für Lernen und Lehre teilen und bearbeiten, wenn Sie Mitsuru Technologies Ltd nennen, es nicht kommerziell nutzen und alles, was Sie daraus machen, unter derselben Lizenz weitergeben. Der vollständige Text steht in der Datei `LICENSE`.

[Ansible](https://www.ansible.com) (`ansible-core`, GPL-3.0) wird aus dem Ubuntu-Archiv installiert, wenn eine VM startet; nichts davon wird hier verteilt.

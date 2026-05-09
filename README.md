# PP4

## Goal

In this exercise you will:

* Use SSH to connect to remote servers from WSL, macOS, or Linux shells, understanding the handshake and authentication process.
* Generate an Ed25519 SSH key pair and explain the concept of digital signatures.
* Configure your local SSH client via the `~/.ssh/config` file for streamlined access.
* Securely copy files between local and remote hosts using `scp`, including local-to-remote, remote-to-local, and remote-to-remote transfers.
* Automate startup tasks on the remote server by writing a shell script that runs at login and explaining the role of `~/.bashrc` vs. `~/.profile`.

**Important:** Start a stopwatch when you begin and work uninterruptedly for **90 minutes**. Once time is up, stop immediately and record exactly where you paused.

---

## Workflow

1. **Fork** this repository
2. **Modify & commit** your solution
3. **Submit your link for Review**

---

## Prerequisites

* Several starter repos are available here:
  [https://github.com/orgs/STEMgraph/repositories?q=SSH%3A](https://github.com/orgs/STEMgraph/repositories?q=SSH%3A)
* Consult the SSH and SCP man-pages for detailed options and explanations:

  * `man ssh`
  * `man scp`

---

## Tasks

### Task 1: SSH Login

**Objective:** Establish an SSH connection and observe each stage of the process.

1. From your local shell (WSL, macOS Terminal, or Linux), log into the `vorlesungsserver` (or any other remote machine of your choice, e.g. your own raspberry pi):

   ```bash
    ssh -v Lu21k09as@vorlesung
   ```
2. Carefully observe and note each step:

   * **TCP connection** to port 22 on `remotehost`.
   * **SSH protocol handshake**: key exchange and algorithm negotiation.
   * **Authentication**: public-key or password exchange.
   * **Shell allocation**: your remote session starts.
3. After login, exit the session with `exit`.

**Provide:**

```bash
# 1) The exact ssh command you ran
 ssh -v Lu21k09as@vorlesung
# 2) A detailed, step-by-step explanation of what happened at each stage
*    TCP-Verbindung wird zu Port 22 aufgebaut
*   Verbindung ist erfolgreich (`Connection established`)
*   Austausch der SSH-Protokollversionen
*   Aushandlung von Algorithmen (Key Exchange, Host Key, Verschlüsselung)
*   Server sendet seinen Host-Key zur Identifikation
*   Versuch der Anmeldung mit Public-Key → nicht akzeptiert
*   Anmeldung mit Passwort → erfolgreich
*   SSH startet eine Session (Channel wird geöffnet)
*   Remote-Shell wird gestartet (Login auf dem Server)
*   Server ist sowohl über Hostname als auch über IP erreichbar
*   Beide Verbindungen nutzen denselben Host-Key
*   Host-Key wird in "known_hosts" gespeichert (Sicherheitsfunktion)



```

---

### Task 2: Ed25519 Key Pair

**Objective:** Create a secure key pair and explain how digital signatures verify identity.

1. Generate an Ed25519 SSH key pair:

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

   * Accept the default file location (`~/.ssh/id_ed25519`). Or provide the `-f <filepath>` option additionally.
   * Enter a passphrase when prompted (optional).
2. Locate and inspect your `id_ed25519` (private key) and `id_ed25519.pub` (public key).
3. Install your key on the remote machine (e.g. `vorlesungsserver`.
4. Explain in writing:

   * How the **private key** is used to sign challenges.
   * How the **public key** on the server verifies signatures without revealing the private key.
   * Why Ed25519 is preferred (performance, security).

**Provide:**

```bash
# 1) The ssh-keygen command you ran
 ssh-keygen -t ed25519 -C "lukas.lazewski@gmx.de"
# 2) The file paths of the generated keys
-rw------- 1 Lu21k09as Lu21k09as  419 May  9 14:48 id_ed25519
-rw-r--r-- 1 Lu21k09as Lu21k09as  106 May  9 14:48 id_ed25519.pub
-rw------- 1 Lu21k09as Lu21k09as 1120 May  9 14:34 known_hosts
-rw-r--r-- 1 Lu21k09as Lu21k09as  142 May  7 08:00 known_hosts.old
# 3) Your written explanation (3–5 sentences) of the signature process
Beim Signaturprozess in SSH wird die Identität des Clients mit einem Schlüsselpaar überprüft. Der Server sendet dabei eine Herausforderung, die der Client mit seinem privaten Schlüssel signiert. Diese Signatur wird zurückgesendet und vom Server mithilfe des öffentlichen Schlüssels überprüft. Ist sie gültig, wird die Authentifizierung akzeptiert. Der private Schlüssel bleibt dabei stets geheim und wird nie über das Netzwerk übertragen.

```

---

### Task 3: SSH Config File

**Objective:** Simplify SSH commands via `~/.ssh/config`.

1. Open (or create) `~/.ssh/config` in `vim`.
2. Add entries for your hosts, for example:

   ```text
   Host my-remote
       HostName remote.example.com
       User youruser
       IdentityFile ~/.ssh/id_ed25519

   Host backup-server
       HostName backup.example.com
       User backupuser
       Port 2222
       IdentityFile ~/.ssh/id_ed25519_backup
   ```
3. Save and close the file, then test:

   ```bash
   ssh my-remote
   ssh backup-server
   ```
4. Explain:

How SSH reads `~/.ssh/config`:

*   SSH liest die Datei von oben nach unten
*   Sucht nach einem passenden `Host`-Eintrag
*   Vergleicht den eingegebenen Namen mit den `Host`-Namen
*   Verwendet den ersten Treffer
*   Wendet die zugehörigen Einstellungen an


Difference between `Host` and `HostName`

*   `Host`: Alias, den du im Befehl eingibst
*   `HostName`: echte Adresse (IP oder DNS) des Servers

How SSH reads ~/.ssh/config and matches hosts.
ssh Lu21k09as@128.140.85.215
einfach:
ssh my-remote

**Provide:**

```text
# 1) The full contents of your ~/.ssh/config
Host my-remote
       HostName 128.140.85.215
       User Lu21k09as
       IdentityFile ~/.ssh/id_ed25519

   Host backup-server
       HostName 128.140.85.215
       User Lu21k09as
       Port 2222
       IdentityFile ~/.ssh/id_ed25519_backup

# 2) A short explanation (3–4 sentences) of how the config simplifies connections
Das SSH-Config-File vereinfacht Verbindungen, indem wiederkehrende Einstellungen zentral gespeichert werden. Dadurch müssen Benutzer nicht jedes Mal Benutzername, IP-Adresse oder Port angeben. Stattdessen kann ein kurzer Alias verwendet werden, der alle notwendigen Optionen enthält. Das spart Zeit und reduziert Fehler bei der Eingabe.

```

---

### Task 4: SCP File Transfers

**Objective:** Practice copying files securely using `scp`.

1. **Local → Remote**:

   ```bash
   scp /path/to/localfile.txt youruser@remotehost:~/destination/
   ```
2. **Remote → Local**:

   ```bash
   scp youruser@remotehost:~/remotefile.log ./local_destination/
   ```
3. **Remote → Remote** (between two directories on the same remote host):

   ```bash
   scp -r youruser@remotehost:/path/dir1 youruser@remotehost:/path/dir2
   ```
4. For each command:

   * Verify file timestamps and sizes after transfer, using `ls -la`
   * Note any flags you used (e.g., `-r`, `-P` for port).
5. Explain:

   * How `scp` initiates an SSH session for each transfer.
   * The role of encryption in protecting data in transit.

**Provide:**

```bash
# 1) Each scp command you ran
scp notes.txt Lu21k09as@128.140.85.215:~/uploads/
scp Lu21k09as@128.140.85.215:~/server.log ./downloads/
scp -r Lu21k09as@128.140.85.215:~/project Lu21k09as@128.140.85.215:~/backup_project

# 2) Any flags or options used
-r   # rekursiv für Verzeichnisse

# 3) Brief explanation
scp nutzt SSH, um eine sichere Verbindung aufzubauen und Dateien zu übertragen. Die Daten werden dabei verschlüsselt, sodass sie während der Übertragung geschützt sind. Für jeden Transfer wird automatisch eine neue SSH-Session gestartet.

```

---

### Task 5: Login Shell Script & Profile Explanation

**Objective:** Automate commands at login and understand shell initialization files.

1. On the **remote** server, create a script `~/login_tasks.sh` containing at least three commands you find useful (e.g., `echo "Welcome $(whoami)"`, `uptime`, `ls ~/projects`). You may either use `vim` or try the following to create a file from your commandline directely:

   ```bash
   cat << 'EOF' > ~/login_tasks.sh
   #!/usr/bin/env bash
   echo "Welcome $(whoami)! Today is $(date)."
   uptime
   ls ~/projects
   EOF
   chmod +x ~/login_tasks.sh
   ```

> The files content should be something akin to:
> ```bash
> #!/usr/bin/env bash
> echo "Welcome $(whoami)! Today is $(date)."
> uptime
> ls ~/projects
> ```

2. Append to your `~/.bashrc` (or `~/.profile` if using a login shell) a line to source this script on each new session:

   ```bash
   echo "source ~/login_tasks.sh" >> ~/.bashrc
   ```
3. Log out and log back in to trigger the script.
4. Explain:

   * The difference between `~/.bashrc` and `~/.profile` (interactive vs. login shells):
     * ~/.bashrc wird bei jeder interaktiven Shell geladen (z. B. wenn man ein Terminal öffnet oder eine neue Shell startet)
     * ~/.profile wird nur bei einer Login-Shell geladen (z. B. beim Einloggen per SSH oder beim Systemstart)

   * Why and when each file is read:
     * ~/.bashrc wird gelesen, wenn eine neue interaktive Shell startet (z. B. neues Terminal oder bash)
     * ~/.profile wird gelesen, wenn sich der Benutzer anmeldet (Login-Shell, z. B. per SSH)
     * .profile wird also einmal pro Login ausgeführt, .bashrc bei jeder neuen Shell

    * How sourcing differs from executing:
      * Beim Sourcen (`source datei` oder `. datei`) wird das Skript in der aktuellen Shell ausgeführt
      * Alle Änderungen (z. B. Variablen, Pfade) bleiben in der aktuellen Umgebung erhalten
      * Beim Ausführen (`./datei`) wird das Skript in einer neuen Subshell gestartet
      * Änderungen wirken nur in dieser Subshell und gehen danach verloren


**Provide:**

```bash
# 1) The contents of login_tasks.sh

Welcome Lu21k09as! Today is Sat May  9 03:32:01 PM UTC 2026.

# 2) The lines you added to ~/.bashrc or ~/.profile
source ~/login_tasks.sh
# 3) Your explanation (3–5 sentences) of shell init files and sourcing vs. executing
Die Datei ~/.bashrc wird bei jeder interaktiven Shell ausgeführt, während ~/.profile nur bei einer Login-Shell beim Einloggen geladen wird. Deshalb nutzt man ~/.bashrc für Befehle, die in jeder Shell verfügbar sein sollen. Beim Sourcen wird ein Skript direkt in der aktuellen Shell ausgeführt, sodass Änderungen erhalten bleiben. Beim normalen Ausführen wird eine neue Subshell gestartet, wodurch Änderungen nach dem Beenden verloren gehen.
```

---

**Remember:** Stop working after **90 minutes** and record where you stopped.

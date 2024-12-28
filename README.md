# prot-ger-un-serveur-Fedora-avec-Apache-contre-les-attaques
Pour protéger un serveur Fedora avec Apache contre les attaques, vous devez prendre des mesures de sécurité à plusieurs niveaux. Voici un guide pour sécuriser efficacement votre serveur Apache sur Fedora.


---

### 1. **Mise à jour et maintenance**
Maintenir votre système et vos logiciels à jour est crucial :
```bash
sudo dnf update -y
```
Cela inclut Apache, les modules, et toutes les dépendances.

---

### 2. **Configurer le pare-feu**
Utilisez **firewalld** pour limiter l'accès aux ports nécessaires (par défaut, le port 80 pour HTTP et 443 pour HTTPS) :
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

---

### 3. **Désactiver les modules inutiles**
Réduisez la surface d'attaque en désactivant les modules Apache non nécessaires :
```bash
sudo dnf remove mod_php
sudo dnf remove mod_perl
sudo dnf remove mod_python
```

Vérifiez les modules actifs :
```bash
sudo apachectl -M
```

---

### 4. **Configurer SELinux**
Fedora utilise SELinux pour renforcer la sécurité. Assurez-vous qu'il est activé et correctement configuré :
```bash
sestatus
```
Pour autoriser des actions spécifiques (par exemple, permettre Apache d'accéder à un fichier spécifique) :
```bash
sudo semanage fcontext -a -t httpd_sys_content_t "/path/to/your/file(/.*)?"
sudo restorecon -Rv /path/to/your/file
```

---

### 5. **Configurer les permissions des fichiers**
Les fichiers web doivent appartenir à l'utilisateur et au groupe Apache (`apache:apache`), mais les permissions doivent être strictement définies :
```bash
sudo chown -R apache:apache /var/www/html
sudo chmod -R 755 /var/www/html
```

---

### 6. **Activer HTTPS avec SSL/TLS**
#### Installer `mod_ssl` et générer un certificat auto-signé :
```bash
sudo dnf install mod_ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/pki/tls/private/selfsigned.key \
  -out /etc/pki/tls/certs/selfsigned.crt
```
#### Modifier la configuration SSL :
Éditez le fichier `/etc/httpd/conf.d/ssl.conf` pour pointer vers les certificats :
```bash
SSLCertificateFile /etc/pki/tls/certs/selfsigned.crt
SSLCertificateKeyFile /etc/pki/tls/private/selfsigned.key
```
#### Redémarrer Apache :
```bash
sudo systemctl restart httpd
```

---

### 7. **Configurer les en-têtes HTTP de sécurité**
Ajoutez ces lignes dans le fichier de configuration Apache (`/etc/httpd/conf/httpd.conf` ou `/etc/httpd/conf.d/ssl.conf`) :
```apache
Header always set X-Content-Type-Options "nosniff"
Header always set X-Frame-Options "DENY"
Header always set X-XSS-Protection "1; mode=block"
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
Header always set Content-Security-Policy "default-src 'self';"
```
N'oubliez pas d'activer le module headers :
```bash
sudo dnf install mod_headers
sudo systemctl restart httpd
```

---

### 8. **Limiter les requêtes pour éviter les attaques DoS**
Installez le module **mod_evasive** :
```bash
sudo dnf install mod_evasive
```
Configurez-le dans `/etc/httpd/conf.d/mod_evasive.conf` :
```apache
DOSHashTableSize    2048
DOSPageCount        20
DOSSiteCount        100
DOSBlockingPeriod   600
```
Créez un dossier pour les logs :
```bash
sudo mkdir -p /var/log/httpd/evasive
sudo chown -R apache:apache /var/log/httpd/evasive
```

---

### 9. **Surveiller les journaux**
Utilisez les journaux pour identifier les activités suspectes :
```bash
sudo tail -f /var/log/httpd/access_log
sudo tail -f /var/log/httpd/error_log
```

---

### 10. **Protection contre les injections SQL ou XSS**
Si vous utilisez des applications web dynamiques, installez un WAF (pare-feu applicatif) comme **ModSecurity** :
```bash
sudo dnf install mod_security
sudo systemctl restart httpd
```
Activez le jeu de règles de base (OWASP CRS) pour une protection avancée.

---

### 11. **Configurer Fail2ban**
Fail2ban peut bloquer les adresses IP après plusieurs tentatives échouées :
```bash
sudo dnf install fail2ban
```
Configurez `/etc/fail2ban/jail.local` pour surveiller Apache :
```ini
[apache-auth]
enabled = true
port = http,https
filter = apache-auth
logpath = /var/log/httpd/error_log
maxretry = 3
```
Redémarrez le service :
```bash
sudo systemctl restart fail2ban
```

---

### 12. **Auditer régulièrement le serveur**
- **Lynis** pour un audit de sécurité :
```bash
sudo dnf install lynis
sudo lynis audit system
```
- Vérifiez les vulnérabilités avec des outils comme **OpenVAS** ou **Nessus**.

---

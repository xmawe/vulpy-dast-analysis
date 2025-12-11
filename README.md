# Vulpy DAST Analysis

## 📋 Description

Ce projet effectue une analyse DAST (Dynamic Application Security Testing) de l'application Vulpy en utilisant OWASP ZAP. L'analyse compare deux versions :
- **Bad** (vulnérable) : version intentionnellement vulnérable sur le port 5000
- **Good** (sécurisée) : version corrigée sur le port 5001

## 🛠️ Outils Utilisés

- **OWASP ZAP** : Scanner de sécurité pour applications web (DAST)
- **Jenkins** : Automatisation du pipeline de scan
- **Docker** : Conteneurisation des services

## 🚀 Démarrage

### 1. Lancer l'environnement

```bash
cd vulpy-dast-analysis
docker-compose up -d
```

### 2. Accéder aux services

- **Jenkins** : http://localhost:8080
- **OWASP ZAP** : http://localhost:8090
- **Vulpy Bad** : http://localhost:5000
- **Vulpy Good** : http://localhost:5001

### 3. Configuration Jenkins

```bash
# Récupérer le mot de passe initial
docker exec jenkins-dast cat /var/jenkins_home/secrets/initialAdminPassword
```

### 4. Lancer le scan DAST

1. Créer un nouveau pipeline dans Jenkins
2. Pointer vers le Jenkinsfile
3. Cliquer sur "Build Now"

## 📊 Pipeline DAST

Le pipeline exécute 9 étapes :

1. **Setup** : Préparation des répertoires
2. **Wait for Services** : Attente que ZAP et Vulpy soient prêts
3. **Spider Scan - Bad** : Exploration de la version vulnérable
4. **Active Scan - Bad** : Scan actif de la version vulnérable
5. **Generate Report - Bad** : Génération des rapports (HTML, JSON, XML)
6. **Spider Scan - Good** : Exploration de la version sécurisée
7. **Active Scan - Good** : Scan actif de la version sécurisée
8. **Generate Report - Good** : Génération des rapports (HTML, JSON, XML)
9. **Summary** : Résumé des vulnérabilités détectées

## 📁 Rapports Générés

```
dast-reports/
├── zap-report-bad.html      # Rapport HTML version vulnérable
├── zap-alerts-bad.json      # Alertes JSON version vulnérable
├── zap-report-bad.xml       # Rapport XML version vulnérable
├── zap-report-good.html     # Rapport HTML version sécurisée
├── zap-alerts-good.json     # Alertes JSON version sécurisée
├── zap-report-good.xml      # Rapport XML version sécurisée
└── scan-info.txt            # Informations de scan
```

## 🔍 Types de Vulnérabilités DAST

OWASP ZAP détecte :
- Injection SQL
- Cross-Site Scripting (XSS)
- Failles d'authentification
- Exposition de données sensibles
- Mauvaise configuration de sécurité
- Cross-Site Request Forgery (CSRF)
- En-têtes de sécurité manquants
- Cookies non sécurisés

## 📝 Workflow

1. **Scan initial** : Générer les rapports before-correction
2. **Analyse** : Identifier 2 vulnérabilités critiques
3. **Correction** : Corriger les vulnérabilités dans good/
4. **Re-scan** : Générer les rapports after-correction
5. **Vérification** : Confirmer que les vulnérabilités sont corrigées

## 🔧 Commandes Utiles

```bash
# Voir les logs ZAP
docker logs owasp-zap -f

# Voir les logs Jenkins
docker logs jenkins-dast -f

# Redémarrer les services
docker-compose restart

# Arrêter l'environnement
docker-compose down

# Arrêter et supprimer les volumes
docker-compose down -v
```

## 📚 Ressources

- [OWASP ZAP Documentation](https://www.zaproxy.org/docs/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [ZAP API Documentation](https://www.zaproxy.org/docs/api/)

## ⚠️ Avertissement

Cette analyse est effectuée dans un environnement contrôlé à des fins pédagogiques. Ne jamais scanner d'applications en production sans autorisation explicite.

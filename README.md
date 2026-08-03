# 🛡️ Synthetic Benchmark of 80 Cyberattacks for Microservices (Online Boutique)

![Version](https://img.shields.io/badge/version-1.0-blue.svg)
![Research](https://img.shields.io/badge/Research-Loria_RESIST-darkred.svg)
![Task](https://img.shields.io/badge/Task-Threat_Detection_&_RCA-critical.svg)
![Format](https://img.shields.io/badge/Format-JSON--ChatML-yellow.svg)

## 📌 Description du Projet

Ce dépôt contient un jeu de données d'évaluation synthétique (Benchmark) composé de **80 scénarios de cyberattaques**. Il a été spécialement conçu pour évaluer les capacités de détection des menaces et d'analyse de cause racine (*Root Cause Analysis - RCA*) des modèles d'Intelligence Artificielle (notamment les LLMs) dans un contexte de sécurité opérationnelle (*SOC / DevSecOps*) sur des architectures microservices *cloud-native*.

Contrairement aux jeux de données massifs d'entraînement, ce benchmark sert d'**ensemble de test de référence (Test Set)** pour mesurer la capacité de généralisation d'un modèle face à des empreintes d'attaques hétérogènes dispersées à travers la télémétrie (Métriques, Logs, Traces).

---

## 🚫 Pourquoi une génération synthétique ? (Zero Data Leakage)

Dans le cadre de l'évaluation scientifique des LLMs (ex: Llama 3.2), tester un modèle sur un sous-ensemble de ses propres données d'entraînement introduit un biais mathématique majeur appelé **Fuite de données (*Data Leakage*)**. 

Ce générateur a été créé spécifiquement pour produire un corpus de test **strictement inédit (*Unseen Test Set*)**. Il permet d'évaluer la véritable capacité de raisonnement (*Zero-Shot Reasoning*) d'une IA face à une intrusion complexe. L'objectif est de garantir que le modèle a compris la sémantique et la topologie de l'attaque en corrélant des traces, et non qu'il a simplement mémorisé une signature textuelle issue de ses bases de données d'entraînement publiques.

---

## 🔬 Fiabilité et Légitimité des Sources

Bien que généré de manière procédurale pour les besoins stricts de l'évaluation comparative, ce jeu de données est fondé sur les standards de l'industrie de la cybersécurité et de la recherche académique :

1. **Architecture Cible Officielle** : Les données simulent l'architecture **Online Boutique**, l'application de référence microservices open-source maintenue par Google Cloud, offrant une topologie réseau réaliste avec une passerelle (*API Gateway/Frontend*) et des services backend hautement couplés.
2. **Taxonomie des Menaces (Threat Models)** : Les scénarios reproduisent fidèlement des vecteurs d'attaques majeurs documentés par l'**OWASP Top 10** et la matrice **MITRE ATT&CK**.
3. **Structure Multi-source (Triplets)** : Chaque observation respecte le triptyque de télémétrie standard (Métriques, Logs applicatifs, Traces OpenTelemetry), garantissant que les empreintes laissées par les attaquants sont topologiquement et temporellement cohérentes.

---

## ⚙️ Architecture du Générateur & Logique Algorithmique

La génération de ce benchmark repose sur un couplage entre des **modèles statistiques d'anomalies de trafic** et l'**injection de gabarits d'attaques sémantiques (payload templates)**.

### 1. Modélisation et Injection des Métriques (Volumétrie vs Furtivité)
Les métriques numériques (séries temporelles) simulent deux grandes catégories de comportements malveillants :
* **Attaques Volumétriques (Spikes)** : Pour les scénarios DDoS, une fonction mathématique force le métrique cible (ex: `network_receive_bytes`, `cpu_usage` du point d'entrée) à transiter vers un état critique d'épuisement des ressources, simulant des milliers de requêtes par seconde.
* **Attaques Furtives (Stealth Noise)** : Pour le SSRF ou l'Injection SQL, les variations de métriques sont minimes (maintien d'un bruit gaussien normal). Cela force le modèle IA à se concentrer exclusivement sur la sémantique des logs et des traces, reproduisant ainsi la difficulté des attaques ciblées modernes (Advanced Persistent Threats).

### 2. Gabarits Applicatifs Déterministes (Log & Payload Templates)
Afin de coller à la réalité des intrusions, le générateur utilise un dictionnaire dynamique contenant les signatures d'attaques réelles :
* **Bourrage d'identifiants (Brute Force)** : Injection massive de gabarits canoniques de type `HTTP 401 Unauthorized - Invalid credentials for user [...]`.
* **Injections SQL (SQLi)** : Insertion de payloads d'évasion (ex: `' OR '1'='1`) déclenchant des exceptions de type `java.sql.SQLSyntaxErrorException` dans les services backend interrogeant des bases de données (ex: `cartservice`, `checkoutservice`).
* **SSRF (Server-Side Request Forgery)** : Empreintes de requêtes illégitimes vers des adresses IP privées/internes (`127.0.0.1`, `169.254.169.254`) générant des erreurs réseau isolées de type `Connection Refused`.

### 3. Génération Procédurale des Traces (Attack Propagation)
Les arbres d'appels distribués (OpenTelemetry/Jaeger) tracent le cheminement de l'attaque depuis l'extérieur vers l'intérieur du cluster Kubernetes :
* **Stochasticité des IDs** : Chaque transaction se voit attribuer un `TraceID` et un `SpanID` aléatoires uniques (UUID hexadécimaux).
* **Topologie de l'Exploit** : Le script simule la traversée de la requête malveillante. Par exemple, une attaque SQLi passe par le `frontend` (qui enregistre une requête HTTP 200 standard) mais échoue de manière critique au niveau du microservice final, illustrant l'importance d'une analyse topologique complète pour localiser le service vulnérable.

---

## ⚖️ Structure du Dataset (Perfectly Balanced)

Pour éviter tout biais de classification et assurer une évaluation stricte des modèles d'IA (notamment pour le calcul du Macro F1-Score), le dataset est **parfaitement équilibré**. Il génère 4 classes distinctes d'attaques cyber avec exactement 20 occurrences pour chaque catégorie, produisant un total de $4 \times 20 = 80$ cas uniques :

* 🌊 **DDoS (Déni de Service)** : Saturation des ressources (Réseau/CPU) aux points d'entrée Ingress du cluster. (20 scénarios)
* 🔐 **Brute Force** : Tentatives d'authentification automatisées et massives. (20 scénarios)
* 💉 **SQL Injection (SQLi)** : Altération des requêtes en base de données provoquant des exceptions syntaxiques. (20 scénarios)
* 🕵️ **SSRF (Server-Side Request Forgery)** : Détournement d'un microservice pour scanner ou interroger des ressources internes non autorisées. (20 scénarios)

---

## 📄 Aperçu des Données (Exemple au format JSONL / ChatML)

Les données sont formatées de manière à être directement utilisables pour le fine-tuning (LoRA) ou l'inférence via prompt. Chaque ligne du fichier JSONL contient l'état du système (`input`) et le diagnostic attendu (`reponse_attendue`).

```json
{
  "system_prompt": "You are a DevSecOps AI. Analyze the unified telemetry to diagnose the root cause and locate the targeted microservice.",
  "input": "=== METRICS ===\nfrontend: cpu_usage=0.45, network_rx=1200\ncheckoutservice: cpu_usage=0.55, network_rx=300\n\n=== LOGS ===\n[frontend] 2026-08-03T11:45:01Z INFO: Request received POST /checkout\n[checkoutservice] 2026-08-03T11:45:02Z ERROR: java.sql.SQLSyntaxErrorException: You have an error in your SQL syntax near ''1'='1' at line 1\n\n=== TRACES ===\nTraceID: a3f89b2c\n  [frontend] HTTP POST /checkout (200ms)\n    -> [checkoutservice] ProcessPayment (Timeout - Error: SQL Exception)",
  "reponse_attendue": {
    "Domain": "Cybersecurity (SOC)",
    "Root_Cause": "SQL Injection (SQLi)",
    "Target_Service": "checkoutservice",
    "Confidence_Score": 0.99
  }
}

# Benchmark & Rapport d'Évaluation Mistral AI (Challenge CoHoMa 4)

Ce document consigne les résultats officiels du benchmark d'inférence locale exécuté avec les modèles **Mistral AI** dans le cadre du **Challenge CoHoMa 4**.

---

## 1. Contexte & Objectif du Benchmark

L'objectif de ce benchmark est de mesurer et comparer la capacité de la suite de modèles **Mistral AI** exécutés localement (Edge AI) à opérer comme **Cerveau Tactique Autonome (Cognitive Tactical Engine)** lors de la perte de liaison de données (C2) avec le robot terrestre (Tank / Travelers).

### Les 3 Critères de Succès (KPIs) :

1. **KPI 1 - Structure JSON 100 % Valide :** Formatage strict et parsable sans faille par le middleware robotique ROS2 (10/10 essais).
2. **KPI 2 - Temps de Latence Inférence Temps Réel :** Génération et retour de la décision en **moins de 3,0 secondes**.
3. **KPI 3 - Respect des Règles Tactiques de Survie (Véracité) :** Adhésion stricte et déterministe aux règles de priorité opérationnelles (Batterie < 20% > Obstacle > Zone LIMA > Nominal).

---

## 2. Tableau Récapitulatif des Modèles Mistral AI Évalués

| Modèle Ollama | Nom de Modèle Réel Explicite | Architecture / Taille | Temps de Warmup VRAM | Structure JSON (KPI 1) | Latence Moyenne (KPI 2) | Fourchette Latence (Min - Max) | Véracité Tactique (KPI 3) | KPI Total (Verdict) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| **`ministral-3:3b`** | **Ministral 3 3B Instruct** | ~3.8B (3.0 Go) | 11.1285 s | **10/10 (100 %)** | **1.9208 s** | **1.7600 – 2.3313 s** | **10/10 (100 %)** | **SUCCÈS** |
| **`mistral:latest`** | **Mistral 7B Instruct v0.3** | ~7.2B (4.4 Go) | 9.4604 s | **10/10 (100 %)** | **2.8561 s** | **1.9009 – 3.9198 s** | **10/10 (100 %)** | **SUCCÈS** |
| **`ministral-3:8b`** | **Ministral 3 8B Instruct** | ~8.9B (6.0 Go) | 12.3720 s | **10/10 (100 %)** | **4.1429 s** | **2.6880 – 8.6477 s** | **10/10 (100 %)** | **ÉCHEC** *(> 3.0s)* |

---

## 3. Configuration Matérielle & Accélération GPU

* **Machine de Test :** Apple MacBook Pro (16-inch)
* **Accélération Matérielle :** Apple Silicon Metal GPU (Architecture RAM Unifiée)
* **Configuration d'Inférence GPU :** 
  - L'option `"options": {"num_gpu": -1}` est injectée dans toutes les requêtes afin d'offloader **100 % des couches du modèle sur le GPU** (Metal sous macOS, CUDA sous Linux/Windows).
* **Plateforme Locale :** Ollama API v0.4+ (`/api/chat` avec contrainte de grammaire `"format": "json"`)
* **Couverture Scénarios :** 10 scénarios tactiques variés (gestion batterie critique, franchissement d'obstacles, zones d'interdiction LIMA, patrouille nominale).

---

## 4. Analyse Technique des Résultats

### 4.1 `ministral-3:3b` : Le Champion du Déploiement Embarqué
* **Vitesse d'exécution :** Avec une latence moyenne de **1.92 s** et une stabilité exemplaire (toujours entre 1.76s et 2.33s), `ministral-3:3b` est **le modèle le plus rapide et le plus régulier** testé sur la plateforme.
* **Respect des contraintes critiques :** 100% de conformité JSON et 100% de décisions correctes.
* **Empreinte mémoire :** 3.0 Go seulement, laissant une marge substantielle de RAM/VRAM pour les autres nœuds ROS2 (SLAM, navigation Nav2, vision par ordinateur).

### 4.2 `mistral:latest` (`Mistral 7B Instruct v0.3`) : La Référence Éprouvée
* **Performance :** Valide le KPI avec **2.85 s** de moyenne sous la barre fatidique des 3 secondes.
* **Richesse contextuelle :** Fournit des justifications tactiques très bien formulées tout en respectant scrupuleusement le schéma Pydantic attendu.

### 4.3 `ministral-3:8b` : Puissant mais Limité par la Bande Passante Mémoire
* Malgré une exactitude parfaite sur les règles (10/10), son empreinte (6.0 Go) porte sa latence moyenne à **4.14 s**, dépassant le budget de 3.0 s sur ce palier matériel. Il conviendrait davantage à un nœud de commandement déporté qu'à un micro-calculateur embarqué sur chenilles.

---

## 5. Synthèse & Recommandation Opérationnelle

Pour le challenge **CoHoMa 4**, le modèle **`ministral-3:3b`** développé par **Mistral AI** représente la solution optimale :
1. **Ultra-faible latence (< 2.0 s)** garantissant la réactivité immédiate du robot face aux menaces ou obstacles.
2. **Parfaite adhésion au format JSON structuré**, permettant une intégration directe avec les topics et services ROS2.
3. **100 % hors-ligne et souverain**, respectant les exigences de silence radio et d'indépendance technologique en zone de guerre électronique.
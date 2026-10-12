# Benchmark & Rapport d'Analyse de Performance Matérielle (CoHoMa 4)

Ce document consigne les résultats officiels du benchmark d'inférence locale exécuté sur la plateforme de test.

---

## 1. Contexte & Objectif du Benchmark

L'objectif de ce benchmark est de mesurer et comparer la capacité de différents modèles de langage locaux (Edge AI) à faire office de **Cerveau Tactique de Secours (Failsafe Cognitive Engine)** lors de la perte de liaison de données (C2) avec le robot terrestre.

### Les 3 Critères de Succès (KPIs) :

1. **KPI 1 - Structure JSON 100 % Valide :** Formatage parfait et parsable par le code ROS2 (10/10).
2. **KPI 2 - Temps de Latence Inférence :** Génération de la décision en **moins de 3,0 secondes**.
3. **KPI 3 - Respect des Règles de Survie (Véracité) :** Adhésion stricte aux règles de priorité tactiques (Batterie < 20% > Obstacle > Zone LIMA > Nominal).

---

## 2. Tableau Récapitulatif des Modèles Testés

> [!NOTE]
> Le tag Ollama **`mistral:latest`** pointe actuellement vers **`Mistral 7B Instruct v0.3`** (architecture 7.2B, quantification Q4_K_M). Ce nommage explicite est consigné afin de garantir la reproductibilité pérenne des tests si l'alias `latest` venait à changer dans les versions futures d'Ollama.

| Modèle Local (Alias Ollama) | Modèle Réel Explicite | Taille / Paramètres | Warmup | Structure JSON (KPI 1) | Latence Moyenne (KPI 2) | Latence Min - Max | Véracité (KPI 3) | KPI Total (Verdict) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **mistral:latest** | **Mistral 7B Instruct v0.3** | ~7.2B (4.4 Go) | 6.7012 s | 10/10 (100 %) | **2.8561 s** | 1.9009 – 3.9198 s | 10/10 (100 %) | **SUCCÈS** |
| **ministral-3:3b** | **Ministral 3 3B Instruct** | ~3.8B (3.0 Go) | 11.1285 s | 10/10 (100 %) | **1.9208 s** | 1.7600 – 2.3313 s | 10/10 (100 %) | **SUCCÈS** |
| **ministral-3:8b** | **Ministral 3 8B Instruct** | ~8.9B (6.0 Go) | 12.3720 s | 10/10 (100 %) | **4.1429 s** | 2.6880 – 8.6477 s | 10/10 (100 %) | **ÉCHEC** |
| **qwen3:14b** | **Qwen 2.5/3 14B Reasoning** | ~14.8B (9.3 Go) | 20.8432 s | 10/10 (100 %) | **15.5359 s** | 11.3476 – 21.5677 s | 10/10 (100 %) | **ÉCHEC** |
| **qwen3:8b** | **Qwen 2.5/3 8B Reasoning** | ~8.2B (5.2 Go) | 21.0165 s | 10/10 (100 %) | **13.8974 s** | 10.4717 – 19.0131 s | 10/10 (100 %) | **ÉCHEC** |
| **qwen3:4b** | **Qwen 2.5/3 4B Reasoning** | ~4.0B (2.5 Go) | 26.1985 s | 10/10 (100 %) | **22.2704 s** | 11.4100 – 41.3863 s | 10/10 (100 %) | **ÉCHEC** |
| **qwen3:1.7b** | **Qwen 2.5/3 1.7B Reasoning**| ~2.0B (1.4 Go) | 5.5746 s | 10/10 (100 %) | **3.6790 s** | 2.5663 – 4.8930 s | 10/10 (100 %) | **ÉCHEC** |
| **failsafe_engine** | **Moteur Déterministe Python** | < 10 Mo | N/A | 10/10 (100 %) | **0.0002 s** | 0.0000 – 0.0020 s | 10/10 (100 %) | **SUCCÈS** |

---

## 3. Configuration Matérielle & Accélération GPU

* **Machine de Test :** Apple MacBook Pro (16-inch)
* **Processeur (CPU) & Accélération GPU :** Apple M3 Pro (Architecture RAM Unifiée & Metal GPU)
* **Mémoire RAM / VRAM Disponibles :** 18 Go / 36 Go RAM Unifiée
* **Paramétrage Accélération Matérielle :**
  - Dans le code (`tactical_agent.py`), l'option Ollama `"options": {"num_gpu": -1}` est explicitement injectée dans les requêtes API `/api/chat`.
  - Cette directive force le déchargement de **100 % des couches du réseau neuronal** sur le GPU actif :
    - Sur **macOS** : GPU Metal via mémoire unifiée Apple Silicon.
    - Sur **Windows / Linux** : GPU dédié NVIDIA (CUDA) ou AMD (ROCm).
* **Plateforme d'Inférence Locale :** Ollama API v0.4+ (Backends `/api/chat` et `/api/generate`)
* **Système d'Exploitation :** macOS

---

## 4. Analyse des Résultats & Recommandations

### 4.1 Analyse de la Latence (Seuil critique < 3.0s)

* **Ministral 3 3B (`ministral-3:3b`) est le meilleur compromis actuel :**
  - Avec une latence moyenne remarquable de **1.9208 s** (min 1.76 s, max 2.33 s), il est **plus rapide de près d'une seconde** que Mistral 7B.
  - Sa taille compacte (3.0 Go) et sa rapidité d'exécution le rendent idéal pour l'embarqué sans compromettre la fiabilité.
* **Mistral 7B (`mistral:latest` = `Mistral 7B Instruct v0.3`) :**
  - Valide également le KPI 2 avec **2.8561 s** en moyenne, mais avec des pointes jusqu'à 3.91 s qui peuvent ponctuellement frôler ou dépasser le timeout sur des contextes plus chargés.
* **Ministral 3 8B (`ministral-3:8b`) :**
  - Échoue au KPI 2 avec une moyenne de **4.1429 s** (et des pics à 8.64 s). Malgré une bonne précision tactique, il s'avère trop lourd pour le seuil strict de 3 secondes sur cette configuration.
* **Série Qwen3 Reasoning (1.7B à 14B) :**
  - Tous les modèles Qwen3 échouent au KPI 2 en raison de leur mécanisme de réflexion interne (*Thinking Tokens* `<think>...</think>`) qui démultiplie le temps de génération.

### 4.2 Analyse du Respect des Consignes (Véracité & JSON)

* **KPI 1 (JSON) : 100 % (10/10)** sur l'ensemble des modèles testés.
* **KPI 3 (Véracité) : 100 % (10/10)** sur l'ensemble des modèles testés. La logique tactique (Règle 1 Batterie < 20% > Règle 2 Obstacle > Règle 3 Zone LIMA > Nominal) est appliquée avec exactitude.

### 4.3 Recommandations Finales pour le Déploiement Embarqué

1. **Choix LLM N°1 (Recommandé) : `ministral-3:3b`**
   - Vitesse sous les 2 secondes (1.92s en moyenne), faible empreinte mémoire VRAM (3.0 Go) et score tactique parfait de 100%.
2. **Choix LLM N°2 (Alternative éprouvée) : `mistral:latest` (`Mistral 7B Instruct v0.3`)**
   - Modèle 7B solide, mais opérant proche de la limite des 3 secondes (2.85s).
3. **Moteur Failsafe Critique (Obligatoire) : `failsafe_engine`**
   - Exécution instantanée en **0.2 ms** (0.0002 s). Assure la sécurité du robot si le LLM dépasse le budget temps ou est indisponible.
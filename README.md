# Failsafe Cognitive COHOMA - Agent Tactique Autonome (Mistral AI Edge)

[![Challenge CoHoMa 4](https://img.shields.io/badge/Challenge-CoHoMa%204-blue.svg)](https://cohoma.fr)
[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-green.svg)](https://www.python.org/)
[![Powered by Mistral AI](https://img.shields.io/badge/LLM-Mistral%20AI%20(Ministral%20%26%20Mistral%207B)-orange.svg)](https://mistral.ai)
[![Cross-Platform](https://img.shields.io/badge/OS-macOS%20%7C%20Linux%20%7C%20Windows-blue.svg)](#)
[![Offline Edge](https://img.shields.io/badge/Execution-100%25%20Offline%20Edge-red.svg)](#)

Ce dépôt contient la preuve de concept (**POC**) du **Cerveau Tactique Autonome** développé pour le challenge militaire **CoHoMa 4**, propulsé par les modèles légers embarqués de **Mistral AI**.

Il démontre qu'un LLM Mistral exécuté 100 % en local sur le matériel embarqué d'un robot terrestre (Tank ou Travelers) peut analyser la télémétrie capteur en temps réel et rendre une décision tactique déterministe (format JSON parsable par ROS2) lors d'un brouillage ou d'une rupture totale de liaison Command & Control (C2).

---

## 1. Critères de Succès & Synthèse des 3 KPIs

Pour qualifier le POC dans les conditions opérationnelles du Challenge CoHoMa 4, le système est évalué sur 3 KPIs stricts :

| Critère de Succès (KPI) | Exigence Technique | Validation & Mesure | Statut |
| :--- | :--- | :--- | :---: |
| **KPI 1 : Formatage JSON Strict** | Output 100 % parsable sans texte parasite | Validation Pydantic (`TacticalDecision`) sur 10 scénarios | ✅ **100% Parsable** |
| **KPI 2 : Temps de Latence Inférence** | Temps de réponse inférieur à 3,0 secondes | Horodatage haute précision `time.perf_counter()` | ✅ **< 2.0s avec Ministral-3 3B** |
| **KPI 3 : Respect des Règles Tactiques** | Conformité décisionnelle par scénario | Vérification de la décision Obtenue vs Attendue | ✅ **100% Conforme** |

> **Accélération Matérielle (GPU) :** L'agent configure l'option `num_gpu: -1` afin de décharger 100 % des couches du modèle sur le GPU (Metal sur macOS, CUDA sur Linux/Windows) pour garantir une latence minimale.

---

## 2. Configuration Multi-Modèles Mistral (`config.json`)

Le système est configuré pour évaluer la gamme Edge de **Mistral AI** via Ollama ou toute API compatible OpenAI :

```json
{
  "platform": "ollama",
  "model": "ministral-3:3b",
  "models": [
    "ministral-3:3b",
    "mistral:latest",
    "ministral-3:8b"
  ],
  "platforms_config": {
    "ollama": { "url": "http://localhost:11434" },
    "lm_studio": { "url": "http://localhost:1234" },
    "llama_cpp": { "url": "http://localhost:8080" }
  },
  "timeout_sec": 3.0,
  "fallback_to_failsafe": true
}
```

---

## 3. Installation & Exécution du Benchmark

```bash
# 1. Création de l'environnement virtuel et installation des dépendances
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# 2. Téléchargement des modèles Mistral requis via Ollama
ollama pull ministral-3:3b
ollama pull mistral
ollama pull ministral-3:8b

# 3. Lancement du benchmark comparatif Mistral
python3 tactical_agent.py --benchmark

# 4. Exécution avec affichage détaillé de chaque scénario
python3 tactical_agent.py --benchmark --verbose

# 5. Tester un modèle spécifique en direct sur une télémétrie ROS2
python3 tactical_agent.py --model ministral-3:3b --battery 15 --sensor OBSTACLE_DETECTED_2M
```

---

## 4. Résultats Comparatifs de la Suite Mistral AI

Les tests exécutés sur les 10 scénarios tactiques COHOMA 4 donnent les résultats suivants :

```text
======================================================================================================
                              TABLEAU COMPARATIF DES PERFORMANCES MODÈLES
======================================================================================================
| Modèle             | Taille    | Warmup       | Structure JSON  | Latence Moy.  | Latence Min-Max    | Véracité        | KPI Total  |
|--------------------|-----------|--------------|-----------------|---------------|--------------------|-----------------|------------|
| ministral-3:3b     | ~3.8B     | 11.1285 s    | 10/10 (100 %)   | 1.9208 s      | 1.7600–2.3313 s    | 10/10 (100 %)   | SUCCÈS     |
| mistral:latest     | ~7.2B     | 9.4604 s     | 10/10 (100 %)   | 2.8561 s      | 1.9009–3.9198 s    | 10/10 (100 %)   | SUCCÈS     |
| ministral-3:8b     | ~8.9B     | 12.3720 s    | 10/10 (100 %)   | 4.1429 s      | 2.6880–8.6477 s    | 10/10 (100 %)   | ÉCHEC      |
======================================================================================================
```

*Note : `mistral:latest` correspond à la version **Mistral 7B Instruct v0.3**.*

---

## 5. Tests Unitaires & Intégrité du Code

Pour vérifier la robustesse du parsing JSON et la validation des règles :
```bash
python3 -m pytest -v
```

Pour consulter l'analyse matérielle détaillée, voir [BENCHMARK_RESULTS.md](BENCHMARK_RESULTS.md).

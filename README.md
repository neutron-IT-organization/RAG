# RAG Avancé en Environnement Déconnecté avec Llama 3

Ce projet met en œuvre un système de **Retrieval-Augmented Generation (RAG)** avancé, conçu pour fonctionner dans un environnement entièrement déconnecté. Il s'appuie sur des modèles open-source (Meta Llama 3) et des techniques de pointe pour améliorer la pertinence des réponses.

Le système est capable de répondre à des questions en se basant sur une collection de documents PDF, tout en offrant une comparaison directe entre la réponse enrichie par le contexte (RAG) et la réponse basée uniquement sur les connaissances du modèle.

## Fonctionnalités Principales

* **Mode 100% Déconnecté** : Toutes les ressources (modèles, documents, dépendances Python) sont chargées depuis des sources locales. Les modèles et les documents PDF sont récupérés depuis un serveur de stockage objet compatible S3 (Minio).
* **Modèle de Langage (LLM)** : Utilise **Meta-Llama-3.2-3B-Instruct**, un modèle puissant et compact, chargé avec une **quantisation 4-bit** (`bitsandbytes`) pour une exécution efficace sur GPU.
* **Chunking Sémantique** : Au lieu d'un découpage de texte arbitraire, le projet utilise `SemanticChunker` de LangChain pour diviser les documents en blocs de texte sémantiquement cohérents, améliorant ainsi la qualité du contexte.
* **Retriever de Voisins (Contexte Enrichi)** : Implémentation d'un retriever personnalisé (`NeighborRetriever`) qui, après avoir trouvé le chunk le plus pertinent, récupère également les chunks précédent et suivant. Cela fournit au LLM un contexte plus large et plus complet pour formuler sa réponse.
* **Transformation de Requête par le LLM** : La question de l'utilisateur est d'abord reformulée par le LLM lui-même pour être plus adaptée à une recherche de similarité sémantique.
* **Comparaison Directe RAG vs LLM seul** : L'interface finale affiche à la fois la réponse du modèle sans contexte et la réponse enrichie par les documents, permettant d'évaluer l'apport du RAG en temps réel.

## Architecture du Projet

Le workflow se décompose en deux grandes phases :

1.  **Phase d'Ingestion (Indexation)**
    * Téléchargement sécurisé des modèles (LLM et Embedding) et des documents PDF depuis un serveur Minio.
    * Chargement et traitement des documents PDF.
    * Découpage des documents en chunks sémantiques.
    * Création des *embeddings* (vecteurs numériques) pour chaque chunk.
    * Stockage des chunks et de leurs embeddings dans une base de données vectorielle locale (ChromaDB).

2.  **Phase d'Interrogation (Génération)**
    * L'utilisateur pose une question.
    * Le LLM transforme la question initiale en une requête optimisée.
    * Le `NeighborRetriever` recherche dans ChromaDB les 3 chunks les plus pertinents (le meilleur + ses deux voisins).
    * Les chunks récupérés sont injectés dans un *prompt* spécifique avec la question transformée.
    * Le LLM génère une réponse finale basée sur ce contexte enrichi.

## Prérequis

* Python 3.10+
* Un GPU NVIDIA avec support CUDA pour l'exécution du modèle.
* Un serveur de stockage objet compatible S3 (Minio) accessible depuis l'environnement d'exécution.

## Installation

1.  Clonez ce dépôt :
    ```bash
    git clone[ <votre-repo>](https://github.com/neutron-IT-organization/RAG.git)
    cd RAG
    ```

2.  Il est recommandé de créer un environnement virtuel :
    ```bash
    python -m venv venv
    source venv/bin/activate
    ```

## Configuration

Avant de lancer le notebook, assurez-vous de configurer les points suivants :

1.  **Serveur Minio** :
    * Vos modèles (Llama 3 et le modèle d'embedding) doivent être uploadés sur votre serveur Minio.
    * Vos documents PDF doivent également être placés dans un dossier sur Minio.
    * Mettez à jour les variables dans le code pour correspondre à vos chemins :
        * `MINIO_BUCKET_NAME`
        * `s3_endpoint`
        * `LOCAL_MODEL_PATH` (préfixe du LLM sur Minio)
        * `EMBEDDING_MINIO_PREFIX` (préfixe du modèle d'embedding sur Minio)
        * `PDF_DIRECTORY_ON_MINIO`
## Utilisation

1.  Lancez le notebook Jupyter :
    ```bash
    jupyter notebook impl_rag-semantic_vs_model-deconecte.ipynb
    ```

2.  Exécutez les cellules séquentiellement. Le script va :
    * Vérifier et télécharger les dépendances NLTK.
    * Se connecter à Minio et télécharger les modèles et documents si nécessaire.
    * Charger les modèles sur le GPU.
    * Indexer les documents.
    * Lancer une boucle interactive où vous pourrez poser vos questions.

3.  Dans le prompt final, tapez votre question et appuyez sur `Entrée`. Pour arrêter, tapeez `exit`.

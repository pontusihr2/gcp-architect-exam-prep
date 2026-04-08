# GCP AI/ML Services — Exam Reference (2025)

---

## Vertex AI Platform

### Overview

- **Unified ML platform** for the entire ML lifecycle: data prep → training → deployment → monitoring
- Consolidates AutoML, AI Platform, and Notebooks under one umbrella
- All Vertex AI workloads run in a VPC; supports VPC Service Controls

### Training

- **Custom Training**: Run any ML framework (TensorFlow, PyTorch, JAX, scikit-learn, XGBoost) using pre-built or custom containers
- **Training Jobs**: Single-node or distributed training; choose machine type (CPU/GPU/TPU)
- **Hyperparameter Tuning**: Automated HPT using Vertex AI Vizier; supports grid, random, Bayesian optimization
- **Distributed Training**: Data parallelism and model parallelism; supports tf.distribute, PyTorch DDP, Horovod
- **Training Pipeline**: Combine preprocessing, training, and evaluation into an automated pipeline

### Prediction (Model Serving)

- **Online Prediction (Endpoints)**: Low-latency, real-time predictions; autoscaling replicas; min/max nodes; multiple models on one endpoint (traffic splitting)
- **Batch Prediction**: Large-scale, offline predictions; no endpoint needed; output to GCS or BigQuery
- **Private Endpoints**: Vertex AI endpoint accessible only within VPC (Private Service Connect)

### Vertex AI Pipelines

- **Managed Kubeflow Pipelines**: Define ML workflows as DAGs using Python SDK (KFP or TFX)
- Reusable **components**: containerized steps; versioned; shareable via Artifact Registry
- **Caching**: Skip already-completed steps on re-run (based on inputs/outputs hash)
- Integration with Cloud Scheduler for recurring pipeline runs

### Model Registry

- Central repository for versioned ML models
- Track model lineage (training job → dataset → model version)
- Deploy models from registry to endpoints
- Stage models through environments (staging → production) with gating

### Vertex AI Experiments

- Track, compare, and visualize training runs, parameters, and metrics
- Integrated with TensorBoard for visualization
- Log custom metrics, parameters, artifacts from any training framework

### Vertex AI Feature Store

- **Online serving**: Low-latency feature retrieval for real-time prediction (< 10ms p99)
- **Offline serving**: Bulk feature export to BigQuery for training dataset generation
- **Point-in-time lookups**: Retrieve feature values as of a historical timestamp (prevents data leakage)
- **Feature monitoring**: Detect feature distribution drift over time
- **Bigtable-backed** for online store; BigQuery for offline store

---

## Vertex AI Workbench

- **Managed JupyterLab** notebooks on GCP
- **User-managed notebooks**: Full control; can customize environment; manual start/stop; persistent disk
- **Managed notebooks (Instances)**: Simplified; auto-shutdown on idle; scheduled execution; pre-configured environments
- Integration with Vertex AI, BigQuery, Dataproc, GCS
- Supports GPU-attached VMs for large-scale exploration

---

## AutoML

- Train high-quality ML models with minimal ML expertise using GUI or API

| AutoML Type | Data Type | Output |
|-------------|-----------|--------|
| AutoML Image | Images | Classification, object detection, segmentation |
| AutoML Video | Videos | Classification, object tracking, action recognition |
| AutoML Text | Text | Classification, entity extraction, sentiment |
| AutoML Tabular | Structured/tabular | Classification, regression, forecasting |
| AutoML Translation | Text | Neural machine translation models |

- Models trained are exportable (edge deployment); some models deployable to Vertex AI endpoints
- AutoML Tabular uses **Neural Architecture Search** and **ensemble methods** under the hood
- **Explainability**: Feature importance available for tabular models (Shapley values)

> **Exam Tip**: AutoML is the right choice when you have labeled data but no ML engineers. It's more expensive per training hour than custom training but requires far less ML expertise.

---

## Pre-trained APIs (AI Building Blocks)

### Vision AI

- **Cloud Vision API**: Label detection, OCR, face detection, safe search, object localization, image properties
- **Video Intelligence API**: Shot detection, label detection, object tracking, speech transcription in video, explicit content detection

### Natural Language AI

- **Natural Language API**: Entity recognition, sentiment analysis, syntax analysis, entity sentiment, content classification
- **Healthcare Natural Language API**: Extract medical entities from clinical notes (medications, diagnoses, procedures)

### Speech AI

- **Speech-to-Text API**: Audio → text; 125+ languages; diarization (speaker separation); word-level timestamps; medical/phone call models
- **Text-to-Speech API**: Text → audio; 380+ voices; WaveNet and Neural2 voices; SSML support; custom voice creation

### Translation AI

- **Cloud Translation API**: Text translation; 100+ languages; basic (v2) and advanced (v3) versions
- **Translation Hub**: Document translation at scale with human review workflow

### Document AI

- **Document AI**: Extract structured data from documents (invoices, contracts, medical forms, driver's licenses)
- **Processors**: Pre-trained (Invoice Parser, Form Parser, ID Processor) or custom trained
- **Document AI Warehouse**: Store, search, and govern processed documents

### Contact Center AI (CCAI)

- Combines Dialogflow, Speech-to-Text, Text-to-Speech for AI-powered contact center
- **Agent Assist**: Real-time suggestions to human agents
- **Virtual Agents (CCAI VA)**: Fully automated conversational agents

---

## Generative AI on Vertex AI (2025 Critical Topic)

### Gemini Models

- **Gemini 2.0 Flash**: Best cost/performance; multimodal (text, image, audio, video, code); 1M token context
- **Gemini 2.0 Pro**: Highest capability; complex reasoning, coding, long-context
- **Gemini 2.0 Flash Lite**: Lowest latency and cost; simple tasks
- Models accessible via Vertex AI API, Vertex AI Studio, and direct REST/gRPC

### Vertex AI Studio

- **Prompt Design**: Test and iterate prompts (freeform, chat, structured); compare models
- **Tune**: Fine-tune Gemini on custom data (supervised fine-tuning, RLHF)
- **Evaluate**: Benchmark models on custom evaluation datasets

### Grounding & RAG

- **Grounding with Google Search**: Gemini responses augmented with real-time Google Search results; reduces hallucination
- **Grounding with Vertex AI Search**: Augment responses with your own documents/data (RAG)
- **RAG Architecture**: User query → vector search (Vertex AI Vector Search) → retrieve relevant chunks → augment prompt → LLM response
- **Vertex AI Vector Search (formerly Matching Engine)**: High-scale approximate nearest neighbor (ANN) search; HNSW and tree-AH algorithms

### Embeddings

- **Text Embeddings API**: Convert text to dense vector representations for semantic search, clustering, classification
- **Multimodal Embeddings**: Embed images + text in same vector space for cross-modal search

### Model Garden

- Hub of 100+ models: Google first-party (Gemini, Imagen, Chirp), third-party (Llama, Mistral, Claude via partner)
- One-click deployment to Vertex AI endpoints
- Compare models side-by-side

### Vertex AI Agent Builder

- **Grounded generation**: Build production RAG pipelines with configurable chunking, retrieval, and generation
- **Vertex AI Search**: Semantic search over enterprise data (websites, structured, unstructured documents)
- **Vertex AI Conversation (Dialogflow CX)**: Build conversational agents with LLM backing
- **Extensions**: Connect agents to external APIs (tool use / function calling)

---

## MLOps Concepts

### Model Monitoring

- **Vertex AI Model Monitoring**: Detect feature drift (training-serving skew) and prediction drift in production
- **Skew Detection**: Compare serving data distribution vs training baseline (Jensen-Shannon divergence)
- **Drift Detection**: Compare serving data distribution over time
- Alerts when drift exceeds threshold; trigger retraining pipeline

### Retraining Pipelines

- Pattern: Monitoring trigger → Pub/Sub notification → Cloud Scheduler/Eventarc → Vertex AI Pipeline → Model evaluation → Conditional deployment
- **Continuous Training**: Automated retraining on schedule or data trigger
- Use **Model Registry** to version, compare, and promote models

### Feature Management

- Store features in Vertex AI Feature Store; ensures training-serving consistency
- Use Feature Store for point-in-time correct training data (prevent data leakage)

### Experiment Tracking

- Log all hyperparameters, metrics, and artifacts to Vertex AI Experiments
- Compare experiment runs to select best model before registering

---

## When to Use Which AI/ML Option

| Scenario | Recommended Approach |
|----------|---------------------|
| Classify images, no ML expertise | AutoML Image |
| Predict tabular data, no ML expertise | AutoML Tabular |
| Extract text from documents | Document AI |
| Speech transcription in app | Speech-to-Text API |
| Sentiment analysis on text | Natural Language API |
| Custom NLP model, own data | AutoML Text or Custom Training |
| Large language model / generative AI | Vertex AI Gemini models |
| Build chatbot / virtual agent | Vertex AI Agent Builder / Dialogflow CX |
| Semantic search over documents | Vertex AI Search |
| Train custom model with TensorFlow/PyTorch | Vertex AI Custom Training |
| Automate ML pipeline (retraining etc.) | Vertex AI Pipelines |
| Serve predictions at low latency | Vertex AI Online Prediction |
| Score millions of records offline | Vertex AI Batch Prediction |
| Feature store for real-time ML | Vertex AI Feature Store |
| Track/compare model experiments | Vertex AI Experiments |
| Pre-built ML model examples | Model Garden |

---

## Common Exam Gotchas

1. **AutoML vs custom training**: AutoML handles feature engineering; custom training gives full control — choose based on expertise and customization needs
2. **Vertex AI Pipelines** are Kubeflow Pipelines managed by Google — not Cloud Composer; use for ML workflows specifically
3. **Online prediction** endpoints support traffic splitting for A/B testing between model versions
4. **Feature Store point-in-time** lookups are critical for preventing data leakage in training
5. **Grounding with Google Search** requires explicit enabling and affects cost; reduces hallucination significantly
6. **Document AI processors** are region-specific — deploy processors in the same region as data for compliance
7. **Gemini token limits**: Flash supports 1M context tokens; Pro has higher limits but higher cost — choose based on context window needs
8. **Model monitoring** requires a baseline dataset (training data statistics) to detect skew

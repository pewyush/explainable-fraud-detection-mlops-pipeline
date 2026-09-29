# Problem Statement
Financial institutions lose significant revenue to fraudulent transactions, but naive fraud classifiers create their own cost: false positives block legitimate customers and erode trust, while false negatives let fraud through.

This project builds a fraud detection model whose predictions are explainable and auditable enough for a model-risk review, and demonstrates how a bank would safely roll out a model update in production — via shadow deployment — without risking customer-facing decisions on an unvalidated model.

### Final Deliverables
- Model: precision/recall/F1 and precision@k reported, cost-sensitive threshold chosen and justified (not default 0.5) — since false positives ≠ false negatives cost-wise
- Explainability: every flagged transaction has a SHAP-based reason code, plus a model card documenting training data, metrics, known limitations
- Shadow deployment: new model version scores live traffic silently, a comparison report (agreement rate, metric delta vs current champion) is generated automatically, and promotion to production is gated on that report passing a threshold
- Deployed: a FastAPI scoring endpoint + a small dashboard showing flagged transactions with their reason codes and champion/challenger comparison

### Data Source
IEEE-CIS Fraud Detection - https://www.kaggle.com/competitions/ieee-fraud-detection/data

### Tentative tech stack
- Language & environment

        Python 3.12.10
        
- Data & modeling (Phases 3-5)

        pandas, numpy — data handling
        scikit-learn — pipelines, preprocessing, train/test splitting
        imbalanced-learn (SMOTE or class-weighting) — handling the fraud class imbalance
        XGBoost as the model — industry standard for tabular fraud, trains fast, and SHAP has native fast support for tree models (TreeExplainer)
        SHAP — explainability layer

- Database (Phase 4 — "data goes into a real database")

        PostgreSQL, run via Docker locally — more realistic than SQLite, and it's the same engine you'd likely meet at a bank

- Experiment tracking & automation

        MLflow — experiment tracking + model registry (staging → production), run locally via mlflow server with a SQLite backend store and local artifact store (skip AWS-hosted tracking — adds setup cost for zero CV value at solo scale)
        DVC — pipeline automation (dvc.yaml + params.yaml), triggers retraining on parameter change

- Testing

        pytest — unit tests for data transforms and pipeline stages

- Serving & deployment (Phases 7)

        FastAPI — backend API (auto-generated docs, async, the modern default — better CV signal than Flask)
        Streamlit — dashboard/frontend (fastest path to a real deployable UI for one person; shows predictions + SHAP reason codes without you needing to build a separate JS frontend)
        Docker — containerize the API
        AWS: ECR (image registry), EC2 (compute), Application Load Balancer, Auto Scaling Group, CodeDeploy (blue-green rollout) — matches your blueprint exactly

- CI/CD

        GitHub Actions — run tests + build/push Docker image on merge to main

- Stretch goals (name them in the README, don't build unless ahead of schedule):

        Terraform for infra-as-code instead of manual AWS console setup
        Prometheus/Grafana for live API monitoring
        Swap Streamlit for a proper React frontend

This stack touches every phase of the pipeline without introducing anything exotic or hard to justify in an interview — every tool here is something an actual bank's ML platform team would recognize immediately.
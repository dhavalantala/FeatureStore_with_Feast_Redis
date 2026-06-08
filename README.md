# MLOps Feature Store with Feast & Redis

A practical MLOps project demonstrating a local feature store using **Parquet** as the offline store and **Redis** as the online store. Features are materialized from the offline store into Redis, then served for ML model training and real-time inference via a Streamlit web app.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Python | Programming Language |
| Feast | Feature Store Framework |
| Parquet | Offline Feature Store |
| Redis | Online Feature Store |
| Pandas | Data Processing |
| Scikit-learn | Machine Learning |
| Streamlit | Interactive Web UI |
| Docker | Containerization |
| Plotly | Gauge Chart Visualization |

---

## Project Structure

```
my_feature_repo/
├── data/
│   ├── customer_transactions.csv
│   └── customer_transactions.parquet
├── feature_repo/
│   ├── feature_store.yaml
│   └── example_feature_repo.py
├── generate_data.py
├── train_model.py
├── streamlit-app.py
├── model.pkl
├── Dockerfile
└── requirements.txt
```

---

## Setup

### 1. Create and Activate a Virtual Environment

```bash
python3 -m venv venv
source venv/bin/activate
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Start Redis with Docker

Create a shared Docker network so Redis and RedisInsight can communicate:

```bash
docker network create redis-network
```

Run the Redis container:

```bash
docker run -d --name redis --network redis-network redis
```

Get the Redis container's IP address (you'll need this for configuration):

```bash
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis
```

> **Note:** Save this IP address — you'll use it as `<YOUR_REDIS_IP>` in the steps below.

---

## Feature Store Configuration

### 4. Initialize the Feast Repository

```bash
feast init my_feature_repo
cd my_feature_repo
mkdir data
```

### 5. Configure `feature_store.yaml`

Edit `my_feature_repo/feature_repo/feature_store.yaml` and replace its contents with:

```yaml
project: my_feature_repo
registry: data/registry.db
provider: local
online_store:
  type: redis
  connection_string: "<YOUR_REDIS_IP>:6379"
offline_store:
  type: file
```

> Replace `<YOUR_REDIS_IP>` with the IP address you copied in Step 3.

### 6. Generate the Dataset

Run the data generation script to create the Parquet offline store:

```bash
python generate_data.py
```

### 7. Apply Feature Definitions

Register your feature definitions with Feast:

```bash
cd feature_repo
PYTHONWARNINGS="ignore" feast apply
```

A successful run will confirm your feature definitions are registered.

### 8. Materialize Features to Redis

Load the latest feature values from the offline store into Redis:

```bash
feast materialize-incremental $(date +%F)
```

You should see output similar to:

```
Materializing 3 feature views to 2025-05-23 00:00:00+00:00 into the redis online store.
```

---

## Redis Insight (Optional Web UI)

Spin up RedisInsight to visually inspect your Redis data:

```bash
docker run -d --name redisinsight --network redis-network -p 5540:5540 redis/redisinsight:latest
```

Open in your browser: [http://localhost:5540](http://localhost:5540)

Add your Redis database via **Connection Settings**:

| Field | Value |
|-------|-------|
| Alias | Any name you prefer |
| Host | `<YOUR_REDIS_IP>` |
| Port | `6379` |

> For persistent RedisInsight data, add `-v redisinsight:/data` to the run command above.

---

## Running the Application

### 9. Train the ML Model

From the `my_feature_repo` directory:

```bash
python train_model.py
```

This trains a Logistic Regression model on customer churn data and saves it as `model.pkl`.

### 10. Launch the Streamlit App

```bash
streamlit run streamlit-app.py
```

Open in your browser: [http://localhost:8501](http://localhost:8501)

Enter a Customer ID (1–5) to get a real-time churn prediction with a probability gauge chart.

---

## Docker Deployment

Build and run the full Streamlit app in a container, connected to the Redis network:

```bash
docker build -t feast-streamlit-app .
docker run -d -p 8501:8501 --network redis-network --name streamlit-app feast-streamlit-app
```

Open in your browser: [http://localhost:8501](http://localhost:8501)

---

## Cleanup

```bash
# Stop and remove containers
docker stop redis redisinsight streamlit-app
docker rm redis redisinsight streamlit-app

# Remove the Docker image
docker rmi feast-streamlit-app

# Deactivate and remove the virtual environment
deactivate
rm -rf venv
```

---

## How It Works

```
Parquet (Offline Store)
        │
        │  feast materialize-incremental
        ▼
   Redis (Online Store)
        │
        │  store.get_online_features()
        ▼
 Streamlit App ──► ML Model ──► Churn Prediction
```

1. **Offline Store** — Historical customer transaction data stored as Parquet files.
2. **Feature Materialization** — Feast loads the latest feature values into Redis.
3. **Online Serving** — The app fetches real-time features from Redis for inference.
4. **Prediction** — A Logistic Regression model predicts customer churn probability.
# MLOps Feature Store w/ Feast & Redis
In this project, I build a practical MLOps project that demonstrates a local feature store with a Parquet file offline feature store and a Redis online feature store to serve features for the training of a Machine Learning model and for inferencing.

## Set Up Environment 

### Tech Stack
1. Python:          Programming Language
2. Feast:           Feature Store Framework
3. Parquet:         Offline Feature Store
4. Redis:           Online Feature Store
5. Pandas:          Numerical Computations
6. Scikit-learn:    Machine Learning Python Package
7. Streamlit:       Package ML model with Interactive web GUI
8. Docker:          Package ML model & all dependencies in a single image file or container.
9. Plotly:          Generating a Gauge chart to display probability

### Set up Project Environment

```sh
python3 -m venv venv
source venv/bin/activate
```

### Install Required Packages
```sh
pip install -r requirements.txt
```
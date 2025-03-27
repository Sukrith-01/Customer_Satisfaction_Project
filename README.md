# Predicting how a customer will feel about a product before they even ordered it

[![PyPI - Python Version](https://img.shields.io/pypi/pyversions/zenml)](https://pypi.org/project/zenml/)

Using a customer’s historical data, this project aims to forecast the review score they might assign to their next order or purchase. The analysis leverages the Brazilian E-Commerce Public Dataset by Olist, which includes details on 100,000 orders placed across various Brazilian marketplaces between 2016 and 2018. This dataset encompasses multiple aspects of transactions, such as order status, pricing, payment details, shipping performance, customer locations, product characteristics, and customer reviews. The goal is to predict customer satisfaction scores for future orders based on features like order status, price, and payment information. To simulate a real-world application, we employ ZenML to construct a production-ready pipeline for predicting these satisfaction scores.

Purpose of This Repository
This repository showcases how ZenML enables businesses to create and deploy machine learning pipelines effectively by:

Providing a customizable framework and template for your projects.
Integrating with tools like MLflow for experiment tracking, deployment, and beyond.
Simplifying the process of building and deploying ML pipelines.

🐍 Python Requirements
To get started, install the necessary Python packages in your preferred environment by running:

## :snake: Python Requirements

```bash
git clone https://github.com/zenml-io/zenml-projects.git  
cd zenml-projects/customer-satisfaction  
pip install -r requirements.txt  
```

Starting with ZenML 0.20.0, ZenML comes bundled with a React-based dashboard. This dashboard allows you
to observe your stacks, stack components and pipeline DAGs in a dashboard interface. To access this, you need to [launch the ZenML Server and Dashboard locally](https://docs.zenml.io/user-guide/starter-guide#explore-the-dashboard), but first you must install the optional dependencies for the ZenML server:

```bash
pip install zenml["server"]
zenml up
```

If you are running the `run_deployment.py` script, you will also need to install some integrations using ZenML:

```bash
zenml integration install mlflow -y
```

The project can only be executed with a ZenML stack that has an MLflow experiment tracker and model deployer as a component. Configuring a new stack with the two components are as follows:

```bash
zenml integration install mlflow -y
zenml experiment-tracker register mlflow_tracker --flavor=mlflow
zenml model-deployer register mlflow --flavor=mlflow
zenml stack register mlflow_stack -a default -o default -d mlflow -e mlflow_tracker --set
```

## 📙 Resources & References

We had written a blog that explains this project in-depth: [Predicting how a customer will feel about a product before they even ordered it](https://blog.zenml.io/customer_satisfaction/).


## :thumbsup: The Solution

Predicting customer satisfaction for future purchases requires more than a one-time model. This project builds an end-to-end pipeline that continuously trains, deploys, and updates a machine learning model, paired with a data application that leverages the latest deployed model for business use.

The pipeline is cloud-deployable, scalable, and tracks all parameters and data flowing through it—including raw inputs, features, outputs, models, and predictions. ZenML simplifies this process with its intuitive yet robust framework.

Special emphasis is placed on ZenML’s MLflow integration:

MLflow Tracking: Logs metrics, parameters, and models.
MLflow Deployment: Deploys models seamlessly.
Streamlit: Demonstrates real-world model usage via an interactive app.

### Training Pipeline

Our standard training pipeline consists of several steps:

The core training pipeline includes these steps:

ingest_data: Loads data and constructs a DataFrame.
clean_data: Processes data by removing irrelevant columns.
train_model: Trains the model, saving it with MLflow autologging.
evaluation: Assesses model performance and logs metrics via MLflow.

### Deployment Pipeline

The deployment_pipeline.py extends the training pipeline into a continuous deployment workflow. It processes data, trains a model, and deploys it as a prediction server if it meets a configurable MSE threshold. Additional steps include:

deployment_trigger: Evaluates if the model qualifies for deployment.
model_deployer: Deploys the model using MLflow if criteria are met.
In this pipeline, ZenML’s MLflow integration logs hyperparameters, models, and metrics to a local MLflow backend. If the model’s accuracy exceeds the threshold, a local MLflow deployment server is launched or updated to serve the latest model, running as a background daemon.

A Streamlit app complements this by asynchronously consuming the deployed model. Within the app, the prediction service is accessed as follows:

```python
service = prediction_service_loader(  
    pipeline_name="continuous_deployment_pipeline",  
    pipeline_step_name="mlflow_model_deployer_step",  
    running=False,  
)  
service.predict(...)  # Makes predictions on app inputs
 
```

```python
service = prediction_service_loader(
   pipeline_name="continuous_deployment_pipeline",
   pipeline_step_name="mlflow_model_deployer_step",
   running=False,
)
...
service.predict(...)  # Predict on incoming data from the application


While this ZenML Project trains and deploys a model locally, other ZenML integrations such as the [Seldon](https://github.com/zenml-io/zenml/tree/main/examples/seldon_deployment) deployer can also be used in a similar manner to deploy the model in a more production setting (such as on a Kubernetes cluster). We use MLflow here for the convenience of its local deployment.

![training_and_deployment_pipeline](_assets/training_and_deployment_pipeline_updated.png)

## :notebook: Diving into the code

You can run two pipelines as follows:

- Training pipeline:

```bash
python run_pipeline.py
```

- The continuous deployment pipeline:

```bash
python run_deployment.py
```

## 🕹 Demo Streamlit App

There is a live demo of this project using [Streamlit](https://streamlit.io/) which you can find [here](https://share.streamlit.io/ayush714/customer-satisfaction/main). It takes some input features for the product and predicts the customer satisfaction rate using the latest trained models. If you want to run this Streamlit app in your local system, you can run the following command:-

```bash
streamlit run streamlit_app.py
```

## :question: FAQ

1. When running the continuous deployment pipeline, I get an error stating: `No Step found for the name mlflow_deployer`.

   Solution: It happens because your artifact store is overridden after running the continuous deployment pipeline. So, you need to delete the artifact store and rerun the pipeline. You can get the location of the artifact store by running the following command:

   ```bash
   zenml artifact-store describe
   ```

   and then you can delete the artifact store with the following command:

   **Note**: This is a dangerous / destructive command! Please enter your path carefully, otherwise it may delete other folders from your computer.

   ```bash
   rm -rf PATH
   ```

2. When running the continuous deployment pipeline, I get the following error: `No Environment component with name mlflow is currently registered.`

   Solution: You forgot to install the MLflow integration in your ZenML environment. So, you need to install the MLflow integration by running the following command:

   ```bash
   zenml integration install mlflow -y
   ```

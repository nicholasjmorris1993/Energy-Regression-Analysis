{
 "cells": [
  {
   "cell_type": "code",
   "execution_count": 1,
   "id": "abb00f0e-b45a-475a-ab3f-1647e3f87002",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Requirement already satisfied: plotly in /opt/conda/envs/sagemaker-distribution/lib/python3.8/site-packages (5.17.0)\n",
      "Requirement already satisfied: tenacity>=6.2.0 in /opt/conda/envs/sagemaker-distribution/lib/python3.8/site-packages (from plotly) (8.2.3)\n",
      "Requirement already satisfied: packaging in /opt/conda/envs/sagemaker-distribution/lib/python3.8/site-packages (from plotly) (23.1)\n",
      "Note: you may need to restart the kernel to use updated packages.\n",
      "Requirement already satisfied: py-cpuinfo in /opt/conda/envs/sagemaker-distribution/lib/python3.8/site-packages (9.0.0)\n",
      "Note: you may need to restart the kernel to use updated packages.\n"
     ]
    }
   ],
   "source": [
    "# install missing packages\n",
    "%pip install plotly\n",
    "%pip install py-cpuinfo"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 2,
   "id": "28211ea4-4ff7-48eb-a395-53442492a061",
   "metadata": {
    "tags": []
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Total RAM: 15.47 GB\n",
      "Available RAM: 14.16 GB\n",
      "Used RAM: 1.04 GB\n",
      "Percentage Usage Of RAM: 8.5%\n",
      "CPU Cores: 4\n",
      "CPU Speed: 2.5000 GHz\n"
     ]
    }
   ],
   "source": [
    "# check system details\n",
    "import os\n",
    "import psutil\n",
    "import cpuinfo\n",
    "\n",
    "try:\n",
    "    ram_info = psutil.virtual_memory()\n",
    "    print(f\"Total RAM: {ram_info.total / 1024 / 1024 / 1024:.2f} GB\")\n",
    "    print(f\"Available RAM: {ram_info.available / 1024 / 1024 / 1024:.2f} GB\")\n",
    "    print(f\"Used RAM: {ram_info.used / 1024 / 1024 / 1024:.2f} GB\")\n",
    "    print(f\"Percentage Usage Of RAM: {ram_info.percent}%\")\n",
    "    print(f\"CPU Cores: {os.cpu_count()}\")\n",
    "    print(f\"CPU Speed: {cpuinfo.get_cpu_info()['hz_actual_friendly']}\")\n",
    "except:\n",
    "    print(\"RAM and CPU info not available on this system\")"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 3,
   "id": "3aa0ef51-d46b-4ccf-9caa-741ad90c77c1",
   "metadata": {
    "tags": []
   },
   "outputs": [],
   "source": [
    "# import requirements\n",
    "import pandas as pd\n",
    "from sklearn.metrics import mean_squared_error, r2_score\n",
    "from regression import Regression"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 4,
   "id": "32da91cf-563e-4206-97d7-6a3c7af2bcfa",
   "metadata": {
    "tags": []
   },
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "/tmp/ipykernel_1138/860931725.py:2: DtypeWarning:\n",
      "\n",
      "Columns (716,717) have mixed types. Specify dtype option on import or set low_memory=False.\n",
      "\n"
     ]
    }
   ],
   "source": [
    "# get the data\n",
    "energy = pd.read_csv(\"energy.csv\")"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 5,
   "id": "45f9301a-5582-4abd-8e1b-33a9302eb898",
   "metadata": {
    "tags": []
   },
   "outputs": [],
   "source": [
    "# split up the data into training and testing\n",
    "y = energy[[\"NWEIGHT\"]]\n",
    "X = energy.drop(columns=\"NWEIGHT\")\n",
    "trainX = X.head(int(0.8 * X.shape[0]))\n",
    "trainy = y.head(int(0.8 * y.shape[0]))\n",
    "testX = X.tail(int(0.2 * X.shape[0])).reset_index(drop=True)\n",
    "testy = y.tail(int(0.2 * y.shape[0])).reset_index(drop=True)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 6,
   "id": "a543694e-96f5-4e1e-ac1c-17e2694f648b",
   "metadata": {
    "tags": []
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "\n",
      "---- Energy Regression Analysis ----\n",
      "\n",
      "1/6) Model Training\n",
      "> Renaming Features\n",
      "> Extracting Time Features\n",
      "> Transforming Categorical Features\n",
      "> Filling In Missing Values\n",
      "> Removing Constant Features\n",
      "> Selecting Features\n",
      "> Computing Reciprocals\n",
      "> Computing Interactions\n",
      "> Selecting Features\n",
      "> Transforming The Training Data\n",
      "> Training XGBoost\n",
      "3.14 Minutes\n",
      "2/6) Model Performance\n",
      "> Transforming The Testing Data\n",
      "6.2 Seconds\n",
      "3/6) Model Deployment\n",
      "> Transforming All The Data\n",
      "30.28 Seconds\n",
      "4/6) Model Indicators\n",
      "0.12 Seconds\n",
      "5/6) Model Prediction\n",
      "> Transforming The New Data\n",
      "3.86 Seconds\n",
      "6/6) Model Monitoring\n",
      "4.29 Seconds\n"
     ]
    }
   ],
   "source": [
    "# build the model\n",
    "print(\"\\n---- Energy Regression Analysis ----\\n\")\n",
    "model = Regression(name=\"Energy Regression Analysis\", frac=0.5)  # only use half the data for preprocessing\n",
    "try:\n",
    "    model.load()  # load the machine learning pipeline\n",
    "    predictions = model.predict(testX)\n",
    "except:\n",
    "    model.fit(trainX, trainy)  # build the machine learning pipeline\n",
    "    predictions = model.predict(testX)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 7,
   "id": "30b9ca50-06d6-49b7-848b-c0852e302909",
   "metadata": {
    "tags": []
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "RMSE: 2691.619506096193\n",
      "R2: 0.7408783084553288\n"
     ]
    }
   ],
   "source": [
    "# score the model\n",
    "rmse = mean_squared_error(\n",
    "    y_true=testy.iloc[:,0].to_numpy(),\n",
    "    y_pred=predictions,\n",
    "    squared=False,\n",
    ")\n",
    "r2 = r2_score(\n",
    "    y_true=testy.iloc[:,0].to_numpy(),\n",
    "    y_pred=predictions,\n",
    ")\n",
    "\n",
    "print(f\"RMSE: {rmse}\")\n",
    "print(f\"R2: {r2}\")"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 8,
   "id": "2226e286-b762-4265-a3de-de9a5561d5ec",
   "metadata": {
    "tags": []
   },
   "outputs": [],
   "source": [
    "# save the machine learning pipeline\n",
    "model.dump()"
   ]
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "sagemaker-distribution:Python",
   "language": "python",
   "name": "conda-env-sagemaker-distribution-py"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.8.17"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 5
}

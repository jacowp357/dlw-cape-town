# Practicals for Cape Town Deep Learning Workshops

## Setting up your Tensorflow environment

### 1. Anaconda

Download the Python 2.7 Anaconda distribution installer from [here](https://www.anaconda.com/downloads) and install Anaconda on your local system by following the instructions [here](https://conda.io/docs/user-guide/install/index.html).

The Anaconda distribution comes with most of the required Python packages installed by default.

Answer "yes" when prompted if you want to append the Anaconda installation directory to your PATH. To check that your PATH is set correctly execute the command `which python` and make sure it resolved to the Anaconda installed python binary. As an example, the output might look like

```
/home/john/anaconda2/bin/python
```

### 2. Tensorflow

We'll be using a conda enviroment to install Tensorflow. Create a conda environment by executing

```conda create -n tensorflow anaconda```

This will create the Tensorflow conda environment. The conda environment is a virtual environment in which packages may be installed without affecting other virtual environments. You may call your environment some different than `tensorflow` by passing a different argument to the `-n` option.

Activate the conda enivronment

```
source activate tensorflow
```

With your conda environment activated install Tensorflow as follows

```
pip install --ignore-installed --upgrade tensorflow
```

### 3. Jupyter Notebooks

The tutorials are provided in the form of Jupyter notebooks.

Jupyter comes installed with Anaconda by default. 

To start the notebooks run the command

```
jupyter-notebook
```

in the directory containing the practicals notebooks.

# Jupyter notebooks

This readme includes steps for setting up and running Jupyter notebooks.

## Running Jupyter notebooks in a Python Virtual Environment

1. Install basic packages:

   ```
   apt update && apt install -y sudo
   ```

2. Install Python3 with venv and pip:

   ```
   sudo apt install -y python3-pip python3-venv
   ```

3. Create a Python virtual environment and start it:

   ```
   python3 -m venv .venv # .venv is the destination path
   source .venv/bin/activate
   ```

4. Install Jupyter:
   ```
   pip3 install jupyter
   ```

5. Install `ipykernel` and register it as a kernel for Jupyter:
   ```
   pip3 install ipykernel
   python3 -m ipykernel install --user --name=jupyterenv --display-name "Python 3 (Jupyter notebook)"
   ```

   This step is required so that Jupyter uses Python from the virtual environment. Otherwise, `%pip install` steps
   in the notebook could fail with `error: externally-managed-environment`.

6. Run the notebook:
   ```
   jupyter notebook <my_notebook.ipynb>
   ```
   This will start the Jupyter server. Follow the console instructions to open the notebook in a browser.
   Use the `http://127.0.0.1:8888` link, rather than the `file:///` link, which requires the browser and
   notebook to be the same filesystem, which is not the case for WSL.  
   When the notebook starts, set the kernel to the one defined in step 5.

## Running Jupyter notebooks using system Python

1. Run steps 1 and 2 as above.

2. Install the jupyter package:
   ```
   sudo apt install jupyter
   ```

3. Run the notebook:
   ```
   jupyter notebook <my_notebook.ipynb>
   ```

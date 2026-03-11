# Clean Jupyter setup with venv

```bash
cd dev

mkdir j-notebook

cd j-notebook

# Create a venv
python -m venv .venv

# Activate the venv
.\.venv\Scripts\activate

# Install Jupyter ui + kernel support inside this venv
# no pip install notebook ipykernel
# Tip: Always prefer python -m pip ... over bare pip ... so you’re 100% sure the venv’s pip is used.
python -m pip install ipykernel notebook

# Verify installation, should see metadata (version, location)
python -m pip show ipykernel

# Register this venv as a Jupyter kernel
python -m ipykernel install --user --name "j-notebook-env" --display-name 'Python (J-notebook)'
# You now have a Jupyter kernel called: Python (J-notebook)
# Should see: Installed kernelspec j-notebook-env in C:\Users\gujin\AppData\Roaming\jupyter\kernels\j-notebook-env

# Launch Jupyter notebook
jupyter notebook

# Make sure using the correct kernel not global one
jupyter kernelspec list
# should see:
# Available kernels:
#  j-notebook-env    C:\Users\gujin\AppData\Roaming\jupyter\kernels\j-notebook-env
#  python3           C:\Users\gujin\AppData\Local\Programs\Python\Python314\share\jupyter\kernels\python3

```

Choose the correct kernel
Python (J-notebook) → this is the kernel you created when you ran
python -m ipykernel install --name "j-notebook-env" --display-name "Python (J-notebook-env)".
It points to the venv inside your J-Notebook project.

Python 3 (ipykernel) → this is a generic/default kernel that usually points to your global Python (or whichever interpreter Jupyter discovered first). It’s not tied to your project venv.

# How to install packages inside the notebook

- Inside a Jupyter notebook → prefer `%pip install pandas`
- In a terminal/PowerShell → prefer `python -m pip install pandas`
- Avoid plain pip install ... unless you’re sure which interpreter it targets.

Why %pip is better inside notebooks
%pip is a Jupyter magic that runs installation in the currently running kernel’s environment and updates the kernel’s sys.path correctly without restarting the kernel.

```bash
%pip install pandas numpy matplotlib
```

# copy‑pasteable starter kit

- verify you’re on the right kernel/venv
- safely install (or upgrade/pin) packages with %pip
- keep things reproducible with requirements.txt

```bash
# ✅ Environment sanity check — run this first in every new notebook

import sys, site, os, platform
from pathlib import Path

print("Python executable:", sys.executable)
print("Python version   :", platform.python_version())
print("Platform         :", platform.platform())
print("Site-packages    :", site.getsitepackages() if hasattr(site, "getsitepackages") else site.getusersitepackages())

# Optional: assert you're in the intended venv path keyword(s)
EXPECTED_KEYWORD = "J-Notebook\\venv\\Scripts"  # <-- change to match your venv path, or set to None to skip
if EXPECTED_KEYWORD and EXPECTED_KEYWORD not in sys.executable:
    raise RuntimeError(
        f"⚠️ This kernel is not using the expected environment.\n"
        f"Expected to find '{EXPECTED_KEYWORD}' in sys.executable.\n"
        f"Current: {sys.executable}\n"
        f"Use Kernel → Change Kernel to pick the right one."
    )
else:
    print("✅ Kernel is using the expected environment.")

```

# Export / Recreate Environment (for reproducibility)

Export currently installed packages (from this venv):

```bash
# Save a reproducible lock file (requirements.txt) next to your notebook
import subprocess, sys, pathlib
req_path = pathlib.Path("requirements.txt").resolve()
subprocess.run([sys.executable, "-m", "pip", "freeze", "-q"], check=True, stdout=open(req_path, "w", encoding="utf-8"))
print("✅ Wrote", req_path)
```

Recreate the environment later (from terminal or inside the notebook with %pip):

```bash
# In a new venv/kernel, run this to recreate the exact environment:
%pip install -r requirements.txt
```

# Quick “Data Science” Starter Cell

```bash
# Common imports + display config

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

pd.set_option("display.max_columns", 100)
pd.set_option("display.width", 120)
plt.style.use("seaborn-v0_8")

print("✅ Ready — pandas:", pd.__version__, "| numpy:", np.__version__)
```

---
title: "Build and Install Python Package with multiple directories referencing one another."
date: 2024-07-19
categories:
  - tutorials
tags:
  - python
  - project
  - setuptools 
---

### Build and Install Python Package with multiple directories referencing one another.

The problem I had was that I wanted to call a function from another directory in a file in another directory. To solve this, I had to create a python package and the following post is the way I solved it.

In this post, I will be taking you all through creating a python package with sub-packages which all can communicate with each other through absolute imports.

The processs of creating the package involves the following steps:
- Install setuptools
- Create a pyproject.toml
- Build / Install the package

I've summarized the entire process at the end of the article.

---

Before starting the walkthrough, I would like to point out the directory structure and what each of the python scripts contains.

```
root/
  pyproject.toml
  README.md
  
  project/
    __init__.py
    
    pkg1/
      t1.py
      __init__.py
    
    pkg2/
      t2.py
      __init__.py
```
All __init__.py files are empty and are present only to let us treat the folders as python modules.
t2.py contains some functions which does basic printing.
t1.py contains functions which will call the functions in t2.py

```python
# starter code for t2.py
def print_t2(x="t2"):
    print(f"{x} called")
```

```python 
# starter code for t1.py
from project.pkg2.t2 import print_t2

print_t2("t1")
```

As we can see t1.py tries to call a function defined in t2.py, this will result in an error.

One thing which I tried before fixing on my current approach,

### Relative imports -
basically instead of importing from `project.pkg2.t2`, I tried importing `..pkg2.t2`. This resulted in an error `ImportError: attempted relative import with no known parent package.`

---

### Step 1: Installing / Upgrading Setuptools
Now coming on to the main topic, we will start with making sure that setuptools is installed in our system/virtual environment. You can install/upgrade the package using the following command
`python -m pip install -U setuptools` or `pip install -U setuptools`.

To avoid messing up your base python environment, make sure to perform all this inside a virtual environment. A virtual environment can be created using the `conda create -n env_name python=3.8` or `python -m venv .venv`.

### Step 2: Creating a pyproject.toml
This is probably the most important step, I had to try a lot of different things to make it work. Hopefully, you will not have to waste so much time on this step. [Quickstart - setuptools documentation](https://setuptools.pypa.io/en/latest/userguide/quickstart.html)

Basically, in this step, we can use 3 different kinds of file -` pyproject.toml`, `setup.cfg`, `setup.py`. 
1. We can define all the configuration in a single `pyproject.toml` file for simple, static configuration. 
2. If you have any dynamic configuration which needs to be added use `setup.py`. 
More on this topic can be found in the official documentation. I highly recommend you read the documentation carefully to avoid wasting time like I did.
Now, let's see how to fill the pyproject.toml file to

```toml
# pyproject.toml - starting code indicating the backend to use
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"
``` 
This is the part of configuration which tells which backend needs to be used to build our packages.

```toml
# pyproject.toml - configuration for the project and other information.
[metadata]
name = "project"
version = "0.0.1"

[project]
name = "project"
version = "0.0.1"
authors = [
  { name="your_name", email="your_email" },
]
description = "Sample project for medium article"
readme = "README.md"
requires-python = ">=3.8"
```
You can also add metadata to the pyproject file which I believe to be optional. The project section in the pyproject file can also be defined in the setup.py. For simplicity, I've added it in the pyproject file itself.
Final configuration to add to the pyproject file is to point to the location of source files.

```toml
# Automatic detection of all the packages in the project folder. 
# Basically, it will add project, project.pkg2, project.pkg1 as valid packages.
[tool.setuptools.packages]
find = {}  # Scan the project directory with the default parameters
```
If your project directory structure follows the structure I've pointed out in the beginning, then setuptools can automatically infer the different packages, sub-packages in your project. Else you might have to provide the paths manually. This part of configuration can be moved to setup.py as per your convenience.
Finally, the entire pyproject.toml looks like this -

```toml 
# Complete pyproject.toml file 
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"

[metadata]
name = "project"
version = "0.0.1"

[tool.setuptools.packages]
find = {}  # Scan the project directory with the default parameters

[project]
name = "project"
version = "0.0.1"
authors = [
  { name="your_name", email="your_email" },
]
description = "Sample project for medium article"
readme = "README.md"
requires-python = ">=3.8"
``` 

### Step 3: Build and Install the package.
Now comes the part to create our own package. This involves 2 steps.
1. Build the package.
2. Install the package.

We will need to build and install the package everytime there is an update to the code base. To avoid this, we can use the convenient editable option pip install has, which let's us concentrate on development without having to continuously build the code base.

First, we will need to install/upgrade the python package build. Run the following command: pip install -U build to upgrade the build package.

#### Build/Install in Normal mode:
In Normal mode, we have to run install step everytime there has been any changes to the code base. As such, it is not ideal in development period.

The commands will be as follows:
1. `python -m build`
2. `python -m pip install .`

These two commands needs to be run from the root folder which contains the pyproject.toml file.
After you complete running the two commands, we can check whether our package has been installed successfully through the command 1. `python -c "import project"`.

#### Build/Install in Editable mode:
Editable mode is very useful when the project is in development phase.
The commands will be:
1. `python -m build`
2. `python -m pip install -e .`
check for success using 
1. `python -c "import project"`

While editable mode supports live code changes, make sure to install the packages once you are done with the development work .

#### Normal VS Editable Installation
Now, when I try to run the code, I get the expected output.
<div class="container">
<img src="https://kavya006.github.io/mm-portfolio/assets/images/posts/success1-packages-python.png" />
</div>
Since, I installed in editable mode, If I add another function in t2.py, lets say print_t5 which does the following

```python 
# complete t2.py 
def print_t2(x="t2"):
    print(f"{x} called")

def print_t5(x="t5"):
    print(f"{x} called 594859..")
```

And call this in t1.py, I see this output

<div class="container">
<img src="https://kavya006.github.io/mm-portfolio/assets/images/posts/success1-packages-python.png" />
</div>

I haven't passed any arguments to this function call and hence why we see the default printNow, let's try the same in normal mode. I added one more print function to t2.py, let's call it print_t6 and try to call this function in t1.py, we get the following error
<div class="container">
<img src="https://kavya006.github.io/mm-portfolio/assets/images/posts/error1-packages-python.png" />
</div>

When we install the pacakge in normal mode, we get an error when we update the code message.This error will be resolved, if we install the package using the same command `pip install .`

---

To summarize,
1. Install/Upgrade setuptools `pip install -U setuptools`
2. Create the pyproject.toml file in the root folder - [Complete File](https://gist.github.com/kavya006/e5ebfb80584c3d0a99becc16c0059fde)
3. Install/Upgrade the python package build: `pip install -U build`
4. Use build to package the project `python -m build`. Re-run this command everytime the configuration file (`pyproject.toml`) changes.
5. Install in editable mode for ease in development: `pip install -e .`

We are done with creating the python project. I hope this was helpful.

> **UPDATE 1 (06 Dec 2023):**
> An update with the latest setuptools `v69.0.2`
> 1. Adding [`metadata`] in pyproject.toml file is throwing a warning.
> 2. Also, not using the valid email address is throwing an error while building.
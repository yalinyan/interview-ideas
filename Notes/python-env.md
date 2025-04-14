Create a project
Before creating a virtual environment, let's create a folder for a Python project. Then change directories to the project root.

$ mkdir myproject
$ cd myproject


We'll use the Venv environment manager for local project installations of packages with pip.

Environment managers
Use an environment manager to create a virtual environment for project isolation.

Environment managers are needed because pip installs packages globally which can lead to conflicts (dependency hell). Other software languages (JavaScript or Ruby, for example) provide options to install packages locally (within a single project) as well as globally (system-wide for all projects). There are two Python tools for creating virtual environments and using different versions of packages. Venv is a built-in Python package. Virtualenv is a more powerful tool with additional features. These tools allow pip to install Python packages into a virtual environment with its own installation directories, that doesn’t share libraries with other virtual environments.

If you use Rye as an all-in-one tool, you won't need venv or virtualenv for environment management and you can install packages directly with Rye.

If you decide to install packages with pip and use venv, you should run brew pin python to prevent automatic Python upgrades. When Homebrew upgrades Python, virtual environments will break.

Assuming we're in the project directory myproject, we'll set up venv to eliminate the error: externally-managed-environment when using pip.

$ python -m venv ./env
$ source ./env/bin/activate


The python -m venv command creates a ./env that contains project configuration files and utilities.

Now you can install any Python package from the Python Package Index. Here we'll install the cowsay utility.

$ pip install cowsay
Collecting cowsay
Using cached cowsay-6.1-py3-none-any.whl.metadata (5.6 kB)
Using cached cowsay-6.1-py3-none-any.whl (25 kB)
Installing collected packages: cowsay
Successfully installed cowsay-6.1
Run Python
After installing a package within the virtual environment, you can use the Python interpreter interactively (the REPL or Read-Eval-Print Loop).

$ python
Python 3.12.1 (main, Jan  7 2024, 23:31:12) [Clang 16.0.3 ] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import cowsay
>>> cowsay.cow('Hello World')
___________
| Hello World |
  ===========
           \
            \
              ^__^
              (oo)\_______
              (__)\       )\/\
                  ||----w |
                  ||     ||
>>>
Enter quit() or type Control + D to exit the Python interpreter.

I recommend comparing the Homebrew-installed Python approach to Install Python with Rye, as I believe it's better to use a single tool to manage Python projects.


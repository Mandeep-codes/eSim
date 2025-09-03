# eSim 2.5 on Ubuntu 25.04: Installation Report and Fixes

## 1. Objective
To identify, document, and resolve the dependency and compatibility issues encountered while attempting to install the legacy eSim 2.5 application on a modern Ubuntu 25.04 operating system.

## 2. Methodology
The installation was approached by cloning the official Git repository. When the provided installation instructions failed, a manual, step-by-step debugging process was initiated. This involved attempting to run the Python-based installer, identifying each failure point, diagnosing the root cause, and applying patches to the source files to bypass the errors and proceed to the next issue.

## 3. Findings: A Chain of Installation Failures
The installation process failed at multiple, distinct stages, revealing a pattern of outdated dependencies and a lack of maintenance.

* **Bug #1: Outdated Documentation:** The `INSTALL` file is incorrect. It refers to a missing `install-eSim.sh` script, making the primary installation method unusable.

* **Bug #2: Incompatible Python Dependencies:** The `requirements.txt` file is severely outdated and incompatible with modern Python environments (Python 3.12).
    * **hdlparse:** This package relies on `use_2to3`, a Python 2 conversion feature that has been completely removed from modern `setuptools`.
    * **numpy & scipy:** These packages were pinned to old versions that are not compatible with and cannot be compiled on Python 3.12.

* **Bug #3: Conflict with Modern OS Protections:** The required installation methods conflict with modern Linux security and packaging standards.
    * **PEP 668:** Ubuntu's "externally-managed-environment" protection blocks the use of `sudo pip install` by default, requiring a system override.
    * **apt vs. pip Conflict:** The process attempts to use `pip` to manage packages (like `zipp`) that were originally installed by the system's `apt` package manager, causing an uninstall failure.

* **Bug #4: Failed Executable Creation:** The `setup.py` script successfully installs eSim as a library but fails to create the `esim` executable command, preventing the application from being run easily.

* **Bug #5: Fatal Crash on Launch:** When run directly from the source code, the application crashes immediately with a `segmentation fault`. This indicates a deep, low-level incompatibility between one of its core C++-based components (most likely the `PyQt5` GUI toolkit) and the libraries on the modern Ubuntu 25.04 OS.

## 4. Fixes Applied
To progress through the installation, one major class of issues was fixed: **the outdated Python dependencies**. This was achieved by modifying the `requirements.txt` file in several ways:
1.  The broken `hdlparse` dependency was disabled by commenting it out.
2.  The version pins for `numpy` and `scipy` were removed, allowing `pip` to install the latest compatible versions.
3.  The final `pip install` command required the `--break-system-packages` and `--ignore-installed` flags to overcome the OS protections and package manager conflicts.

## 5. Conclusion
eSim 2.5 is not directly installable on a modern Linux distribution like Ubuntu 25.04. While the Python dependency issues can be resolved with manual patching, a fatal low-level `segmentation fault` ultimately prevents the application from running, pointing to a fundamental incompatibility that would require significant code-level changes to fix.

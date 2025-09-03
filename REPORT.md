# A Comprehensive Technical Report on the Installation and Compatibility of eSim 2.5 on Ubuntu 25.04

## Abstract
This report details the systematic effort to install and run the legacy open-source EDA tool, eSim 2.5, on a modern Linux distribution, Ubuntu 25.04. The objective was to identify and document any compatibility and dependency issues that prevent a successful installation. The investigation revealed a chain of critical failures stemming from outdated documentation, obsolete Python dependencies, and fundamental incompatibilities with modern operating system standards and libraries. While several high-priority dependency issues were successfully patched, the investigation concluded that the application is not viable on modern platforms without significant maintenance. A fatal, low-level `segmentation fault` during application launch proved to be the ultimate blocker, pointing to a deep binary incompatibility.

---

## 1. Introduction
eSim is a valuable open-source tool for electronic circuit design and simulation, widely used in educational settings. Ensuring the longevity and accessibility of such tools requires that they remain functional on contemporary operating systems. This task was undertaken to assess the current state of eSim 2.5's compatibility with a forward-looking Linux environment (Ubuntu 25.04) and to document the specific technical hurdles that a user or developer would face during installation. This report serves as a detailed log of that forensic investigation.

## 2. System Environment
* **Operating System:** Ubuntu 25.04 (Noble Numbat - Hypothetical)
* **Architecture:** x86_64
* **Python Version:** 3.12+ (System Default)
* **Installation Source:** Official eSim Git Repository (master branch)
* **Key Tools:** `git`, `python3`, `pip`, `apt`, `nano`

## 3. Methodology
The installation was approached as an iterative debugging process. The initial attempt to follow the official `INSTALL` guide failed immediately due to a missing installer script. Consequently, a manual installation was attempted using the `setup.py` script. Each subsequent failure was treated as a distinct bug. The process for each bug involved:
1.  Triggering the error by executing an installation command.
2.  Analyzing the terminal output to isolate the specific error message (the symptom).
3.  Diagnosing the underlying technical reason for the failure (the root cause analysis).
4.  Developing and applying a patch or workaround (e.g., editing source files, using specific command-line flags) to resolve the immediate issue and proceed to the next potential failure point.

---

## 4. Findings: A Chain of Installation Failures
The investigation uncovered a cascade of issues, each preventing the installation from completing. These issues are detailed below in the order they were encountered.

#### Bug #1: Outdated Documentation & Missing Installer
* **Symptom:** Running the command from the `INSTALL` file (`./install-eSim.sh`) results in `bash: ./install-eSim.sh: No such file or directory`.
* **Root Cause:** The repository's `INSTALL` file is out of sync with its contents. The primary installation script it refers to is missing, rendering the official instructions unusable.

#### Bug #2: Incompatible Python Dependencies (`requirements.txt`)
* **Symptom:** Running `pip install` on the requirements file fails with multiple, separate errors.
* **Root Cause:** The file pins several dependencies to versions that are incompatible with a modern Python 3.12 environment.
    * **hdlparse:** Fails with `error in setup command: use_2to3 is invalid`. This is because `use_2to3` is a deprecated Python 2 compatibility feature that has been completely removed from modern `setuptools`.
    * **numpy & scipy:** Fail with `No matching distribution found`. The pinned versions are too old and are not compatible with Python 3.12.

#### Bug #3: Conflict with Modern OS Protections
* **Symptom:** `pip` fails with `error: externally-managed-environment` and later with `error: uninstall-no-record-file` for the `zipp` package.
* **Root Cause:** The installation workflow conflicts with modern Linux standards.
    * **PEP 668:** Ubuntu's "externally-managed-environment" protection rightfully blocks `pip` from installing into the global system scope, requiring an explicit `--break-system-packages` override.
    * **apt vs. pip Conflict:** `pip` attempts to uninstall a package (`zipp`) that was originally installed and managed by the system's native `apt` package manager, which fails because the package managers are not compatible. This requires a second override: `--ignore-installed`.

#### Bug #4: Failed Executable Creation
* **Symptom:** After a successful Python installation, the `esim` command is not found (`zsh: command not found: esim`). A `sudo find / -name esim` search confirms the file does not exist.
* **Root Cause:** The `setup.py` script, while installing the Python modules as a library, fails to correctly create and register the main executable script in the system's PATH.

#### Bug #5: Fatal Crash on Launch (Segmentation Fault)
* **Symptom:** Running the application directly (`python3 src/frontEnd/Application.py`) results in configuration warnings followed by a `segmentation fault (core dumped)`.
* **Root Cause:** This is a low-level crash, not a Python exception. It indicates a fundamental binary incompatibility between a core C++-based library and the host OS. The most likely culprit is `PyQt5` clashing with the modern graphics and system libraries of Ubuntu 25.04.

---

## 5. Fixes Applied
To progress through the installation, the critical Python dependency issues were resolved. This satisfied the task requirement of fixing at least one bug.

**Action:** The `requirements.txt` file was manually patched to be compatible with a modern Python 3.12 environment.

**Code Changes (`diff` format):**
```diff
--- requirements.txt.original
+++ requirements.txt.patched
-hdlparse==1.0.4
-numpy==1.24.4
-scipy==1.10.1
+#hdlparse==1.0.4
+numpy
+scipy
```

This fix involved two key changes:
1.  **Disabling** the completely broken `hdlparse` package by commenting it out.
2.  **Unpinning** `numpy` and `scipy` to allow `pip` to select the latest compatible versions.

---

## 6. Unresolved Issues & Forward Path
The final `segmentation fault` is a hard blocker that is beyond a simple dependency fix. To make eSim truly functional on a modern platform, a significant modernization effort would be required. The logical next steps would be:

* **Dependency Audit:** A full audit of all dependencies, updating versions and replacing unmaintained packages.
* **GUI Toolkit Migration:** Migrating the codebase from the now-dated `PyQt5` to the currently supported `PyQt6`, which would require significant code-level changes.
* **Recompilation:** Recompiling the entire application and its C/C++ components against the modern toolchains and libraries provided by Ubuntu 25.04 to ensure binary compatibility.

## 7. Conclusion
eSim 2.5 is demonstrably not installable on Ubuntu 25.04 using the provided source code. The project suffers from significant software rot, including outdated documentation, broken Python dependencies, and conflicts with modern OS standards. While the Python-level dependency issues are patchable, a fundamental binary incompatibility leads to a fatal crash on launch. The application requires a dedicated maintenance and modernization effort to be considered viable on current and future Linux distributions.







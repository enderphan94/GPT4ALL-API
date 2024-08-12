# GPT4ALL-API
Troubleshooting for MacOS. It works with M2 pro

# Python GPT4All

This package contains a set of Python bindings around the `llmodel` C-API.

Package on PyPI: https://pypi.org/project/gpt4all/

## Documentation
https://docs.gpt4all.io/gpt4all_python.html

## Installation

The easiest way to install the Python bindings for GPT4All is to use pip:

```
pip install gpt4all
```

This will download the latest version of the `gpt4all` package from PyPI.

## Local Build

As an alternative to downloading via pip, you may build the Python bindings from source.

### Prerequisites

You will need a compiler. On Windows, you should install Visual Studio with the C++ Development components. On macOS, you will need the full version of Xcode&mdash;Xcode Command Line Tools lacks certain required tools. On Linux, you will need a GCC or Clang toolchain with C++ support.

On Windows and Linux, building GPT4All with full GPU support requires the [Vulkan SDK](https://vulkan.lunarg.com/sdk/home) and the latest [CUDA Toolkit](https://developer.nvidia.com/cuda-downloads).

### Building the python bindings

1. Clone GPT4All and change directory:
```
git clone --recurse-submodules https://github.com/nomic-ai/gpt4all.git
cd gpt4all/gpt4all-backend
```

2. Build the backend.

If you are using Windows and have Visual Studio installed:
```
cmake -B build
cmake --build build --parallel --config RelWithDebInfo
```

For all other platforms:
```
cmake -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo
cmake --build build --parallel
```

`RelWithDebInfo` is a good default, but you can also use `Release` or `Debug` depending on the situation.

2. Install the Python package:
```
cd ../gpt4all-bindings/python
pip install -e .
```

## Usage

Test it out! In a Python script or console:

```python
from gpt4all import GPT4All
model = GPT4All("orca-mini-3b-gguf2-q4_0.gguf")
output = model.generate("The capital of France is ", max_tokens=3)
print(output)
```


GPU Usage
```python
from gpt4all import GPT4All
model = GPT4All("orca-mini-3b-gguf2-q4_0.gguf", device='gpu') # device='amd', device='intel'
output = model.generate("The capital of France is ", max_tokens=3)
print(output)
```

## Troubleshooting a Local Build
- If you're on Windows and have compiled with a MinGW toolchain, you might run into an error like:
  ```
  FileNotFoundError: Could not find module '<...>\gpt4all-bindings\python\gpt4all\llmodel_DO_NOT_MODIFY\build\libllmodel.dll'
  (or one of its dependencies). Try using the full path with constructor syntax.
  ```
  The key phrase in this case is _"or one of its dependencies"_. The Python interpreter you're using
  probably doesn't see the MinGW runtime dependencies. At the moment, the following three are required:
  `libgcc_s_seh-1.dll`, `libstdc++-6.dll` and `libwinpthread-1.dll`. You should copy them from MinGW
  into a folder where Python will see them, preferably next to `libllmodel.dll`.

- Note regarding the Microsoft toolchain: Compiling with MSVC is possible, but not the official way to
  go about it at the moment. MSVC doesn't produce DLLs with a `lib` prefix, which the bindings expect.
  You'd have to amend that yourself.

### If it returns the following error of cuda
  ```
  Traceback (most recent call last):
  File "/Users/enderphan/AI/gpt4all/gpt4all-bindings/python/main.py", line 1, in <module>
    from gpt4all import GPT4All
  File "/Users/enderphan/AI/gpt4all/gpt4all-bindings/python/gpt4all/__init__.py", line 1, in <module>
    from .gpt4all import CancellationError as CancellationError, Embed4All as Embed4All, GPT4All as GPT4All
  File "/Users/enderphan/AI/gpt4all/gpt4all-bindings/python/gpt4all/gpt4all.py", line 23, in <module>
    from ._pyllmodel import (CancellationError as CancellationError, EmbCancelCallbackType, EmbedResult as EmbedResult,
  File "/Users/enderphan/AI/gpt4all/gpt4all-bindings/python/gpt4all/_pyllmodel.py", line 48
    cudalib   = fr"bin\cudart64_{rtver.replace(".", "")}.dll"
                                                ^
  SyntaxError: f-string: unmatched '('
  ```
  To resolve this issue, you should ensure that all parentheses are correctly matched. Here’s a possible corrected version of the line:

  ```cudalib = fr"bin\cudart64_{rtver.replace('.', '')}.dll"```

  Notice the change in the replace method call where I’ve replaced the double quotes with single quotes for the argument of replace('.').

### If it still returns another error of ARM64
  ```
  Traceback (most recent call last):
  File "/Users/enderphan/AI/gpt4all/gpt4all-bindings/python/main.py", line 1, in <module>
    from gpt4all import GPT4All
  File "/Users/enderphan/AI/gpt4all/gpt4all-bindings/python/gpt4all/__init__.py", line 1, in <module>
    from .gpt4all import CancellationError as CancellationError, Embed4All as Embed4All, GPT4All as GPT4All
  File "/Users/enderphan/AI/gpt4all/gpt4all-bindings/python/gpt4all/gpt4all.py", line 23, in <module>
    from ._pyllmodel import (CancellationError as CancellationError, EmbCancelCallbackType, EmbedResult as EmbedResult,
  File "/Users/enderphan/AI/gpt4all/gpt4all-bindings/python/gpt4all/_pyllmodel.py", line 37, in <module>
    raise RuntimeError(textwrap.dedent("""\
  RuntimeError: Running GPT4All under Rosetta is not supported due to CPU feature requirements.
  Please install GPT4All in an environment that uses a native ARM64 Python interpreter.
  ```
  The error message you’re encountering indicates that the GPT4All library is trying to run under Rosetta, which is an emulation layer that allows x86 applications to run on ARM-based Macs (such as those with Apple Silicon). However, GPT4All requires specific    CPU features that are not supported when running under Rosetta. The solution is to ensure that you’re using a native ARM64 Python environment.

  Here’s how you can resolve this issue:
  
1. Check Your Python Version:

  Ensure that you’re using a version of Python that is compiled for ARM64. You can check this by running:

  ```
  python3 -c "import platform; print(platform.machine())"
  ```
  
  If it returns arm64, you’re using the correct version. If it returns x86_64, you’re running an x86 version of Python under Rosetta.
  
2. Install a Native ARM64 Version of Python:

  If you’re running an x86 version, you’ll need to install a native ARM64 version of Python. You can do this using Homebrew:
  
  ```
  arch -arm64 brew install python@3.11
  ```

3. Switch to the ARM64 Python Version:

After installing the ARM64 version of Python, you need to ensure that your terminal uses this version by default. You can specify the path to the ARM64 Python binary directly:

```
/opt/homebrew/bin/python3 --version
```

Or, create a virtual environment using the ARM64 Python:

```
/opt/homebrew/bin/python3 -m venv myenv
source myenv/bin/activate
```
4. Reinstall GPT4All in the ARM64 Environment:

```
pip install gpt4all
```

  

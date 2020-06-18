# Build from Source

## Prepare the Python Virtual Environment

1. install miniconda

2. update conda 

```
$ conda update conda
$ conda install nb_conda_kernels
```

3. create a virtual environment

    - check the available python version: `$ conda search "^python$"`
    - install missing packages: `$ conda install -c anaconda ncurses`
    - create virtual environment: `$ conda create -n tf-python2 python=2.7 anaconda`

4. activate the virtual environment

```
$ source activate tf-python2
```

## Install Python libraries

```
$ pip install --upgrade pip
$ pip install -U --user pip six numpy wheel mock
$ pip install -U --user keras_applications==1.0.5 --no-deps
$ pip install -U --user keras_preprocessing==1.0.3 --no-deps
$ pip install enum34
```

## Install Bazel

  - step: https://docs.bazel.build/versions/master/install-redhat.html

## Build from Source
  - code: 
  ```
  $ git clone https://github.com/feihugis/tensorflow.git
  $ git remote add upstream https://github.com/tensorflow/tensorflow.git
  ```

  - run test: `$ asdf`

  - configure: `$ ./configure`

  - build
  ```
  $ bazel build -c dbg //tensorflow/tools/pip_package:build_pip_package
  $ ./bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
  $ pip install /tmp/tensorflow_pkg/tensorflow-version-cp27-cp27mu-linux_x86_64.whl
  ```

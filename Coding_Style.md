- C++
`clang-format tensorflow/core/kernels/data/matching_files_dataset_op.cc --style=google > ~/Desktop/my_cc_file.cc`

- Python
    - `pip install pylint`
    - `wget -O /tmp/pylintrc https://raw.githubusercontent.com/tensorflow/tensorflow/master/tensorflow/tools/ci_build/pylintrc`
    - `pylint --rcfile=/tmp/pylintrc tensorflow/python/data/keops/dataset_ops.py`

- BUILD
`buildifier tensorflow/tensorflow/core/kernels/data/BUILD`

- Prepare Python2 environment
```
virtualenv --python=python2.7 tf-python2 
```

- Run benchmark test
```
bazel test //tensorflow/python/data/kernel_tests:matching_files_dataset_op_test --test_arg=--benchmarks=all
```

- Run test
```
bazel test //tensorflow/contrib/data/python/kernel_tests/serialization:matching_files_dataset_serialization_test
```

```
bazel test //tensorflow/python/data/kernel_tests:matching_files_dataset_op_test 
```

- Run a certain test function in a test file
```
bazel test --test_filter=DivNoNanTest.testBasic //tensorflow/python:math_ops_test
```

- Run test with Python 2
```
bazel test --python_path=/Users/fei/venv-py/tf-python2/bin/python //tensorflow/python/data/kernel_tests:matching_files_dataset_op_test
```

- Pylint check
```
~/venv-py/tf-python2/bin/pylint --rcfile=/tmp/pylintrc tensorflow/python/data/kernel_tests/matching_files_dataset_op_test.py
```

- Run test with Address Sanitizer check
```
bazel test //tensorflow/core/kernels/data:name_utils_test --copt=-fsanitize=address --linkopt=-fsanitize=address --copt=-fno-omit-frame-pointer --copt=-O1
```

- Template for bazel plugin
```
--runs_per_test=100
--copt=-fsanitize=address
--linkopt=-fsanitize=address
--copt
-Wno-macro-redefined
```
-- resource: https://github.com/bazelbuild/bazel/issues/6932
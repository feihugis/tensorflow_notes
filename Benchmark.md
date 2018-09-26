- Run benchmark test
```
bazel test //tensorflow/python/data/kernel_tests:matching_files_dataset_op_test --test_arg=--benchmarks=all
```
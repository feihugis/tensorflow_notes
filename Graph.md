- tensorflow/core/graph/graph.h
- tensorflow/core/framework/op.h
- tensorflow/core/graph/graph_partition.h

Session:
    - DirectSession::RunInternal

GraphView: tensorflow::GraphView


Node:
- tensorflow::NodeItem



Edge:
- hold the data/input/output
- control-flow or data-flow


Grappler:

- Example: [tf.data] [Mechanism for trading determinism for performance](https://github.com/tensorflow/tensorflow/commit/43b6f6c0ebb032c17ed5382150ba989761e036b7)
- tensorflow/core/grappler/optimizers/data/make_sloppy.h


Map:
[tf.py_func](https://www.tensorflow.org/api_docs/python/tf/py_func)


Function Graph

Resourve_variable

Gradient: `tensorflow/python/ops/gradients_impl.py`

ControlFlow: `tensorflow/python/ops/control_flow_ops.py`



`tensorflow/core/grappler/costs/op_level_cost_estimator.cc:363] Missing accurate estimator for op: OneShotIterator`

`tensorflow/core/common_runtime/graph_execution_state.cc:511`
`tensorflow/core/common_runtime/optimization_registry.cc:37`
`tensorflow/core/common_runtime/graph_execution_state.cc:224`
`tensorflow/core/common_runtime/executor.cc:1662`
`tensorflow/core/common_runtime/executor.cc:1806`

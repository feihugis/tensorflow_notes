`tensorflow/python/ops/parallel_for/pfor.py`

"""Implementation of rewrite of parallel-for loops.

  This class takes a DAG or a set of DAGs representing the body of a
  parallel-for loop, and adds new operations to the graph that implements
  functionality equivalent to running that loop body for a specified number of
  iterations. This new set of nodes may or may not use a tensorflow loop
  construct.

  The process of conversion does not delete or change any existing operations.
  It only adds operations that efficiently implement the equivalent
  functionality. We refer to the added ops as "converted ops".

  The conversion process uses a simple greedy heuristic. It walks the loop body
  and tries to express the functionality of running each node in a loop with a
  new set of nodes. When converting an op several cases are possible:
  - The op is not inside the loop body. Hence it can be used as is.
  - The op does not depend on the iteration number and is stateless. In this
    case, it can be used as is.
  - The op is not stateful, and depends on iteration number only through control
    dependencies. In this case, we can create a single op with same inputs and
    attributes, but with "converted" control dependencies.
  - The op is not stateful, and all its inputs are loop invariant. In this
    case, similar to above, we can create a single op with same inputs and
    attributes, but with "converted" control dependencies.
  - The op is stateful or at least one of the inputs is not loop invariant. In
    this case, we run the registered converter for that op to create a set of
    converted ops. All nodes in the set will have converted control dependencies
    corresponding to control dependencies of the original op. If the op returned
    multiple outputs, "converted outputs" could be produced by different ops in
    this set.
  """
# [Motivation](https://github.com/tensorflow/tensorflow/pull/24349#issuecomment-447164523)

The goal would be to develop C++ infrastructure for testing tf.data kernel implementations in C++. Currently, the C++ tf.data kernels are only tested through Python bindings which are not fine-grained enough and some public C++ APIs are not always tested.

The flow of testing the C++ API would be:

* create an instance of the dataset op 
* invoke the Compute method which takes input arguments and produces `DatasetBase` object wrapped in a `variant`
* test the public API of the `DatasetBase` object, including the `MakeIterator` method which produces `IteratorBase` object
* test the public API of the `IteratorBase` object

# Example

* `tensorflow/core/kernels/sparse_add_op_test.cc`
* `tensorflow/core/kernels/range_sampler_test.cc`
* `tensorflow/core/kernels/ops_testutil.h/tensorflow::OpsTestBase`
* `tensorflow/core/graph/testlib.cc`
* `tensorflow/core/kernels/ragged_tensor_to_sparse_kernel_test.cc`

# Note

The python package needs to be built by `bazel build -c dbg //tensorflow/tools/pip_package:build_pip_package` 

# Set breakpoint
```shell
breakpoint set -n ExecutorState::Process
breakpoint set -n FunctionLibraryRuntimeImpl::FunctionLibraryRuntimeImpl
breakpoint set -n OneShotIteratorOp::OneShotIteratorOp
breakpoint set -n RangeDatasetOp::MakeDataset
breakpoint set -n DatasetBase::MakeIterator
```

# Python script
```python
import tensorflow as tf
dataset = tf.data.Dataset.range(0,10,2)
iterator = dataset.make_one_shot_iterator()
sess = tf.Session()
print(sess.run(iterator.get_next()))
```

# Log

```C++
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
  * frame #0: 0x000000010c7694b4 _pywrap_tensorflow_internal.so`tensorflow::data::(anonymous namespace)::OneShotIteratorOp::OneShotIteratorOp(this=0x00007fa3a8406760, ctx=0x00007ffeea7bcbf8) at iterator_ops.cc:862
    
    frame #1: 0x000000010c76948f _pywrap_tensorflow_internal.so`tensorflow::data::(anonymous namespace)::$_11::operator(this=0x00007fa3a86c53a0, context=0x00007ffeea7bcbf8)(tensorflow::OpKernelConstruction*) const at iterator_ops.cc:1306
    
    frame #2: 0x000000010c769458 _pywrap_tensorflow_internal.so`tensorflow::data::(anonymous namespace)::$_11::__invoke(context=0x00007ffeea7bcbf8) at iterator_ops.cc:1306
    
    frame #3: 0x0000000108caed4e _pywrap_tensorflow_internal.so`tensorflow::kernel_factory::OpKernelRegistrar::OpKernelRegistrar(this=0x00007fa3a86c53a0, context=0x00007ffeea7bcbf8)(tensorflow::OpKernelConstruction*))::PtrOpKernelFactory::Create(tensorflow::OpKernelConstruction*) at op_kernel.h:1385
    
    frame #4: 0x000000013a9cc76f libtensorflow_framework.so`tensorflow::CreateOpKernel(device_type=(type_ = "CPU"), device=0x00007fa3a81dcf60, allocator=0x00007fa3aa7d9090, flib=0x00007fa3a812d350, node_def=0x00007fa3aa2a54f0, graph_def_version=27, kernel=0x00007fa3aaca45a0) at op_kernel.cc:1300
    
    frame #5: 0x000000013affd84c libtensorflow_framework.so`tensorflow::CreateNonCachedKernel(device=0x00007fa3a81dcf60, flib=0x00007fa3a812d350, ndef=0x00007fa3aa2a54f0, graph_def_version=27, kernel=0x00007fa3aaca45a0) at executor.cc:2763
    
    frame #6: 0x000000013b03c3a8 libtensorflow_framework.so`tensorflow::FunctionLibraryRuntimeImpl::CreateKernel(this=0x00007fa3a812d350, ndef=0x00007fa3aa2a54f0, lib_def=0x00007fa3aa22f0b0, kernel=0x00007fa3aaca45a0) at function.cc:538
    
    frame #7: 0x000000013b03bebb libtensorflow_framework.so`tensorflow::FunctionLibraryRuntimeImpl::CreateKernel(this=0x00007fa3a812d350, ndef=0x00007fa3aa2a54f0, kernel=0x00007fa3aaca45a0) at function.cc:515
    
    frame #8: 0x00000001141ee19b _pywrap_tensorflow_internal.so`tensorflow::DirectSession::CreateExecutors(this=0x00007ffeea7bdc38, kernel=0x00007fa3aaca45a0)::$_10::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const::'lambda'(tensorflow::OpKernel**)::operator()(tensorflow::OpKernel**) const at direct_session.cc:1264
    
    frame #9: 0x00000001141ee147 _pywrap_tensorflow_internal.so`tensorflow::Status std::__1::__invoke_void_return_wrapper<tensorflow::Status>::__call<tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_10::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const::'lambda'(tensorflow::OpKernel**)&, tensorflow::OpKernel**>(tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_10::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const::'lambda'(tensorflow::OpKernel**)&&&, tensorflow::OpKernel**&&) [inlined] decltype(__f=0x00007ffeea7bdc38, __args=0x00007ffeea7bd928)::$_10::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const::'lambda'(tensorflow::OpKernel**)&>(fp)(std::__1::forward<tensorflow::OpKernel**>(fp0))) std::__1::__invoke<tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_10::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const::'lambda'(tensorflow::OpKernel**)&, tensorflow::OpKernel**>(tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_10::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const::'lambda'(tensorflow::OpKernel**)&&&, tensorflow::OpKernel**&&) at type_traits:4323
    
    frame #10: 0x00000001141ee123 _pywrap_tensorflow_internal.so`tensorflow::Status std::__1::__invoke_void_return_wrapper<tensorflow::Status>::__call<tensorflow::DirectSession::CreateExecutors(__args=0x00007ffeea7bdc38, __args=0x00007ffeea7bd928)::$_10::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const::'lambda'(tensorflow::OpKernel**)&, tensorflow::OpKernel**>(tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_10::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const::'lambda'(tensorflow::OpKernel**)&&&, tensorflow::OpKernel**&&) at __functional_base:318
    
    frame #11: 0x00000001141ee010 _pywrap_tensorflow_internal.so`std::__1::__function::__func<tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_10::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const::'lambda'(tensorflow::OpKernel**), std::__1::allocator<tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_10::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const::'lambda'(tensorflow::OpKernel**)>, tensorflow::Status (tensorflow::OpKernel**)>::operator(this=0x00007ffeea7bdc30, __arg=0x00007ffeea7bd928)(tensorflow::OpKernel**&&) at functional:1562
    
    frame #12: 0x000000013a9e4ab2 libtensorflow_framework.so`std::__1::function<tensorflow::Status (tensorflow::OpKernel**)>::operator(this=0x00007ffeea7bdc30, __arg=0x00007fa3aaca45a0)(tensorflow::OpKernel**) const at functional:1921
    
    frame #13: 0x000000013a9e429b libtensorflow_framework.so`tensorflow::OpSegment::FindOrCreate(this=0x00007fa3a81dd048, session_handle="direct", node_name="OneShotIterator", kernel=0x00007fa3aaca45a0, create_fn=0x00007ffeea7bdc30)>) at op_segment.cc:52
    
    frame #14: 0x00000001141ecfab _pywrap_tensorflow_internal.so`tensorflow::DirectSession::CreateExecutors(this=0x00007fa3aa7e15a8, ndef=0x00007fa3aa2a54f0, kernel=0x00007fa3aaca45a0)::$_10::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const at direct_session.cc:1269
    
    frame #15: 0x00000001141ece27 _pywrap_tensorflow_internal.so`tensorflow::Status std::__1::__invoke_void_return_wrapper<tensorflow::Status>::__call<tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_10&, tensorflow::NodeDef const&, tensorflow::OpKernel**>(tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_10&&&, tensorflow::NodeDef const&&&, tensorflow::OpKernel**&&) [inlined] decltype(__f=0x00007fa3aa7e15a8, __args=0x00007fa3aa2a54f0, __args=0x00007ffeea7bdd88)::$_10&>(fp)(std::__1::forward<tensorflow::NodeDef const&, tensorflow::OpKernel**>(fp0))) std::__1::__invoke<tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_10&, tensorflow::NodeDef const&, tensorflow::OpKernel**>(tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_10&&&, tensorflow::NodeDef const&&&, tensorflow::OpKernel**&&) at type_traits:4323
    
    frame #16: 0x00000001141ecdf7 _pywrap_tensorflow_internal.so`tensorflow::Status std::__1::__invoke_void_return_wrapper<tensorflow::Status>::__call<tensorflow::DirectSession::CreateExecutors(__args=0x00007fa3aa7e15a8, __args=0x00007fa3aa2a54f0, __args=0x00007ffeea7bdd88)::$_10&, tensorflow::NodeDef const&, tensorflow::OpKernel**>(tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_10&&&, tensorflow::NodeDef const&&&, tensorflow::OpKernel**&&) at __functional_base:318
    
    frame #17: 0x00000001141eccd0 _pywrap_tensorflow_internal.so`std::__1::__function::__func<tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_10, std::__1::allocator<tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_10>, tensorflow::Status (tensorflow::NodeDef const&, tensorflow::OpKernel**)>::operator(this=0x00007fa3aa7e15a0, __arg=0x00007fa3aa2a54f0, __arg=0x00007ffeea7bdd88)(tensorflow::NodeDef const&, tensorflow::OpKernel**&&) at functional:1562
    
    frame #18: 0x000000010cb2d6c1 _pywrap_tensorflow_internal.so`std::__1::function<tensorflow::Status (tensorflow::NodeDef const&, tensorflow::OpKernel**)>::operator(this=0x00007fa3aa7e15a0, __arg=0x00007fa3aa2a54f0, __arg=0x00007fa3aaca45a0)(tensorflow::NodeDef const&, tensorflow::OpKernel**) const at functional:1921
    
    frame #19: 0x000000013affcd11 libtensorflow_framework.so`tensorflow::(anonymous namespace)::ExecutorImpl::Initialize(this=0x00007fa3aa7e1580) at executor.cc:620
    
    frame #20: 0x000000013affc56b libtensorflow_framework.so`tensorflow::NewLocalExecutor(params=0x00007ffeea7c1950, graph=unique_ptr<const tensorflow::Graph, std::__1::default_delete<const tensorflow::Graph> > @ 0x00007ffeea7be7e0, executor=0x00007ffeea7be7f0) at executor.cc:2749
    
    frame #21: 0x000000013b035615 libtensorflow_framework.so`tensorflow::(anonymous namespace)::DefaultExecutorRegistrar::Factory::NewExecutor(this=0x00007fa3a8238300, params=0x00007ffeea7c1950, graph=unique_ptr<const tensorflow::Graph, std::__1::default_delete<const tensorflow::Graph> > @ 0x00007ffeea7bea18, out_executor=0x00007fa3aa264e98) at executor.cc:2785
    
    frame #22: 0x000000013b03828d libtensorflow_framework.so`tensorflow::NewExecutor(executor_type="", params=0x00007ffeea7c1950, graph=unique_ptr<const tensorflow::Graph, std::__1::default_delete<const tensorflow::Graph> > @ 0x00007ffeea7bf068, out_executor=0x00007fa3aa264e98) at executor_factory.cc:82
    
    frame #23: 0x00000001141b7d77 _pywrap_tensorflow_internal.so`tensorflow::DirectSession::CreateExecutors(this=0x00007fa3aa7dee70, callable_options=0x00007ffeea7c4a48, out_executors_and_keys=0x00007ffeea7c1fe8, out_func_info=0x00007ffeea7c1fe0, run_state_args=0x00007ffeea7c5368) at direct_session.cc:1296
    
    frame #24: 0x00000001141a4439 _pywrap_tensorflow_internal.so`tensorflow::DirectSession::GetOrCreateExecutors(this=0x00007fa3aa7dee70, inputs=(ptr_ = "", len_ = 0), outputs=(ptr_ = "IteratorGetNext:0", len_ = 1), target_nodes=(ptr_ = "", len_ = 0), executors_and_keys=0x00007ffeea7c53a0, run_state_args=0x00007ffeea7c5368) at direct_session.cc:1429
    
    frame #25: 0x000000011419e758 _pywrap_tensorflow_internal.so`tensorflow::DirectSession::Run(this=0x00007fa3aa7dee70, run_options=0x00007ffeea7c60d8, inputs=size=0, output_names=size=1, target_nodes=size=0, outputs=0x00007ffeea7c6118 size=1, run_metadata=0x00007ffeea7c6080) at direct_session.cc:749
    
    frame #26: 0x0000000108d12426 _pywrap_tensorflow_internal.so`tensorflow::SessionRef::Run(this=0x00007fa3aa7df5a0, run_options=0x00007ffeea7c60d8, inputs=size=0, output_tensor_names=size=1, target_node_names=size=0, outputs=0x00007ffeea7c6118 size=1, run_metadata=0x00007ffeea7c6080) at session_ref.cc:427
    
    frame #27: 0x000000010947baef _pywrap_tensorflow_internal.so`TF_Run_Helper(session=0x00007fa3aa7df5a0, handle=0x0000000000000000, run_options=0x0000000000000000, input_pairs=size=0, output_tensor_names=size=1, c_outputs=0x00007ffeea7c68d0, target_oper_names=size=0, run_metadata=0x0000000000000000, status=0x00007fa3a540a460) at c_api.cc:787
    
    frame #28: 0x0000000109493530 _pywrap_tensorflow_internal.so`::TF_SessionRun(session=0x00007fa3aa7df570, run_options=0x0000000000000000, inputs=0x0000000000000000, input_values=0x00007ffeea7c6970, ninputs=0, outputs=0x00007fa3a54e4000, output_values=0x00007ffeea7c68d0, noutputs=1, target_opers=0x0000000000000000, ntargets=0, run_metadata=0x0000000000000000, status=0x00007fa3a540a460) at c_api.cc:2636
    
    frame #29: 0x0000000108d06cbf _pywrap_tensorflow_internal.so`tensorflow::TF_SessionRun_wrapper_helper(session=0x00007fa3aa7df570, handle=0x0000000000000000, run_options=0x0000000000000000, inputs=size=0, input_ndarrays=size=0, outputs=size=1, targets=size=0, run_metadata=0x0000000000000000, out_status=0x00007fa3a540a460, py_outputs=0x00007ffeea7c7cf8 size=0) at tf_session_helper.cc:407
    
    frame #30: 0x0000000108d080dd _pywrap_tensorflow_internal.so`tensorflow::TF_SessionRun_wrapper(session=0x00007fa3aa7df570, run_options=0x0000000000000000, inputs=size=0, input_ndarrays=size=0, outputs=size=1, targets=size=0, run_metadata=0x0000000000000000, out_status=0x00007fa3a540a460, py_outputs=0x00007ffeea7c7cf8 size=0) at tf_session_helper.cc:450
    ```
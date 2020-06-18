- https://github.com/tensorflow/tensorflow/issues/13101: the running process using sudo gdb -p <PID>, then run thread apply all bt to collect all stack traces
- Debug log: `os.environ['TF_CPP_MIN_VLOG_LEVEL']='4'`

- LLDB:
    - set a breakpoint by a function name: `breakpoint set --method ComputeGradient`
    - set a breakpoint at a certain line in a file: `breakpoint set --file executor.cc --line 1807`
    - check the value of a variable: `frame variable --format x  state`
    - run expression: `p run_options_proto.DebugString()`
    - step into: `thread step-in`

# Debug TensorFlow DatasetOp C++

```shell
$ cd /private/var/tmp/_bazel_fei/5dc7b372a7c45427ff30500d3e22fe26/execroot/org_tensorflow/bazel-out/darwin-dbg/bin/tensorflow/core/kernels/data/range_dataset_op_test.runfiles/org_tensorflow/tensorflow/core/kernels/data
$ lldb range_dataset_op_test
```

# Debug TensorFlow RangeDataset

```python
import tensorflow as tf
dataset = tf.data.Dataset.range(0,10,2)
iterator = dataset.make_one_shot_iterator()
sess = tf.Session()
print(sess.run(iterator.get_next()))
```

```shell
breakpoint set -n BackgroundWorker::WorkerLoop
breakpoint set -n MakeDataset

breakpoint set -n OneShotIteratorOp
breakpoint set --file iterator_ops.cc --line 863

breakpoint set -n MakeIteratorInternal
```

```c++

* thread #26, stop reason = breakpoint 1.10
    frame #0: tensorflow::data::(anonymous namespace)::ModelDatasetOp::Dataset::MakeIteratorInternal(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&) const
    
    frame #1: tensorflow::data::DatasetBase::MakeIterator(tensorflow::data::IteratorContext*, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, std::__1::unique_ptr<tensorflow::data::IteratorBase, std::__1::default_delete<tensorflow::data::IteratorBase> >*) const + 62
    
    frame #2: tensorflow::data::IteratorResource::SetIteratorFromDataset(tensorflow::OpKernelContext*, tensorflow::data::DatasetBase*) + 1547
    
    frame #3: std::__1::__function::__func<tensorflow::data::(anonymous namespace)::OneShotIteratorOp::ComputeAsync(tensorflow::OpKernelContext*, std::__1::function<void ()>)::'lambda'(), std::__1::allocator<tensorflow::data::(anonymous namespace)::OneShotIteratorOp::ComputeAsync(tensorflow::OpKernelContext*, std::__1::function<void ()>)::'lambda'()>, void ()>::operator()() + 2548
    
    frame #4: tensorflow::data::BackgroundWorker::WorkerLoop() + 467
    
    frame #5: void* std::__1::__thread_proxy<std::__1::tuple<std::__1::function<void ()> > >(void*) + 96
    
    frame #6: 0x00007fff77e18661 libsystem_pthread.dylib`_pthread_body + 340
    
    frame #7: 0x00007fff77e1850d libsystem_pthread.dylib`_pthread_start + 377
    
    frame #8: 0x00007fff77e17bf9 libsystem_pthread.dylib`thread_start + 13
```

```C++

* thread #4, stop reason = breakpoint 3.1
  * frame #0: 0x000000011a88f410 _pywrap_tensorflow_internal.so`tensorflow::data::(anonymous namespace)::OneShotIteratorOp::ComputeAsync(tensorflow::OpKernelContext*, std::__1::function<void ()>)
    frame #1: 0x000000011181f3d9 libtensorflow_framework.so`tensorflow::Device::ComputeAsync(tensorflow::AsyncOpKernel*, tensorflow::OpKernelContext*, std::__1::function<void ()>) + 105
    frame #2: 0x00000001118440ba libtensorflow_framework.so`tensorflow::(anonymous namespace)::ExecutorState::Process(tensorflow::(anonymous namespace)::ExecutorState::TaggedNode, long long) + 3322
    frame #3: 0x0000000111846455 libtensorflow_framework.so`tensorflow::(anonymous namespace)::ExecutorState::ScheduleReady(absl::InlinedVector<tensorflow::(anonymous namespace)::ExecutorState::TaggedNode, 8ul, std::__1::allocator<tensorflow::(anonymous namespace)::ExecutorState::TaggedNode> > const&, tensorflow::(anonymous namespace)::ExecutorState::TaggedNodeReadyQueue*)::$_1::operator()() const + 101
    frame #4: 0x00000001118463dd libtensorflow_framework.so`void std::__1::__invoke_void_return_wrapper<void>::__call<tensorflow::(anonymous namespace)::ExecutorState::ScheduleReady(absl::InlinedVector<tensorflow::(anonymous namespace)::ExecutorState::TaggedNode, 8ul, std::__1::allocator<tensorflow::(anonymous namespace)::ExecutorState::TaggedNode> > const&, tensorflow::(anonymous namespace)::ExecutorState::TaggedNodeReadyQueue*)::$_1&>(tensorflow::(anonymous namespace)::ExecutorState::ScheduleReady(absl::InlinedVector<tensorflow::(anonymous namespace)::ExecutorState::TaggedNode, 8ul, std::__1::allocator<tensorflow::(anonymous namespace)::ExecutorState::TaggedNode> > const&, tensorflow::(anonymous namespace)::ExecutorState::TaggedNodeReadyQueue*)::$_1&&&) + 45
    frame #5: 0x00000001118462f9 libtensorflow_framework.so`std::__1::__function::__func<tensorflow::(anonymous namespace)::ExecutorState::ScheduleReady(absl::InlinedVector<tensorflow::(anonymous namespace)::ExecutorState::TaggedNode, 8ul, std::__1::allocator<tensorflow::(anonymous namespace)::ExecutorState::TaggedNode> > const&, tensorflow::(anonymous namespace)::ExecutorState::TaggedNodeReadyQueue*)::$_1, std::__1::allocator<tensorflow::(anonymous namespace)::ExecutorState::ScheduleReady(absl::InlinedVector<tensorflow::(anonymous namespace)::ExecutorState::TaggedNode, 8ul, std::__1::allocator<tensorflow::(anonymous namespace)::ExecutorState::TaggedNode> > const&, tensorflow::(anonymous namespace)::ExecutorState::TaggedNodeReadyQueue*)::$_1>, void ()>::operator()() + 41
    frame #6: 0x00000001111914a5 libtensorflow_framework.so`std::__1::function<void ()>::operator()() const + 53
    frame #7: 0x00000001119dbbf4 libtensorflow_framework.so`tensorflow::thread::EigenEnvironment::ExecuteTask(tensorflow::thread::EigenEnvironment::Task const&) + 148
    frame #8: 0x00000001119da99a libtensorflow_framework.so`Eigen::NonBlockingThreadPoolTempl<tensorflow::thread::EigenEnvironment>::WorkerLoop(int) + 2858
    frame #9: 0x00000001119d9e5e libtensorflow_framework.so`Eigen::NonBlockingThreadPoolTempl<tensorflow::thread::EigenEnvironment>::NonBlockingThreadPoolTempl(int, bool, tensorflow::thread::EigenEnvironment)::'lambda'()::operator()() const + 30
    frame #10: 0x00000001119d9e2d libtensorflow_framework.so`void std::__1::__invoke_void_return_wrapper<void>::__call<Eigen::NonBlockingThreadPoolTempl<tensorflow::thread::EigenEnvironment>::NonBlockingThreadPoolTempl(int, bool, tensorflow::thread::EigenEnvironment)::'lambda'()&>(Eigen::NonBlockingThreadPoolTempl<tensorflow::thread::EigenEnvironment>::NonBlockingThreadPoolTempl(int, bool, tensorflow::thread::EigenEnvironment)::'lambda'()&&&) + 45
    frame #11: 0x00000001119d9d69 libtensorflow_framework.so`std::__1::__function::__func<Eigen::NonBlockingThreadPoolTempl<tensorflow::thread::EigenEnvironment>::NonBlockingThreadPoolTempl(int, bool, tensorflow::thread::EigenEnvironment)::'lambda'(), std::__1::allocator<Eigen::NonBlockingThreadPoolTempl<tensorflow::thread::EigenEnvironment>::NonBlockingThreadPoolTempl(int, bool, tensorflow::thread::EigenEnvironment)::'lambda'()>, void ()>::operator()() + 41
    frame #12: 0x00000001111914a5 libtensorflow_framework.so`std::__1::function<void ()>::operator()() const + 53
    frame #13: 0x00000001119d8e54 libtensorflow_framework.so`tensorflow::thread::EigenEnvironment::CreateThread(std::__1::function<void ()>)::'lambda'()::operator()() const + 52
    frame #14: 0x00000001119d8e0d libtensorflow_framework.so`void std::__1::__invoke_void_return_wrapper<void>::__call<tensorflow::thread::EigenEnvironment::CreateThread(std::__1::function<void ()>)::'lambda'()&>(tensorflow::thread::EigenEnvironment::CreateThread(std::__1::function<void ()>)::'lambda'()&&&) + 45
    frame #15: 0x00000001119d8ca9 libtensorflow_framework.so`std::__1::__function::__func<tensorflow::thread::EigenEnvironment::CreateThread(std::__1::function<void ()>)::'lambda'(), std::__1::allocator<tensorflow::thread::EigenEnvironment::CreateThread(std::__1::function<void ()>)::'lambda'()>, void ()>::operator()() + 41
    frame #16: 0x00000001111914a5 libtensorflow_framework.so`std::__1::function<void ()>::operator()() const + 53
    frame #17: 0x0000000111a65aaa libtensorflow_framework.so`void* std::__1::__thread_proxy<std::__1::tuple<std::__1::unique_ptr<std::__1::__thread_struct, std::__1::default_delete<std::__1::__thread_struct> >, std::__1::function<void ()> > >(void*) + 442
    frame #18: 0x00007fff77e18661 libsystem_pthread.dylib`_pthread_body + 340
    frame #19: 0x00007fff77e1850d libsystem_pthread.dylib`_pthread_start + 377
    frame #20: 0x00007fff77e17bf9 libsystem_pthread.dylib`thread_start + 13
    ```

    ```C++

    * thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 6.1
  * frame #0: 0x000000011a88e9d0 _pywrap_tensorflow_internal.so`tensorflow::data::(anonymous namespace)::OneShotIteratorOp::OneShotIteratorOp(tensorflow::OpKernelConstruction*)
    frame #1: 0x000000011a88e9bf _pywrap_tensorflow_internal.so`tensorflow::data::(anonymous namespace)::$_11::operator()(tensorflow::OpKernelConstruction*) const + 47
    frame #2: 0x000000011a88e988 _pywrap_tensorflow_internal.so`tensorflow::data::(anonymous namespace)::$_11::__invoke(tensorflow::OpKernelConstruction*) + 24
    frame #3: 0x0000000111310b36 libtensorflow_framework.so`tensorflow::CreateOpKernel(tensorflow::DeviceType, tensorflow::DeviceBase*, tensorflow::Allocator*, tensorflow::FunctionLibraryRuntime*, tensorflow::NodeDef const&, int, tensorflow::OpKernel**) + 1830
    frame #4: 0x0000000111839f8c libtensorflow_framework.so`tensorflow::CreateNonCachedKernel(tensorflow::Device*, tensorflow::FunctionLibraryRuntime*, tensorflow::NodeDef const&, int, tensorflow::OpKernel**) + 268
    frame #5: 0x000000011187455a libtensorflow_framework.so`tensorflow::FunctionLibraryRuntimeImpl::CreateKernel(tensorflow::NodeDef const&, tensorflow::FunctionLibraryDefinition const*, tensorflow::OpKernel**) + 1162
    frame #6: 0x00000001118740bb libtensorflow_framework.so`tensorflow::FunctionLibraryRuntimeImpl::CreateKernel(tensorflow::NodeDef const&, tensorflow::OpKernel**) + 59
    frame #7: 0x00000001202749bb _pywrap_tensorflow_internal.so`tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_8::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const::'lambda'(tensorflow::OpKernel**)::operator()(tensorflow::OpKernel**) const + 59
    frame #8: 0x0000000120274967 _pywrap_tensorflow_internal.so`tensorflow::Status std::__1::__invoke_void_return_wrapper<tensorflow::Status>::__call<tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_8::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const::'lambda'(tensorflow::OpKernel**)&, tensorflow::OpKernel**>(tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_8::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const::'lambda'(tensorflow::OpKernel**)&&&, tensorflow::OpKernel**&&) + 87
    frame #9: 0x0000000120274830 _pywrap_tensorflow_internal.so`std::__1::__function::__func<tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_8::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const::'lambda'(tensorflow::OpKernel**), std::__1::allocator<tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_8::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const::'lambda'(tensorflow::OpKernel**)>, tensorflow::Status (tensorflow::OpKernel**)>::operator()(tensorflow::OpKernel**&&) + 64
    frame #10: 0x0000000111320312 libtensorflow_framework.so`std::__1::function<tensorflow::Status (tensorflow::OpKernel**)>::operator()(tensorflow::OpKernel**) const + 98
    frame #11: 0x000000011131fafb libtensorflow_framework.so`tensorflow::OpSegment::FindOrCreate(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, tensorflow::OpKernel**, std::__1::function<tensorflow::Status (tensorflow::OpKernel**)>) + 331
    frame #12: 0x00000001202737cb _pywrap_tensorflow_internal.so`tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_8::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const + 363
    frame #13: 0x0000000120273647 _pywrap_tensorflow_internal.so`tensorflow::Status std::__1::__invoke_void_return_wrapper<tensorflow::Status>::__call<tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_8&, tensorflow::NodeDef const&, tensorflow::OpKernel**>(tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_8&&&, tensorflow::NodeDef const&&&, tensorflow::OpKernel**&&) + 119
    frame #14: 0x00000001202734f0 _pywrap_tensorflow_internal.so`std::__1::__function::__func<tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_8, std::__1::allocator<tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*)::$_8>, tensorflow::Status (tensorflow::NodeDef const&, tensorflow::OpKernel**)>::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**&&) + 80
    frame #15: 0x000000011ac518d1 _pywrap_tensorflow_internal.so`std::__1::function<tensorflow::Status (tensorflow::NodeDef const&, tensorflow::OpKernel**)>::operator()(tensorflow::NodeDef const&, tensorflow::OpKernel**) const + 193
    frame #16: 0x0000000111839451 libtensorflow_framework.so`tensorflow::(anonymous namespace)::ExecutorImpl::Initialize() + 1761
    frame #17: 0x0000000111838cab libtensorflow_framework.so`tensorflow::NewLocalExecutor(tensorflow::LocalExecutorParams const&, std::__1::unique_ptr<tensorflow::Graph const, std::__1::default_delete<tensorflow::Graph const> >, tensorflow::Executor**) + 747
    frame #18: 0x000000011186d815 libtensorflow_framework.so`tensorflow::(anonymous namespace)::DefaultExecutorRegistrar::Factory::NewExecutor(tensorflow::LocalExecutorParams const&, std::__1::unique_ptr<tensorflow::Graph const, std::__1::default_delete<tensorflow::Graph const> >, std::__1::unique_ptr<tensorflow::Executor, std::__1::default_delete<tensorflow::Executor> >*) + 549
    frame #19: 0x000000011187048d libtensorflow_framework.so`tensorflow::NewExecutor(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, tensorflow::LocalExecutorParams const&, std::__1::unique_ptr<tensorflow::Graph const, std::__1::default_delete<tensorflow::Graph const> >, std::__1::unique_ptr<tensorflow::Executor, std::__1::default_delete<tensorflow::Executor> >*) + 669
    frame #20: 0x0000000120240b07 _pywrap_tensorflow_internal.so`tensorflow::DirectSession::CreateExecutors(tensorflow::CallableOptions const&, std::__1::unique_ptr<tensorflow::DirectSession::ExecutorsAndKeys, std::__1::default_delete<tensorflow::DirectSession::ExecutorsAndKeys> >*, std::__1::unique_ptr<tensorflow::DirectSession::FunctionInfo, std::__1::default_delete<tensorflow::DirectSession::FunctionInfo> >*, tensorflow::DirectSession::RunStateArgs*) + 23287
    frame #21: 0x000000012022d1c9 _pywrap_tensorflow_internal.so`tensorflow::DirectSession::GetOrCreateExecutors(absl::Span<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const>, absl::Span<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const>, absl::Span<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const>, tensorflow::DirectSession::ExecutorsAndKeys**, tensorflow::DirectSession::RunStateArgs*) + 14521
    frame #22: 0x00000001202274e8 _pywrap_tensorflow_internal.so`tensorflow::DirectSession::Run(tensorflow::RunOptions const&, std::__1::vector<std::__1::pair<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, tensorflow::Tensor>, std::__1::allocator<std::__1::pair<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, tensorflow::Tensor> > > const&, std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > > > const&, std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > > > const&, std::__1::vector<tensorflow::Tensor, std::__1::allocator<tensorflow::Tensor> >*, tensorflow::RunMetadata*) + 2008
    frame #23: 0x000000011782cbf6 _pywrap_tensorflow_internal.so`tensorflow::SessionRef::Run(tensorflow::RunOptions const&, std::__1::vector<std::__1::pair<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, tensorflow::Tensor>, std::__1::allocator<std::__1::pair<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, tensorflow::Tensor> > > const&, std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > > > const&, std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > > > const&, std::__1::vector<tensorflow::Tensor, std::__1::allocator<tensorflow::Tensor> >*, tensorflow::RunMetadata*) + 662
    frame #24: 0x0000000117f7d80f _pywrap_tensorflow_internal.so`TF_Run_Helper(tensorflow::Session*, char const*, TF_Buffer const*, std::__1::vector<std::__1::pair<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, tensorflow::Tensor>, std::__1::allocator<std::__1::pair<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, tensorflow::Tensor> > > const&, std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > > > const&, TF_Tensor**, std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > > > const&, TF_Buffer*, TF_Status*) + 527
    frame #25: 0x0000000117f952c0 _pywrap_tensorflow_internal.so`TF_SessionRun + 2640
    frame #26: 0x000000011782159f _pywrap_tensorflow_internal.so`tensorflow::TF_SessionRun_wrapper_helper(TF_Session*, char const*, TF_Buffer const*, std::__1::vector<TF_Output, std::__1::allocator<TF_Output> > const&, std::__1::vector<_object*, std::__1::allocator<_object*> > const&, std::__1::vector<TF_Output, std::__1::allocator<TF_Output> > const&, std::__1::vector<TF_Operation*, std::__1::allocator<TF_Operation*> > const&, TF_Buffer*, TF_Status*, std::__1::vector<_object*, std::__1::allocator<_object*> >*) + 3359
    frame #27: 0x00000001178229bd _pywrap_tensorflow_internal.so`tensorflow::TF_SessionRun_wrapper(TF_Session*, TF_Buffer const*, std::__1::vector<TF_Output, std::__1::allocator<TF_Output> > const&, std::__1::vector<_object*, std::__1::allocator<_object*> > const&, std::__1::vector<TF_Output, std::__1::allocator<TF_Output> > const&, std::__1::vector<TF_Operation*, std::__1::allocator<TF_Operation*> > const&, TF_Buffer*, TF_Status*, std::__1::vector<_object*, std::__1::allocator<_object*> >*) + 141
    frame #28: 0x00000001176ed290 _pywrap_tensorflow_internal.so`_wrap_TF_SessionRun_wrapper(_object*, _object*) + 5168
    ```



- Resources:
    - https://gist.github.com/Mistobaan/738e76c3a5bb1f9bcc52e2809a23a7a1
    - https://www.tensorflow.org/programmers_guide/debugger?hl=zh-cn
    - https://pure-earth-7284.herokuapp.com/2016/10/18/debug-tensorflow-using-lldb/
    - https://www.weibo.com/ttarticle/p/show?id=2309404042002588154581
    - https://www.zybuluo.com/qidiandasheng/note/349994
    - https://stackoverflow.com/questions/40889221/debugging-tensorflow-tests-pdb-or-gdb
    - https://casatwy.com/shi-yong-lldbdiao-shi-cheng-xu.html

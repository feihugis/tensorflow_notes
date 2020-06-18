## How does Eager tape compute the gradient

### Forward:

tensorflow/c/eager/c_api.cc::TFE_Execute (688) -> 

tensorflow/core/common_runtime/eager/execute.cc -> EagerExecute (685) -> EagerLocalExecute (249) -> if (ctx->asyn() == false) (334) -> EagerExecute(705)
---

### Backward

pywrap_tensorflow_internal.cc:6485

tensorflow/python/eager/pywrap_tfe_src.cc -> RunCallbacks(2386) -> RecordGradient(2002) -> OpGradientDoesntRequireOutputIndices (1840) -> TFE_Py_TapeGradient (1621)
tensorflow/c/eager/tape.h:: ComputeGradient (468) -> trace = tensorflow::eager::OpTapeEntry (514)
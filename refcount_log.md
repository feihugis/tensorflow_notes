When I debug the issues related to the refcounted objects, I add some logs to `tensorflow/core/lib/core/refcount.h` as following:
```C++
// Inlined routines, since these are performance critical
inline RefCounted::RefCounted() : ref_(1) {
  VLOG(0) << "****** RefCounted::Constructor: " << ref_.load() << " , " << this;
}

inline RefCounted::~RefCounted() { DCHECK_EQ(ref_.load(), 0); }

inline void RefCounted::Ref() const {
  DCHECK_GE(ref_.load(), 1);
  ref_.fetch_add(1, std::memory_order_relaxed);

  VLOG(0) << "****** RefCounted::Ref: " << ref_.load() << " , " << this;
}

inline bool RefCounted::Unref() const {
  DCHECK_GT(ref_.load(), 0);
  // If ref_==1, this object is owned only by the caller. Bypass a locked op
  // in that case.
  if (RefCountIsOne() || ref_.fetch_sub(1) == 1) {
    // Make DCHECK in ~RefCounted happy
    DCHECK((ref_.store(0), true));
    delete this;
    VLOG(0) << "****** RefCounted::Unref: true, " << ref_.load() << " , " << this;
    return true;
  } else {
    VLOG(0) << "****** RefCounted::Unref: false, " << ref_.load() << " , " << this;
    return false;
  }
}
```
Then in the log file, I could check whether the refcounted instance is released.

Here is an example of the log output. Through the log, I compare the number of `****** RefCounted::Constructor: 1` log line and the number of `****** RefCounted::Unref: true, 0`log line. If they match with each other, it means all the refcounted instances are released. I'm wondering if we could support some checking tools for TensorFlow to check whether all the refcounted instances are released. This kind of checks can only be enabled when running tests if it will impair the performance. 

```shell
[ RUN      ] MapDatasetOpTest.DatasetName
2019-01-30 19:39:42.205652: I tensorflow/core/platform/cpu_feature_guard.cc:141] Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.2 AVX AVX2 FMA
2019-01-30 19:39:42.206134: I ./tensorflow/core/lib/core/refcount.h:90] ****** RefCounted::Constructor: 1 , 0x7f869f40f420
2019-01-30 19:39:42.206150: I ./tensorflow/core/lib/core/refcount.h:99] ****** RefCounted::Ref: 2 , 0x7f869f40f420
2019-01-30 19:39:42.206163: I ./tensorflow/core/lib/core/refcount.h:113] ****** RefCounted::Unref: false, 1 , 0x7f869f40f420
2019-01-30 19:39:42.206247: I ./tensorflow/core/lib/core/refcount.h:110] ****** RefCounted::Unref: true, 1866688101 , 0x7f869f40f420
2019-01-30 19:39:42.206899: I ./tensorflow/core/lib/core/refcount.h:90] ****** RefCounted::Constructor: 1 , 0x7f869f432ed0
2019-01-30 19:39:42.206941: I ./tensorflow/core/lib/core/refcount.h:90] ****** RefCounted::Constructor: 1 , 0x7f869f436810
2019-01-30 19:39:42.206954: I ./tensorflow/core/lib/core/refcount.h:90] ****** RefCounted::Constructor: 1 , 0x7f869f440610
2019-01-30 19:39:42.207020: I ./tensorflow/core/lib/core/refcount.h:90] ****** RefCounted::Constructor: 1 , 0x7f869f40f250
2019-01-30 19:39:42.207051: I ./tensorflow/core/lib/core/refcount.h:90] ****** RefCounted::Constructor: 1 , 0x7f869f433780
2019-01-30 19:39:42.207067: I ./tensorflow/core/lib/core/refcount.h:99] ****** RefCounted::Ref: 2 , 0x7f869f40f250
2019-01-30 19:39:42.207080: I ./tensorflow/core/lib/core/refcount.h:113] ****** RefCounted::Unref: false, 1 , 0x7f869f40f250
2019-01-30 19:39:42.207098: I ./tensorflow/core/lib/core/refcount.h:99] ****** RefCounted::Ref: 2 , 0x7f869f40f250
2019-01-30 19:39:42.207107: I ./tensorflow/core/lib/core/refcount.h:113] ****** RefCounted::Unref: false, 1 , 0x7f869f40f250
2019-01-30 19:39:42.207115: I ./tensorflow/core/lib/core/refcount.h:110] ****** RefCounted::Unref: true, 1866688101 , 0x7f869f433780
2019-01-30 19:39:42.207320: I ./tensorflow/core/lib/core/refcount.h:90] ****** RefCounted::Constructor: 1 , 0x7f869f443a90
2019-01-30 19:39:42.207333: I ./tensorflow/core/lib/core/refcount.h:99] ****** RefCounted::Ref: 2 , 0x7f869f40f250
2019-01-30 19:39:42.207341: I ./tensorflow/core/lib/core/refcount.h:113] ****** RefCounted::Unref: false, 1 , 0x7f869f40f250
2019-01-30 19:39:42.207348: I ./tensorflow/core/lib/core/refcount.h:99] ****** RefCounted::Ref: 2 , 0x7f869f40f250
2019-01-30 19:39:42.207354: I ./tensorflow/core/lib/core/refcount.h:99] ****** RefCounted::Ref: 3 , 0x7f869f40f250
2019-01-30 19:39:42.207363: I ./tensorflow/core/lib/core/refcount.h:90] ****** RefCounted::Constructor: 1 , 0x7f869f437450
2019-01-30 19:39:42.207372: I ./tensorflow/core/lib/core/refcount.h:99] ****** RefCounted::Ref: 4 , 0x7f869f40f250
2019-01-30 19:39:42.207379: I ./tensorflow/core/lib/core/refcount.h:113] ****** RefCounted::Unref: false, 3 , 0x7f869f40f250
2019-01-30 19:39:42.207385: I ./tensorflow/core/lib/core/refcount.h:99] ****** RefCounted::Ref: 4 , 0x7f869f40f250
2019-01-30 19:39:42.207397: I ./tensorflow/core/lib/core/refcount.h:113] ****** RefCounted::Unref: false, 3 , 0x7f869f40f250
2019-01-30 19:39:42.207404: I ./tensorflow/core/lib/core/refcount.h:113] ****** RefCounted::Unref: false, 2 , 0x7f869f40f250
2019-01-30 19:39:42.207411: I ./tensorflow/core/lib/core/refcount.h:110] ****** RefCounted::Unref: true, 1866688101 , 0x7f869f443a90
2019-01-30 19:39:42.208699: I ./tensorflow/core/lib/core/refcount.h:90] ****** RefCounted::Constructor: 1 , 0x7f869f42d480
2019-01-30 19:39:42.208718: I ./tensorflow/core/lib/core/refcount.h:99] ****** RefCounted::Ref: 3 , 0x7f869f40f250
2019-01-30 19:39:42.208732: I ./tensorflow/core/lib/core/refcount.h:90] ****** RefCounted::Constructor: 1 , 0x7f869f438ef0
2019-01-30 19:39:42.208741: I ./tensorflow/core/lib/core/refcount.h:99] ****** RefCounted::Ref: 2 , 0x7f869f42d480
2019-01-30 19:39:42.208749: I ./tensorflow/core/lib/core/refcount.h:113] ****** RefCounted::Unref: false, 1 , 0x7f869f42d480
2019-01-30 19:39:42.208756: I ./tensorflow/core/lib/core/refcount.h:99] ****** RefCounted::Ref: 2 , 0x7f869f42d480
2019-01-30 19:39:42.208763: I ./tensorflow/core/lib/core/refcount.h:77] ****** ScopedUnref
2019-01-30 19:39:42.208768: I ./tensorflow/core/lib/core/refcount.h:113] ****** RefCounted::Unref: false, 1 , 0x7f869f42d480
2019-01-30 19:39:42.208776: I ./tensorflow/core/lib/core/refcount.h:113] ****** RefCounted::Unref: false, 2 , 0x7f869f40f250
2019-01-30 19:39:42.208798: I ./tensorflow/core/lib/core/refcount.h:110] ****** RefCounted::Unref: true, 0 , 0x7f869f42d480
2019-01-30 19:39:42.208810: I ./tensorflow/core/lib/core/refcount.h:110] ****** RefCounted::Unref: true, 1866688101 , 0x7f869f438ef0
2019-01-30 19:39:42.208834: I ./tensorflow/core/lib/core/refcount.h:77] ****** ScopedUnref
2019-01-30 19:39:42.208842: I ./tensorflow/core/lib/core/refcount.h:113] ****** RefCounted::Unref: false, 1 , 0x7f869f40f250
2019-01-30 19:39:42.208882: I ./tensorflow/core/lib/core/refcount.h:110] ****** RefCounted::Unref: true, 0 , 0x7f869f40f250
2019-01-30 19:39:42.208893: I ./tensorflow/core/lib/core/refcount.h:110] ****** RefCounted::Unref: true, 1866688101 , 0x7f869f437450
2019-01-30 19:39:42.208902: I ./tensorflow/core/lib/core/refcount.h:110] ****** RefCounted::Unref: true, 1866688101 , 0x7f869f440610
2019-01-30 19:39:42.208910: I ./tensorflow/core/lib/core/refcount.h:110] ****** RefCounted::Unref: true, 1866688101 , 0x7f869f436810
2019-01-30 19:39:42.208939: I ./tensorflow/core/lib/core/refcount.h:110] ****** RefCounted::Unref: true, 0 , 0x7f869f432ed0
[       OK ] MapDatasetOpTest.DatasetName (5 ms)
```

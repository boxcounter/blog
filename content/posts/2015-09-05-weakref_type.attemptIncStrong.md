---
title           : RefBase::weakref_type::attemptIncStrong的理解
date            : 2015-09-05
tags            : ["Android"]
category        : "研发"
isCJKLanguage   : true
---

见中文注释内容：

{{< highlight cpp >}}
bool RefBase::weakref_type::attemptIncStrong(const void* id)
{
    incWeak(id);
    
    weakref_impl* const impl = static_cast<weakref_impl*>(this);
    
    int32_t curCount = impl->mStrong;
    LOG_ASSERT(curCount >= 0, "attemptIncStrong called on %p after underflow",
               this);
    while (curCount > 0 && curCount != INITIAL_STRONG_VALUE) {
        if (android_atomic_cmpxchg(curCount, curCount+1, &impl->mStrong) == 0) {
            break;
        }
        curCount = impl->mStrong;
    }
    
    if (curCount <= 0 || curCount == INITIAL_STRONG_VALUE) {
        bool allow;
        if (curCount == INITIAL_STRONG_VALUE) {
            // Attempting to acquire first strong reference...  this is allowed
            // if the object does NOT have a longer lifetime (meaning the
            // implementation doesn't need to see this), or if the implementation
            // allows it to happen.
            //
            // ====>
            // 未曾被强引用过，如果也不受弱引用影响，那肯定还未被销毁。
            // <====
            //
            allow = (impl->mFlags&OBJECT_LIFETIME_WEAK) != OBJECT_LIFETIME_WEAK
                  || impl->mBase->onIncStrongAttempted(FIRST_INC_STRONG, id);
        } else {
            // Attempting to revive the object...  this is allowed
            // if the object DOES have a longer lifetime (so we can safely
            // call the object with only a weak ref) and the implementation
            // allows it to happen.
            //
            // ====>
            // 曾经被强引用过，如果不受弱引用影响，那么在RefBase.decStrong()中已经被销毁了，无法revive。
            // 所以，一定要受弱引用影响。
            // <====
            //
            allow = (impl->mFlags&OBJECT_LIFETIME_WEAK) == OBJECT_LIFETIME_WEAK
                  && impl->mBase->onIncStrongAttempted(FIRST_INC_STRONG, id);
        }
        if (!allow) {
            decWeak(id);
            return false;
        }
        curCount = android_atomic_inc(&impl->mStrong);
        // If the strong reference count has already been incremented by
        // someone else, the implementor of onIncStrongAttempted() is holding
        // an unneeded reference.  So call onLastStrongRef() here to remove it.
        // (No, this is not pretty.)  Note that we MUST NOT do this if we
        // are in fact acquiring the first reference.
        if (curCount > 0 && curCount < INITIAL_STRONG_VALUE) {
            impl->mBase->onLastStrongRef(id);
        }
    }
    
    impl->addWeakRef(id);
    impl->addStrongRef(id);
#if PRINT_REFS
    LOGD("attemptIncStrong of %p from %p: cnt=%d\n", this, id, curCount);
#endif
    if (curCount == INITIAL_STRONG_VALUE) {
        android_atomic_add(-INITIAL_STRONG_VALUE, &impl->mStrong);
        impl->mBase->onFirstRef();
    }
    
    return true;
}
{{< /highlight >}}

{{< highlight cpp >}}
void RefBase::decStrong(const void* id) const
{
    weakref_impl* const refs = mRefs;
    refs->removeStrongRef(id);
    const int32_t c = android_atomic_dec(&refs->mStrong);
#if PRINT_REFS
    LOGD("decStrong of %p from %p: cnt=%d\n", this, id, c);
#endif
    LOG_ASSERT(c >= 1, "decStrong() called on %p too many times", refs);
    if (c == 1) {
        const_cast<RefBase*>(this)->onLastStrongRef(id);
        if ((refs->mFlags&OBJECT_LIFETIME_WEAK) != OBJECT_LIFETIME_WEAK) {
            if (refs->mDestroyer) {
                refs->mDestroyer->destroy(this);
            } else {
                delete this;
            }
        }
    }
    refs->removeWeakRef(id);
    refs->decWeak(id);
}
{{< /highlight >}}

---

完整代码：

- [RefBase.h](https://android.googlesource.com/platform/frameworks/base/+/gingerbread/include/utils/RefBase.h)
- [RefBase.cpp](https://android.googlesource.com/platform/frameworks/base/+/gingerbread/libs/utils/RefBase.cpp)

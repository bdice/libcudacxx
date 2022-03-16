---
grand_parent: Extended API
parent: Synchronization Primitives
nav_order: 1
---

# `cuda::atomic_ref`

Defined in header `<cuda/atomic>`:

```cuda
template <typename T, cuda::thread_scope Scope = cuda::thread_scope_system>
class cuda::atomic_ref;
```

The class template `cuda::atomic_ref` is an extended form of [`cuda::std::atomic_ref`]
  that takes an additional [`cuda::thread_scope`] argument, defaulted to
  `cuda::std::thread_scope_system`.
It has the same interface and semantics as [`cuda::std::atomic_ref`], with the
  following additional operations.
This class additionally deviates from the standard by being backported to C++11.

## Limitations

`cuda::atomic_ref<T>` and `cuda::std::atomic_ref<T>` may only be instantiated with types sized between 4 and 8 bytes.

## Atomic Fence Operations

| [`cuda::atomic_thread_fence`] | Memory order and scope dependent fence synchronization primitive. `(function)` |

## Atomic Extrema Operations

| [`cuda::atomic::fetch_min`] | Atomically find the minimum of the stored value and a provided value. `(member function)` |
| [`cuda::atomic::fetch_max`] | Atomically find the maximum of the stored value and a provided value. `(member function)` |

## Concurrency Restrictions

An object of type `cuda::atomic_ref` or [`cuda::std::atomic_ref`] shall not be accessed
  concurrently by CPU and GPU threads unless:
- it is in unified memory and the [`concurrentManagedAccess` property] is 1, or
- it is in CPU memory and the [`hostNativeAtomicSupported` property] is 1.

Note, for objects of scopes other than `cuda::thread_scope_system` this is a
  data-race, and thefore also prohibited regardless of memory characteristics.

Under CUDA Compute Capability 6 (Pascal), an object of type `atomic` may not be
  used:
- with automatic storage duration, or
- if `is_always_lock_free()` is `false`.

Under CUDA Compute Capability prior to 6 (Pascal), objects of type
  `cuda::atomic_ref` or [`cuda::std::atomic_ref`] may not be used.

## Implementation-Defined Behavior

For each type `T` and [`cuda::thread_scope`] `S`, the value of
  `cuda::atomic_ref<T, S>::is_always_lock_free()` is as follows:

| Type `T` | [`cuda::thread_scope`] `S` | `cuda::atomic_ref<T, S>::is_always_lock_free()` |
|----------|----------------------------|---------------------------------------------|
| Any      | Any                        | `sizeof(T) <= 8`                            |

## Example

```cuda
#include <cuda/atomic>

__global__ void example_kernel(int *gmem, int *pinned_mem) {
  // This atomic is suitable for all threads in the system.
  cuda::atomic_ref<int, cuda::thread_scope_system> a(pinned_mem);

  // This atomic has the same type as the previous one (`a`).
  cuda::atomic_ref<int> b(pinned_mem);

  // This atomic is suitable for all threads on the current processor (e.g. GPU).
  cuda::atomic_ref<int, cuda::thread_scope_device> c(gmem);

  __shared__ int shared_v;
  // This atomic is suitable for threads in the same thread block.
  cuda::atomic_ref<int, cuda::thread_scope_block> d(&shared);
}
```

[See it on Godbolt](https://godbolt.org/z/fr4K7ErEh){: .btn }


[`cuda::thread_scope`]: ../thread_scopes.md

[`cuda::atomic_thread_fence`]: ./atomic/atomic_thread_fence.md

[`cuda::atomic::fetch_min`]: ./atomic/fetch_min.md
[`cuda::atomic::fetch_max`]: ./atomic/fetch_max.md

[`cuda::std::atomic_ref`]: https://en.cppreference.com/w/cpp/atomic/atomic_ref

[atomics.types.int]: https://eel.is/c++draft/atomics.types.int
[atomics.types.pointer]: https://eel.is/c++draft/atomics.types.pointer

[`concurrentManagedAccess` property]: https://docs.nvidia.com/cuda/cuda-runtime-api/structcudaDeviceProp.html#structcudaDeviceProp_116f9619ccc85e93bc456b8c69c80e78b
[`hostNativeAtomicSupported` property]: https://docs.nvidia.com/cuda/cuda-runtime-api/structcudaDeviceProp.html#structcudaDeviceProp_1ef82fd7d1d0413c7d6f33287e5b6306f


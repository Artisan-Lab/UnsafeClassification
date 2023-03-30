consecutive
overlap

# ControlFlow
## Reachable
```rust
pub const unsafe fn unreachable_unchecked() -> ! {}
```

# Pre Condition
## Numeric
### ABSOLUTE Value
```rust
// transmute
pub const unsafe fn from_u32_unchecked(i: u32) -> char
// i < 256
```

```rust
pub unsafe trait GlobalAlloc {}
unsafe fn realloc(&self, ptr: *mut u8, layout: Layout, new_size: usize) -> *mut u8 {}
// new_size > 0
// new_size < usize::Max
```

```rust
impl ValidAlign {}
pub(crate) const unsafe fn new_unchecked(align: usize) -> Self {}
// align.is_power_of_two(), align > 0
```

```rust
// float
pub unsafe fn to_int_unchecked<Int>(self) -> Int
where
        Self: FloatToInt<Int> {}
// Not be NaN
// Not be infinite
// truncating off < isize::max
```
---- 

### INDIRECT Bound
```rust
unsafe fn shrink(&self, ptr: NonNull<u8>, old_layout: Layout, new_layout: Layout) -> Result<NonNull<[u8]>, AllocError> {}
// new_layout.size() <= old_layout.size
```

```rust
impl<T, const N: usize> IntoIter<T, N> {}
pub const unsafe fn new_unchecked(buffer: [MaybeUninit<T>; N], initialized: Range<usize>) -> Self {}
// initialized.start <= initialized.end
```

```rust
unsafe fn try_collect_into_array_unchecked<I, T, R, const N: usize>(iter: &mut I) -> R::TryType
where
    // Note: `TrustedLen` here is somewhat of an experiment. This is just an
    // internal function, so feel free to remove if this bound turns out to be a
    // bad idea. In that case, remember to also remove the lower bound
    // `debug_assert!` below!
    I: Iterator + TrustedLen,
    I::Item: Try<Output = T, Residual = R>,
    R: Residual<[T; N]>,
{
    debug_assert!(N <= iter.size_hint().1.unwrap_or(usize::MAX));
    debug_assert!(N <= iter.size_hint().0);

    // SAFETY: covered by the function contract.
    unsafe { try_collect_into_array(iter).unwrap_unchecked() }
}
```

```rust
impl<T, const N: usize> IntoIter<T, N> {}
 pub const unsafe fn new_unchecked(buffer: [MaybeUninit<T>; N], initialized: Range<usize>) -> Self {}
// initialized.end <= N
```

```rust
pub trait Step: Clone + PartialOrd + Sized {}
unsafe fn forward_unchecked(start: Self, count: usize) -> Self {}
```
---- 

### Encode / Format
```rust
impl String {}
pub unsafe fn as_mut_vec(&mut self) -> &mut Vec<u8, Global> {}
// UTF-8
```

```rust
impl CStr {}
pub unsafe fn from_ptr<'a>(ptr: *const c_char) -> &'a CStr {}
// CString
```
----

## Pointer
### Consistency
```rust
impl dyn Any {} // downcast
pub unsafe fn downcast_ref_unchecked<T: Any>(&self) -> &T {}
```

```rust
pub unsafe trait GlobalAlloc {}
unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout);
```

```rust
impl<T> MaybeUninit<T> {}
pub unsafe fn assume_init_drop(&mut self) {}
```
----

## Ownership

### Orphan Object
```rust
impl CString {}
pub fn into_raw(self) -> *mut c_char {}
```

```rust
impl<T: ?Sized, A: Allocator> Box<T, A> {}
pub fn into_raw(b: Self) -> *mut T {}
```
----

### Initialization
```rust
impl<T, const N: usize> IntoIter<T, N> {}
pub const unsafe fn new_unchecked(buffer: [MaybeUninit<T>; N], initialized: Range<usize>) -> Self {}
```

```rust
impl<T> MaybeUninit<T> {}
pub const unsafe fn assume_init(self) -> T {}
```

```rust
// zeroed
pub unsafe fn zeroed<T>() -> T {}
```
----

## Mutation & Lifetime

### Borrow Mutation
```rust
impl CStr {}
pub unsafe fn from_ptr<'a>(ptr: *const c_char) -> &'a CStr {}
```

```rust
impl<T: ?Sized> RefCell<T> {}
pub unsafe fn try_borrow_unguarded(&self) -> Result<&T, BorrowError> {}
```
---- 

### Arbitrarily Lifetime
```rust
impl<T: ?Sized> *const T {}
pub const unsafe fn as_ref<'a>(self) -> Option<&'a T> {}
```

```rust
impl<T> *const [T] {}
pub const unsafe fn as_uninit_slice<'a>(self) -> Option<&'a [MaybeUninit<T>]> {}
```
---- 

### Extending/Shortening Lifetime
```rust
struct R<'a>(&'a i32);
unsafe fn extend_lifetime<'b>(r: R<'b>) -> R<'static> {
    std::mem::transmute::<R<'b>, R<'static>>(r)
}

unsafe fn shorten_invariant_lifetime<'b, 'c>(r: &'b mut R<'static>)
                                             -> &'b mut R<'c> {
    std::mem::transmute::<&'b mut R<'static>, &'b mut R<'c>>(r)
}
```
----

## Pointer

### Non-null
```rust
impl<T: ?Sized> Unique<T> {}
pub const unsafe fn new_unchecked(ptr: *mut T) -> Self {}
```
---- 

### Size
```rust
pub const unsafe extern "rust-intrinsic" fn transmute<Src, Dst>(
    src: Src
) -> Dst {}
```
----

### Alignment
```rust
pub(crate) trait ByteSlice: AsRef<[u8]> {}
unsafe fn first_unchecked(&self) -> u8 {}
```
----

### Dereference
```rust
// the memory range of the given size starting at the pointer
// must all be within the bounds of a single allocated object.
// Note that in Rust, every (stack-allocated) variable is considered a separate allocated object.
```

```rust
impl<T: ?Sized> *const T {}
pub const unsafe fn offset_from(self, origin: *const T) -> isize
    where
        T: Sized {}
```
----

### Exotically Sized Types
```rust
pub unsafe trait GlobalAlloc {}
unsafe fn alloc(&self, layout: Layout) -> *mut u8;
```

```rust
// a trait object, then the vtable part of the pointer must point to a valid vtable acquired by an unsizing coercion
// the size of the entire value (dynamic tail length + statically sized prefix) must fit in isize.
pub const unsafe fn size_of_val_raw<T: ?Sized>(val: *const T) -> usize {}
```
----

# Post Condition

## Owner Taking
```rust
impl<T> ManuallyDrop<T> {}
pub unsafe fn take(slot: &mut ManuallyDrop<T>) -> T {}
pub const unsafe fn assume_init_read(&self) -> T {}
```
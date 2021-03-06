
.. _handle:

:c:type:`uv_handle_t` --- Base handle
=====================================

`uv_handle_t` is the base type for all libuv handle types.

Strcutures are aligned so that any libuv handle can be cast to `uv_handle_t`.
All API functions defined here work with any handle type.


Data types
----------

.. c:type:: uv_handle_t

    The base libuv handle structure.

.. c:type:: uv_any_handle

    Union of all handle types.

.. c:type:: void (*uv_close_cb)(uv_handle_t* handle)

    Type definition for callback passed to :c:func:`uv_close`.


Public members
^^^^^^^^^^^^^^

.. c:member:: uv_loop_t* uv_handle_t.loop

    Pointer to the :c:type:`uv_loop_t` where the handle is running on. Readonly.

.. c:member:: void* uv_handle_t.data

    Space for user-defined arbitrary data. libuv does not use this field.


API
---

.. c:function:: int uv_is_active(const uv_handle_t* handle)

    Returns non-zero if the handle is active, zero if it's inactive. What
    "active" means depends on the type of handle:

    - A uv_async_t handle is always active and cannot be deactivated, except
      by closing it with uv_close().

    - A uv_pipe_t, uv_tcp_t, uv_udp_t, etc. handle - basically any handle that
      deals with i/o - is active when it is doing something that involves i/o,
      like reading, writing, connecting, accepting new connections, etc.

    - A uv_check_t, uv_idle_t, uv_timer_t, etc. handle is active when it has
      been started with a call to uv_check_start(), uv_idle_start(), etc.

    Rule of thumb: if a handle of type `uv_foo_t` has a `uv_foo_start()`
    function, then it's active from the moment that function is called.
    Likewise, `uv_foo_stop()` deactivates the handle again.

.. c:function:: int uv_is_closing(const uv_handle_t* handle)

    Returns non-zero if the handle is closing or closed, zero otherwise.

    .. note:: This function should only be used between the initialization of
              the handle and the arrival of the close callback.

.. c:function:: void uv_close(uv_handle_t* handle, uv_close_cb close_cb)

    Request handle to be closed. `close_cb` will be called asynchronously after
    this call. This MUST be called on each handle before memory is released.

    Handles that wrap file descriptors are closed immediately but
    `close_cb` will still be deferred to the next iteration of the event loop.
    It gives you a chance to free up any resources associated with the handle.

    In-progress requests, like uv_connect_t or uv_write_t, are cancelled and
    have their callbacks called asynchronously with status=UV_ECANCELED.

.. c:function:: void uv_ref(uv_handle_t* handle)

    Reference the given handle. References are idempotent, that is, if a handle
    is already referenced calling this function again will have no effect.

    See :ref:`refcount`.

.. c:function:: void uv_unref(uv_handle_t* handle)

    Un-reference the given handle. References are idempotent, that is, if a handle
    is not referenced calling this function again will have no effect.

    See :ref:`refcount`.

.. c:function:: int uv_has_ref(const uv_handle_t* handle)

    Returns non-zero if the handle referenced, zero otherwise.

    See :ref:`refcount`.

.. c:function:: size_t uv_handle_size(uv_handle_type type)

    Returns the size of the given handle type. Useful for FFI binding writers
    who don't want to know the structure layout.


.. _refcount:

Refcounting
-----------

The libuv event loop (if run in the default mode) will run until there are no
active `and` referenced handles left. The user can force the loop to exit early
by unreferencing handles which are active, for example by calling :c:func:`uv_unref`
after calling :c:func:`uv_timer_start`.

A handle can be referenced or unreferenced, the refcounting scheme doesn't use
a counter, so both operations are idempotent.

All handles are referenced when active by default, see :c:func:`uv_is_active`
for a more detailed explanation on what being `active` involves.


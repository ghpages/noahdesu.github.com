---
layout: post
title: Custom Lua VM Panic Handler
---

Extending applications with Lua is amazingly powerful. The task can be a
little mind-binding, but with a bit of practice it all begins to make sense.
One challenge with embedding a Lua VM is avoiding the possibility of crashing
the host application. This is especially important for high-availability
systems such as file system servers.

It is good practice to execute everything in a Lua protected environment in
which case errors are reported through the normal `lua_error` path. However
Lua may also find itself in an unrecoverable state in which it calls its
installed `panic` handler. The default handler will call `exit()` in the host
process, thus it is crucial for stability that we provide a customized version
that recovers gracefully.

Since a Lua panic is a dead-end road, meaning Lua can't recover, we must help
it recover by returning execution to a safe context. We do this using
[setjmp.h](http://en.wikipedia.org/wiki/Setjmp.h), which happens to be used
extensively in the implementation of Lua.

<h2>Example Setup</h2>

In order to use `setjmp/longjmp` we record a jump position that we can return
to. This position in the code is help in the variable `custom_lua_panic_jmp`,
and set before we begin interacting with the Lua VM (shown in the code snippet
below).

{% highlight cpp %}
/* jump point */
static jmp_buf custom_lua_panic_jump;

/* custom panic handler */
static int custom_lua_atpanic(lua_State *lua)
{
    longjmp(custom_lua_panic_jump, 1);
    /* will never return */
    return 0;
}
{% endhighlight %}

Next we setup the Lua VM normally, and then immediately setup the panic
handler.

{% highlight cpp %}
#include <setjmp.h>

/* build Lua VM state */
lua_State *L = luaL_newstate();
if (!L) {
  /* log error and return */
}

/* immediately install panic handler */
lua_atpanic(L, &custom_lua_atpanic);
{% endhighlight %}

Finally we set the jump point which will be used to return to this execution
context from the panic handler, and then begin to interact with the Lua VM.
Note that in the panic handler we return the value `1`, which corresponds to
the recovery case below.

{% highlight cpp %}
if (setjmp(custom_lua_panic_jump) == 0) {
  /* interact with Lua VM */
} else {
  /* recovered from panic. log and return */
}
{% endhighlight %}


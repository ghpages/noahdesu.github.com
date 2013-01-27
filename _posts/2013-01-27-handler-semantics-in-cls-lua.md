---
date: 3013-01-27
layout: post
title: Handler Semantics in Lua RADOS
---

Common Handler Patterns
-----------------------

* Cls operations failure normally abort the handler by immediately returning
* For some calls (such as stat or create) error are handled without aborting
* Class-specific failures are generated, and a negative error value is returned

When performing an I/O operation (e.g. reading an extent from an object) it is
common to simply return the error because there is little that can be done to
recover from most errors.

{% highlight cpp %}
int ret = cls_cxx_read(hctx, 0, len, &bl);
if (ret < 0)
  return ret;
{% endhighlight %}

However some operations return error codes that we may want to ignore. For
example, when retrieving a value from the object map, -ENOENT is used to
indicate that the given key was not present in the map. If the handler code can
handle this case easily (e.g. creating a new key), then it is simple enough to
just return all other error codes.

{% highlight cpp %}
int ret = cls_cxx_map_get_val(hctx, key, &bl);
if (ret < 0 && ret != -ENOENT)
  return ret;
{% endhighlight %}

xyz

{% highlight cpp %}
try {
  bufferlist::iterator it = bl.begin();
  ::decode(out, it);
} catch (const buffer::error &err) {
  CLS_ERR("error decoding bufferlist");
  return -EIO;
}
{% endhighlight %}

Lua Handler Patterns
--------------------

{% highlight lua %}
function handler(ctx, input, output)
  rados.create(ctx, true)
end
{% endhighlight %}

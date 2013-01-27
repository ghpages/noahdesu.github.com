---
layout: post
title: Simple Moving Average Awk Script
---

I often find myself with a lot of multivariate time series data. It's also
usually quite noisy, which makes for hard-to-interpret plots. Taking a simple
moving average over the variables is a good way to smooth things out.

I use the Awk script below to process my data files, which normally have a
format in which the first column is time and the remaining columns contains the
value of each variable. The script will output the first column untouched, and
will take an average over a sliding window to compute a new value for each
variable. It isn't particularly sophisticated, but it usually gets the job done
for ad hoc data visualizations.

The Script: `sma.awk`
---------------------

{% highlight awk %}
#!/bin/awk -f

BEGIN {
  window = 5
}

function sma(id, value)
{
  sma_seen[id]++;
  mod = sma_seen[id] % window;
  if (sma_seen[id] <= window)
    sma_count[id]++;
  else
    sma_sum[id] -= sma_arr[id, mod]

  sma_sum[id] += value;
  sma_arr[id, mod] = value;
  return sma_sum[id] / sma_count[id];
}

{
  printf("%d", $1);

  for (i = 2; i <= NF; i++) {
    sma_val = sma(i, $i);
    printf("\t%f", sma_val);
  }

  printf("\n");
}
{% endhighlight %}

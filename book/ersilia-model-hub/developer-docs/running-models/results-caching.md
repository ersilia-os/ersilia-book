# Results caching

Ersilia can cache a model's predictions on your machine using [Redis](https://redis.io/). When you run the model again on an input it has already seen, the result comes from the cache instead of the model. Caching is off by default.

For a permanent store of predictions that you can share with others, see [Precalculation Store](../data-storage/precalculation-store.md).

## Turn caching on

```bash
ersilia serve eos3b5e --enable_cache
```

Redis runs in a Docker container, so caching needs Docker installed and running. The first time, Ersilia downloads the Redis image and starts a container named `redis`. If Docker isn't available, the model runs without caching.

To limit how much memory the cache can use, give a fraction of your system's memory between 0 and 1. The default is 0.3:

```bash
ersilia serve eos3b5e --enable_cache --max_cache_memory_frac 0.5
```

When the cache is full, Redis drops the least recently used results first.

## How the cache works

* Results are stored per model, keyed by the input string, such as the SMILES.
* Cached results expire after seven days.
* Models with variable output, such as generative models, are never cached, because the same input can give a different result each time.
* The cache is keyed by model identifier, not version. If a model is updated, results from the previous version can still come from the cache until they expire. Clear the cache after updating a model you're testing.

## Clear the cache

The cache is stored in a Docker volume, `redis_data_volume`, so it survives restarts. To clear it, stop the Redis container, which also removes it, and delete the volume:

```bash
docker stop redis
docker volume rm redis_data_volume
```

Ersilia creates both again the next time you serve a model with caching on.

{% hint style="warning" %}
Clear the cache when you're testing models or changing their code. Otherwise you may get stale results from the cache instead of the model's current output.
{% endhint %}

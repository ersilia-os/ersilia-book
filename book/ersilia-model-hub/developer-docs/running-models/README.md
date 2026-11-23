# Running models

When you serve a model, Ersilia starts its server and sends your inputs to it in batches. These pages explain two ways that process is made faster: Warm serving keeps the model loaded in memory between batches, so it doesn't start again for each one, and Results caching stores predictions on your machine, so inputs the model has already seen return instantly.

# Data storage

Ersilia keeps two kinds of data outside Git. The Precalculation Store holds model predictions, so results for molecules a model has already seen can be reused instead of computed again. Ersilia Version Control (EOSVC) holds large files, such as model checkpoints and training data, in S3, while the code stays in GitHub.

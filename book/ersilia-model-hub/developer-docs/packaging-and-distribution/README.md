# Packaging and distribution

Every model in the Ersilia Model Hub is packaged in the same way, so it can run anywhere with the same commands. Ersilia Pack turns a model repository into a self-contained server, and that packaged model is then distributed in several forms:

* **Source code**, in the model's GitHub repository, with a copy in Ersilia's S3 storage.
* **Docker images** on Docker Hub, for AMD64 and, when possible, ARM64. This is the default when you run `ersilia fetch`.
* **Apptainer (`.sif`) images**, for HPC clusters and other places where Docker isn't available.

Ersilia's CI builds and publishes all of these automatically when a model is released. See CI/CD workflows.

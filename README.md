# Building a Confidential Flask-Based Application using `sconectl`

This is a variant of the Flask-Based Application to show how to determine which key/value pairs we need to define in the mesh file.

To do so, we first define a mesh file `mesh-1.yml` that defines

- what CAS to use
- which services to include

but we do not define any key/value pairs in section `env`.  Instead, we generate a second mesh file from the first mesh file:



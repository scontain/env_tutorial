# Building a Confidential Flask-Based Application using `sconectl`

This is a variant of the Flask-Based Application to show how to determine which key/value pairs we need to define in the mesh file.

1. To do so, we first create a mesh file `mesh-base.yaml.template` that defines

   - what CAS to use
   - which services to include in this mesh

    We can configure this template using the following environment variables:

   - `$APP_IMAGE_REPO`: the container image repo to which we push the generated container image 
   - `$APP_NAMESPACE`: the namespace used by SCONE CAS. By default we create a random namespace to avoid conflicts.
   - `$SCONECTL_REPO`: the repo used for the sconectl images. By default this is `registry.scontain.com:5050/sconectl`
   - `$RELEASE`: the name of the helm release. (We do not use this in this template file)

2. These environment variables are set by the `run.sh` script. This script generates a new mesh file in which these environment variables are replaced by their values:

   - `mesh-1.yaml` is derived from file `mesh-base.yaml.template` with the help of command `envsubst`

3. Now that we have defined the services, we can apply mesh file `mesh-1.yaml` to get a new mesh file that is a superset of `mesh-1.yaml`.

    - `mesh-2.yaml` is created by applying `mesh-1.yaml` and adding argument `--print-defaults`
    - it contains a definition of all environment variables that we need to define. These contain default values or might indicate that they do not have default values.

4. We define a `patch.yaml` that defines all environment variables that we want to overwrite or define. We create a new mesh file

    - `mesh-3.yaml` by appending `patch.yaml` to `mesh-2.yaml`

5. We now apply this final mesh file using `scone apply -f mesh-3.yaml`

6. We install the mesh using `helm install`


Note that `run.sh` automates all these steps. By default, a mesh in `production` mode is generated. The script also permits to generate a `--debug` version.




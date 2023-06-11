# Building a Confidential Service Mesh using `sconectl`

## TL'DR

```bash
# In case you want to test a release candidate of `sconectl`, you can change the repo and the VERSION
export SCONECTL_REPO=registry.scontain.com/cicd
export VERSION=5.8.0-rc.25
# if you want to use the latest stable release, ensure that these variables are not set:
unset SCONECTL_REPO
unset VERSION
# cleanup the last state
rm -rf release.sh target
# define REPO to which you are # define REPO to which you are permitted to push container images
REPO="<YOUR-REPO>"
# execute all steps of this tutorial
./run.sh -i "$REPO" --release secure-doc-management -v
```

## Motivation

When building confidential applications, we need to connect multiple confidential services into one service mesh.
In this context, we need to define lots of configuration parameters. To get a first draft regarding what parameters are needed,
we can use option `--print-defaults` to emit all the environment variables that must or can be defined.

We can then edit or patch these defaults. In this example, we just append a patch file to the emitted definitions: the values of the patch file overwrite the definitions of the emitted file.

## Details

This is a variant of the Flask-Based Application to show how to determine which key/value pairs we need to define in the mesh file.

1. To do so, we first create a mesh file `mesh-base.yaml.template` that defines

   - what CAS to use: you can specify the CAS instance by adding options `--cas` and `--cas-namespace` to the run script.
     - default is `cas` in namespace `default`

   - which services to include in this mesh

    We can configure this template using the following environment variables or via command line flags:

   - `$APP_IMAGE_REPO` / `--image_repo`: the container image repo to which we push the generated container image
   - `$APP_NAMESPACE` / `--namespace`: the policy namespace used by SCONE CAS. By default we create a random namespace to avoid conflicts.
   - `$SCONECTL_REPO` : the repo used for the sconectl images. By default this is `registry.scontain.com/sconectl`
   - `$RELEASE` / `--release`: the name of the helm release. (We do not use this in this template file)

2. These environment variables are set by the `run.sh` script. This script generates a new mesh file in which these environment variables are replaced by their values:

   - `mesh-1.yaml` is derived from file `mesh-base.yaml.template` with the help of command `envsubst`

3. Now that we have defined the services, we can apply mesh file `mesh-1.yaml` to get a new mesh file that is a superset of `mesh-1.yaml`.

    - `mesh-2.yaml` is created by applying `mesh-1.yaml` and adding argument `--print-defaults`
    - it contains a definition of all environment variables that we need to define. These contain default values or might indicate that they do not have default values.

4. We define a `patch.yaml` that defines all environment variables that we want to overwrite or define. We create a new mesh file

    - `mesh-3.yaml` by appending `patch.yaml` to `mesh-2.yaml`

5. We now apply this final mesh file using `scone apply -f mesh-3.yaml`

6. We install the mesh using `helm install`

Note that `run.sh` automates all these steps. By default, a mesh in `production` mode is generated. The script generates a debug version by setting flag `--debug`.

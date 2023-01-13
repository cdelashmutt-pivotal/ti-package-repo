# A sample Carvel Package Repository
This git repo contains the source for a simple Carvel [kapp-controller](https://carvel.dev/kapp-controller/) Package Repository.  Package repositories let you present Packages, which are bundles of Kubernetes manifests and OCI images that can be installed to a Kubernetes cluster.

## Structure
The `packages` directory contains sub-directories cooresponding to different packages we want this repo to expose.

The `.imgpkg` directory is used to capture idempotent image identifiers to any OCI images or bundles this repository refers to.  This is primarily for capturing published package bundle SHAs.

There is no magic to this particular folder structure.  I chose this structure as it is similar to the one presented by the kapp-controller tutorial, and it makes it simple to know which resources go to which package we want to publish via the repo.  When this package repository is installed in your cluster, all folders are scanned for packaging meta-data resources (packages and their versions) to apply to the cluster.

## Publishing the repo
Any time you want to add new packages or new versions of existing packages, you will need to use the [`kbld`](https://carvel.dev/kbld/) and [`imgpkg`](https://carvel.dev/imgpkg/) utilities.

### The `kbld` utility
Before publishing, you need to run the `kbld` utility to "lock" any package image bundle references that this repository is using.

You can run the following command to generate the lock file:
```
kbld -f packages/ --imgpkg-lock-output .imgpkg/images.yml
...
imgpkg push -b ghcr.io/cdelashmutt-pivotal/ti-package-repo:0.0.1 -f .
```

The `kbld` utility scans yaml files for anything that looks like the coordinates of an OCI image, and resolves that image to a particular, idempotent SHA256 reference.  This is to enable package version references to remain static and only modified when intended.  Any references to images that use tags will get put into mapping file that can be used later on by kapp-controller to get the specific package bundle image that was referenced at the time of publishing.

The `imgpkg` utility scans bundles up all the files in a directory and stores them in an OCI image format in an image registry.  It's primarly leveraging an OCI image registry as an archive storage mechanism, and not creating a runnable container image.
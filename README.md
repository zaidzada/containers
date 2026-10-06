# Containers

Podman images for neuroimaging. The Containerfiles here build FSL,
FreeSurfer, ANTs, and pycortex (on top of the FreeSurfer image) with pinned
versions. Shell aliases run each tool on files in the current directory as if
it were installed locally. Other tools come from upstream images, listed
below.

## Pre-built images

```sh
podman pull docker.io/nipreps/fmriprep:25.2.5
podman pull docker.io/nipreps/mriqc:24.0.2
podman pull docker.io/nipy/heudiconv:1.5.1
podman pull docker.io/freesurfer/synthstrip:1.8
```

## Custom builds

FSL:

```sh
podman build -f Containerfile.fsl -t fsl:6.0.7.23 .
podman build -f Containerfile.fsl --build-arg FSL_VERSION=6.0.6 -t fsl:6.0.6 .
```

FreeSurfer:

```sh
podman build -f Containerfile.freesurfer -t freesurfer:8.2.0 .
```

ANTs (official pre-built binaries):

```sh
podman build -f Containerfile.ants -t ants:2.6.5 .
podman build -f Containerfile.ants --build-arg ANTS_VERSION=2.6.2 -t ants:2.6.2 .
```

pycortex (builds on the FreeSurfer image, so build that first):

```sh
podman build -f Containerfile.pyc -t pycortex:1.4.0 .
podman build -f Containerfile.pyc --build-arg PYCORTEX_VERSION=1.3.2 -t pycortex:1.3.2 .
```

pydeface (built from upstream source):

```sh
podman build --build-arg VERSION=2.1.0 -t pydeface:2.1.0 https://github.com/poldracklab/pydeface.git#v2.1.0
```

## Usage

```bash
pod() {
  podman run --rm -v "$PWD":"$PWD" -w "$PWD" "$@";
}

fsl() {
  pod localhost/fsl:6.0.7.23 "$@";
}

synthstrip() {
  pod -v "$PWD/nifti":/data:rw freesurfer/synthstrip:1.8 "$@";
}

ants() {
  pod localhost/ants:2.6.5 "$@";
}

freesurfer() {
  pod -v "$SUBJECTS_DIR":/data/subjects \
      -v "$FS_LICENSE":/opt/freesurfer/license.txt:ro,z \
      localhost/freesurfer:8.2.0 "$@" 
}

pyc() { 
  podman run --rm -it \
      -v "$PYCORTEX_STORE":/data/pycortex-db:z \
      -v "$SUBJECTS_DIR":/data/subjects:z \
      -v "$FS_LICENSE":/opt/freesurfer/license.txt:ro,z \
      -v "$PWD":/work:z \
      -p 127.0.0.1:8900:8900 \
      localhost/pycortex:1.4.0'
}

```

**Note**: FreeSurfer and pycortex need a (free) license from <https://surfer.nmr.mgh.harvard.edu/registration.html>, saved as `~/.freesurfer/license.txt`, with `FS_LICENSE` pointing at it. `pyc` (needs `PYCORTEX_STORE`, `SUBJECTS_DIR`, and `FS_LICENSE` set to existing host paths). Then, you can use these functions like so:

```sh
fsl bet --help
ants antsRegistrationSyN.sh
freesurfer mri_convert --help
pyc           # IPython with cortex imported
pyc bash      # shell with the FreeSurfer environment loaded
```

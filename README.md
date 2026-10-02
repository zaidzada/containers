# Containers

Container images.

```sh
podman pull docker.io/nipreps/fmriprep:25.2.5
podman pull docker.io/nipreps/mriqc:24.0.2
podman pull docker.io/nipy/heudiconv:1.5.1
podman pull docker.io/freesurfer/synthstrip:1.8
podman build --build-arg VERSION=2.1.0 -t pydeface:2.1.0 https://github.com/poldracklab/pydeface.git#v2.1.0
```

## Aliases

```sh
alias pod='podman run --rm -v "$PWD":"$PWD" -w "$PWD"'
alias fsl='pod localhost/fsl:6.0.7.23'
alias freesurfer='pod -v "$SUBJECTS_DIR":/data/subjects -v "$HOME"/.freesurfer/license.txt:/opt/freesurfer/license.txt:ro,z localhost/freesurfer:8.2.0'
```

Usage:

```sh
fsl siena --help
freesurfer mri_convert --help
```

## FSL

Build with the default FSL version (6.0.7.23), or pick another with `FSL_VERSION`:

```sh
podman build -f Containerfile.fsl -t fsl:6.0.7.23 .
podman build -f Containerfile.fsl --build-arg FSL_VERSION=6.0.6 -t fsl:6.0.6 .
```

## FreeSurfer

```sh
podman build -f Containerfile.freesurfer -t freesurfer:8.2.0 .
```

## pycortex

1. **Build the image** on top of the FreeSurfer image, so build that first
   (see [FreeSurfer](#freesurfer)):
   ```sh
   podman build -f Containerfile.pyc -t pycortex:1.4.0 .
   podman build -f Containerfile.pyc --build-arg PYCORTEX_VERSION=1.3.2
   # or pick the pycortex version and/or FreeSurfer base image:
   podman build -f Containerfile.pyc --build-arg PYCORTEX_VERSION=1.3.2 \
       --build-arg BASE_IMAGE=localhost/freesurfer:x.y.z -t pycortex:1.3.2 .
   ```

2. **Get a FreeSurfer license** (free) at
   <https://surfer.nmr.mgh.harvard.edu/registration.html> and save it as
   `~/.freesurfer/license.txt`.

3. **Add the alias** below (see [Files](#files)) to `~/.zshrc` or `~/.bashrc`.

### Usage

```sh
pyc                         # IPython with cortex imported
pyc recon-all -s sub01 -all # run any command instead
pyc bash                    # shell with the FreeSurfer environment loaded
```

The alias mounts these host paths:

| Host (override with env var) | Default | In container |
|---|---|---|
| `PYCORTEX_STORE` | `~/projects/pycortex-db` | `/data/pycortex-db` |
| `SUBJECTS_DIR` | `~/freesurfer/subjects` | `/data/subjects` |
| `FS_LICENSE` | `~/.freesurfer/license.txt` | `/opt/freesurfer/license.txt` |
| current directory | `$PWD` | `/work` (working dir) |

All of these paths must exist on the host, or podman won't start.

## Notes

- **Web viewer:** run `cortex.webshow(data, port=8900)`, then open
  <http://localhost:8900> on the host.
- **Plots:** there's no display in the container (`MPLBACKEND=Agg`). Save
  figures to files, e.g. with `cortex.quickflat.make_png(...)` or
  `plt.savefig(...)`; anything written under `/work` ends up in your current
  directory.
- **Importing FreeSurfer subjects:** `cortex.freesurfer.import_subj("sub01")`
  reads from `/data/subjects` and writes to the mounted pycortex store.
- **File ownership:** with rootless podman, files the container writes are
  owned by your host user.
- **Config:** pycortex settings in the container come from
  the options file written in `Containerfile.pyc`, not your host
  `~/.config/pycortex/options.cfg`. Rebuild after editing it.

## Files

```bash
# Paste into ~/.zshrc or ~/.bashrc.
#
# Host paths (override by exporting before use):
#   PYCORTEX_STORE  pycortex filestore          (default: ~/projects/pycortex-db)
#   SUBJECTS_DIR    FreeSurfer subjects dir      (default: ~/freesurfer/subjects)
#   FS_LICENSE      FreeSurfer license.txt       (default: ~/.freesurfer/license.txt)
# The current directory is mounted at /work. Port 8900 is published for cortex.webshow.
#
#   pyc                      -> IPython with cortex imported
#   pyc recon-all -s ... -all -> run any command instead
#   pyc bash                 -> shell with FreeSurfer env loaded

alias pyc='podman run --rm -it \
  -v "${PYCORTEX_STORE:-$HOME/projects/pycortex-db}":/data/pycortex-db:z \
  -v "${SUBJECTS_DIR:-$HOME/freesurfer/subjects}":/data/subjects:z \
  -v "${FS_LICENSE:-$HOME/.freesurfer/license.txt}":/opt/freesurfer/license.txt:ro,z \
  -v "$PWD":/work:z \
  -p 127.0.0.1:8900:8900 \
  localhost/pycortex:1.4.0'
```

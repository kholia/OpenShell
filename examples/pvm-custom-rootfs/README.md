# PVM custom rootfs development example

Build a custom OCI rootfs and run it with the experimental PVM-backed VM
driver. Use this example only for local driver testing and development.

> [!WARNING]
> The included policy is intentionally broad. It authorizes every sandbox
> binary to use L4 connections to the listed package and source-code hosts.
> Do not use it for production workloads or with untrusted code.

OpenShell always runs sandbox workloads as a non-root identity. It rejects UID
or GID 0 and sets `no_new_privs`, so installing `sudo` in the image does not
enable runtime elevation. Add packages to `dev-packages.txt` so the image build
installs them as root before OpenShell starts the sandbox.

## Prerequisites

- A Linux x86-64 host with KVM available
- An uncompressed x86-64 PVM guest `vmlinux`
- A compatible qboot firmware ROM
- `qemu-system-x86_64` on `PATH`
- Docker or a running Podman API socket for the local OCI build

## Customize the image

Edit `dev-packages.txt` and add one Debian or Ubuntu package name per line.

To install internal NV packages without committing them, copy the artifacts
into the ignored `local-debs/` directory:

```shell
cp /secure/path/nvidia-example_1.0.0_amd64.deb \
  examples/pvm-custom-rootfs/local-debs/
git status --short
```

The `.deb` file should not appear in `git status`. During the local image
build, the Dockerfile passes all `.deb` files to `apt-get install` together so
dependencies can be resolved from the Ubuntu repositories. It then removes the
package archives from the final rootfs.

Both the installed package contents and the copied archive can remain
recoverable from OCI image layers. Do not push the resulting image to a public
registry; retain it in the local daemon or an access-controlled private
registry.

The image includes the tools required by the VM guest setup, creates a
non-root OCI user for Docker and Podman compatibility, and prepares `/sandbox`
as the writable workspace. The VM driver injects its configured sandbox
identity and continues to use `/sandbox` when it prepares the rootfs.

## Start the PVM gateway

From the repository root, stage the VM runtime and start the development
gateway. Both PVM paths are required together:

```shell
mise run vm:setup
mise run vm:supervisor

OPENSHELL_VM_PVM_KERNEL=/absolute/path/to/vmlinux-guest \
OPENSHELL_VM_PVM_FIRMWARE=/absolute/path/to/qboot.rom \
mise run gateway:vm
```

The gateway registers itself as `vm-dev`. Keep it running and use another
terminal for the sandbox commands.

## Create and test the sandbox

Build the directory as an OCI image, create the sandbox, and apply the
development-only policy:

```shell
openshell --gateway vm-dev sandbox create \
  --name pvm-rootfs-dev \
  --from examples/pvm-custom-rootfs \
  --policy examples/pvm-custom-rootfs/policy.dev-only.yaml
```

Confirm that the workload is non-root and that customized packages are
present:

```shell
openshell --gateway vm-dev sandbox exec --name pvm-rootfs-dev -- id
openshell --gateway vm-dev sandbox exec --name pvm-rootfs-dev -- \
  dpkg-query -W build-essential git jq
```

The policy uses L4 pass-through rather than HTTP inspection. It permits every
method and path on its listed hosts, and `binaries: []` permits every workload
binary. OpenShell does not accept a match-all host wildcard, so add exact hosts
or permitted domain suffix patterns when another package repository is needed.

## Cleanup

```shell
openshell --gateway vm-dev sandbox delete pvm-rootfs-dev
```

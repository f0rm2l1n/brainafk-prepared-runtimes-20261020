# Prepared Docker runtimes

The [v2026.10.20 release](https://github.com/f0rm2l1n/brainafk-prepared-runtimes-20261020/releases/tag/v2026.10.20) has two separate Docker image archives:

| Asset | Image tag | What is inside |
| --- | --- | --- |
| `civetweb_web_docker-image.tar` | `brainafk-civetweb-asan-ready:20261020` | CivetWeb source, an ASAN-built server, its service configuration, and preset helpers. |
| `linux_rxrpc_kasan_qemu-image.tar` | `brainafk-linux-rxrpc-kasan-ready:20261020` | A KASAN `bzImage` and `vmlinux`, BusyBox initramfs, QEMU, RxGK lab fixtures, and preset helpers. |

The full Linux source tree is not in the Linux image. Mount the matching source at `/workspace/code` if an agent must change and rebuild the kernel. Both images contain the preset resources at `/opt/brainafk/preset`.

## Import and start manually

Download the two assets from the release, then run on the Docker host:

```sh
docker load -i civetweb_web_docker-image.tar
docker load -i linux_rxrpc_kasan_qemu-image.tar

docker run -d --name brainafk-civetweb -p 127.0.0.1:8080:8080 \
  brainafk-civetweb-asan-ready:20261020
docker run -d --name brainafk-rxrpc --entrypoint sleep \
  brainafk-linux-rxrpc-kasan-ready:20261020 infinity
```

The CivetWeb container starts its HTTP service. The Linux container stays ready for the agent to launch QEMU when a test needs it. QEMU uses KVM when the container has access to `/dev/kvm`; otherwise it uses software emulation.

Give the agent a shell in the appropriate running container:

```sh
docker exec -it brainafk-civetweb bash
docker exec -it brainafk-rxrpc bash
```

In a BrainAFK managed run, the platform starts a separate container for each task and supplies the agent's shell. These host commands are for manual use. Loading an archive does not automatically change the original preset builder to use that image tag.

## Agent guides

- [CivetWeb environment guide](civetweb_web_docker/ENVIRONMENT_GUIDE.md): HTTP and ASAN testing after the agent enters a ready container.
- [Linux RxRPC environment guide](linux_rxrpc_kasan_qemu/ENVIRONMENT_GUIDE.md): kernel, QEMU, initramfs, and KASAN testing after the agent enters a ready container.

SHA-256 of the release assets:

```text
91f7ec8803d64e615edec0ec2f74dac114f7ca0521c19f3d4073083d1facd21f  civetweb_web_docker-image.tar
43a5573bc0733a2b2d86ec1c70494df9862ed76ef659e9fbc697bd64e73cccc9  linux_rxrpc_kasan_qemu-image.tar
```

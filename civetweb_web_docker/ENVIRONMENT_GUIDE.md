# CivetWeb ASAN: agent environment guide

This guide starts after the platform has given the agent a shell in a ready CivetWeb container. The service listens on container port `8080`; use `http://127.0.0.1:8080/` from inside the container. The source snapshot is commit `d7ba35bbb649209c66e582d5a0244ba988a15159`.

## Working locations

| Path | Purpose |
| --- | --- |
| `/workspace/code` | CivetWeb source and ASAN-built `civetweb` binary; the platform may mount a writable task workspace here. |
| `/workspace/customization` | `build.command.sh`, `run.command.sh`, `verify.command.sh`, and `civetweb.conf`. |
| `/workspace/runtime-www` | Test document root. |
| `/artifacts` | Service access/error logs and task evidence. |
| `/opt/brainafk/preset` | Original preset resources and builder. |

## Dynamic testing

1. Establish the baseline with `curl -i http://127.0.0.1:8080/` and `bash /workspace/customization/verify.command.sh`.
2. Inspect or change code under `/workspace/code`. Rebuild with `cd /workspace/code && bash /workspace/customization/build.command.sh`; this uses `TEST_ASAN=1`.
3. Ask the managed runtime to restart the service after a rebuild. In a manually started container, the operator can restart that same container. Then send the test HTTP request again and compare the response with the baseline.
4. Collect the request, response, relevant `/artifacts/civetweb-*.log` entries, and any ASAN report from the managed container's stderr. A successful build alone is not a dynamic test.

The server uses `/workspace/customization/civetweb.conf`. Its default document root is `/workspace/runtime-www`; its HTTP port is `8080`. Source edits inside a disposable container disappear when that container is removed unless the task workspace is mounted externally.

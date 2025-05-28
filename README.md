# kubectl-admin

`kubectl-admin` is a [Krew](https://krew.sigs.k8s.io/) plugin that simplifies node-level debugging in Kubernetes.

It launches a privileged debug pod on a specified node using `kubectl debug`, chroots into the host filesystem, automatically cleans up the pod after exit, and saves the debug session log for auditing.

---

## Features

- 🔧 Launches a temporary debug pod with `chroot /host`
- 🖥 Automatically attaches to an interactive shell
- 🧼 Cleans up the pod after exit
- 📝 Saves session log to `~/logs/kubectl-admin/`
- 💡 Optional support for specifying image, profile, and namespace

---

## Requirements

- `kubectl` version ≥ 1.18 (with `kubectl debug` support)
- The `script` command (part of `util-linux`)

### Install `script`:

On RHEL/CentOS/Rocky:
```bash
sudo dnf install util-linux
```

On Debian/Ubuntu:
```bash
sudo apt install util-linux
```

---

## Installation

Install via Krew:

```bash
kubectl krew install admin
```

> Make sure Krew is [installed](https://krew.sigs.k8s.io/docs/user-guide/setup/) first.

---

## Usage

```bash
kubectl admin <NODE_NAME> [IMAGE] [PROFILE] [NAMESPACE]
```

### Example:

```bash
kubectl admin node-01 rockylinux:9 sysadmin default
```

This will:
- Start a pod on `node-01` using `rockylinux:9`
- Use the `--profile=sysadmin` (if supported by your `kubectl`)
- Attach to a chrooted shell in `/host`
- Record the session
- Automatically delete the pod on exit

---

## Compatibility: `--profile` behavior

The `--profile` flag in `kubectl debug` changes across Kubernetes versions:

| Kubernetes Version | Available Profiles           | Notes                                                   |
|--------------------|------------------------------|----------------------------------------------------------|
| ≤ v1.25            | Not supported                | `--profile` is not available                            |
| v1.26–v1.30        | `legacy` (default), `general`| `--profile=legacy` is used but deprecated                |
| v1.31+             | `general`, `baseline`, etc.  | `legacy` removed; `--profile=legacy` may warn or fail   |
| v1.33+             | `sysadmin`, `restricted`, …  | `--profile=sysadmin` newly supported in many setups     |

This plugin detects your `kubectl` version and chooses a supported profile
based on the context. If no profile is specified, a version-appropriate default is used.

To manually check supported profiles:

```bash
kubectl debug --help | grep -- --profile
```

If an unsupported profile is used, you may see:

```text
error: unknown profile: sysadmin
```

---


## Log Storage

- Logs are saved under `~/logs/kubectl-admin/`
- Filenames include the node name and timestamp
- Example:
  ```
  ~/logs/kubectl-admin/debug-node-01-20250528-145300.log
  ```

---

## Troubleshooting

If the pod fails to delete automatically:

```bash
kubectl get pods -A | grep node-debugger
kubectl delete pod <pod-name> -n <namespace>
```

---

## License

MIT License. See [LICENSE](LICENSE) for details.


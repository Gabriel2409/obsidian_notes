#k8s

## kubectl

- kubectl is kubernetes cli

```bash
VERSION=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)
curl -LO https://storage.googleapis.com/kubernetes-release/release/$VERSION/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

## Krew

Krew is kubectl plugin manager: https://github.com/kubernetes-sigs/krew/
Installation instructions: https://krew.sigs.k8s.io/docs/user-guide/setup/install/

```bash
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)
```

Then in .bashrc:

```
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```

## kubectx and kubens

kubectx is a tool to switch between contexts (clusters) on kubectl faster.
kubens is a tool to switch between Kubernetes namespaces (and configure them for kubectl) easily.
https://github.com/ahmetb/kubectx

To install them with Krew:

```
kubectl krew install ctx
kubectl krew install ns
```

- switch context:
- kubectl: `kubectl config use-context <context-name>`
- plugin: `kubectl ctx <context-name>` (replace cmd with `kubectx` if not installed with Krew)

- switch namespace:
  - kubectl: `kubectl config set-context $(kubectl config current-context) --namespace=<name>`
  - plugin: `kubectl ns <name>` (replace cmd with `kubens` if not installed with Krew)

## multipass

IMPORTANT NOTE FOR WSL. When creating multipass machines and installing kubernetes on them,
kubectl is not able to access them because of network rules.
To solve this issue,

- we can launch this from powershell as admin: `Get-NetIPInterface | where {$_.InterfaceAlias -eq 'vEthernet (WSL)' -or $_.InterfaceAlias -eq 'vEthernet (Default Switch)'} | Set-NetIPInterface -Forwarding Enabled` (see https://stackoverflow.com/questions/65716797/cant-ping-ubuntu-vm-from-wsl2-ubuntu).
- we can also do some port forwarding: see https://pypi.org/project/WSL-Port-Forwarding/

NOTE: I am still unclear if we need both of them or just the first

- Install multipass: https://multipass.run/ : allows to easily launch a VM with either hyperV or virtual box
  To use it on WSL, it must be installed on windows, not on linux. And then, we can create
  an alias to use it directly on WSL: for ex: `alias multipass="/mnt/c/'Program Files'/Multipass/bin/multipass.exe"`
- set driver to hyper V: `multipass set local.driver=hyperv` or virtualbox: `multipass set local.driver=virtualbox`

- IMPORTANT: to use the mount features in WSL, you must be in a folder accessible by windows, such as `/mnt/c/...`
- enable mounting with `multipass set local.privileged-mounts=true`
- create a vm:

  - basic : `multipass launch node1`
  - with parameters: ` multipass launch -n node2 -c 2 -m 3G -d 10G` (2 cores, 3G ram, 10G disk size)

- start/stop/delete vm: `multipass start node1` / `multipass stop node1` / `multipass delete -p node1`
- get infos: `multipass infos node2`
- list vms: `multipass list`
- launch a shell: `multipass shell node1`
- launch a command on the node, for ex:
  - install docker: `multipass exec node1 -- /bin/bash -c "curl -sSL https://get.docker.com | sh"` (here, it launches the bash shell and execute the command between quotes. It is the same as getting into the sell and executing the command between quotes)
  - check that docker is correctly installed: `multipass exec node1 -- sudo docker version`
- copy file from local machine: `multipass transfer /tmp/test/hello node1:/tmp/hello` (both ways work)
- mount a folder: `multipass mount ./testfolder node1:/usr/share/test`.
  Note that if you want to use an absolute path: `multipass mount $(wslpath -w /mnt/d/projects/kubernetes-tuto/test2/) node1:/usr/share/test`
- unmount a folder: `multipass umount node1:/usr/share/test` or unmount all : `multipass umount node1`
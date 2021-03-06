# Boot a Virtual Machine in G8OS Resource Pool

Setting up a resource pool takes three steps:

- [Setup the AYS Server](#setup-ays)
- [Deploy the G8OS nodes to packet.net](#deploy-nodes)
- [Create Storage Cluster](#storagecluster)
- [Deploy VMs](#deployvms)


<a id="setup-ays"></a>
## Setup the AYS Server

* Install JumpScale 8.2

  On the machine where you want to run the AYS Server execute:

  ```bash
  curl -sL https://raw.githubusercontent.com/Jumpscale/developer/master/scripts/js_builder_js82_zerotier.sh?$RANDOM | bash -s <your-ZeroTier-network-ID>
  ```

  To see interactive output do the following in a separate console:

  ```bash
  tail -f /tmp/lastcommandoutput.txt
  ```

  For more details about using `js_builder_js82_zerotier.sh` see [here](https://github.com/Jumpscale/developer/blob/master/docs/installjs8_details.md).



* Add a G8OS resourcepool to your JumpScale 8.2 environment

  This script is based on the JumpScale 8.2 environment above:

  ```bash
  curl -sL https://raw.githubusercontent.com/Jumpscale/developer/master/scripts/g8os_grid_installer82.sh?$RANDOM | bash -s <Branch> <your-ZeroTier-network-ID> <your-ZeroTier-Token>
  ```

  Again, to see interactive output do the following in separate console:

  ```
  tail -f /tmp/lastcommandoutput.txt
  ```

<a id="deploy-nodes"></a>
## Deploying G8OS nodes on packet.net

- Copy [nodes.yaml](./packet.net-resourcepool-10nodes-100vms/blueprints/nodes.yaml) to ays repo `/optvar/cockpit_repos/resourcepool-server` and edit it to match your configuration
- Run ays service to start deploying

  ```bash
  ays blueprint
  ays run create --follow
  ```

<a id="storagecluster"></a>
## Create Storage Cluster

Execute [createcluster.py](./scripts/createcluster.py):

  `python3 createcluster.py --servers 50 --resourcepoolserver http://{ipaddress}:{port}`

  Options:
  - `--servers` number of servers in the storagecluster'
  - `--resourcepoolserver` resourcepool api server endpoint

From your js docker:
```shell
cd /opt/code/gogs/g8os
git clone ssh://git@docs.greenitglobe.com:10022/g8os/demo.git
cd demo/packet.net-resourcepool-10nodes-100vms/scripts
python3 createcluster.py --servers 50 --resourcepoolserver http://192.168.193.212:8080
```
(Replace the number of servers, and url according to your setup)

<a id="deployvms"></a>
## Deploy VMs

Execute [deployvms.py](./scripts/deployvms.py) (It may take long time):

  `python3 deployvms.py [OPTIONS]`

Options:
  - `--resourcepoolserver`     Resourcepool api server endpoint. Eg: http://{ipaddress}:{port}
  - `--storagecluster`         Storagecluster name, e.g: `bigmomma`
  - `--vms`                    Number of vms to create
  - `--bootdisktemplate`       Virtual machine template name, e.g: `ardb://hub.gig.tech:16379/template:ubuntu-1604`
  - `--bootdisksize`           Boot disk size (GiB)
  - `--datadisksize`           Data disk size (GiB)
  - `--memory`                 Memory (MiB)

```shell
cd /opt/code/gogs/g8os/demo/packet.net-resourcepool-10nodes-100vms/scripts/
python3 deployvms.py --resourcepoolserver http://192.168.193.212:8080 --storagecluster bigmomma --vms 1 --bootdisksize 20 --datadisksize 100 --memory 1024 --bootdisktemplate ardb://hub.gig.tech:16379/template:ubuntu-1604
```
(Replace the number of servers, and url according to your setup)

# IoTeX Delegate Manual

## News
- We found a bug in `v0.9.0` which may cause the nodes not agree on the delegates list. We already pushed out a build `v0.9.1` to address this issue.
- `v0.9.0` is released, so that delegates should upgrade their softwares to this new version. The fork will happen at block height 1641601. Before restarting with `v0.9.0` docker image, please re-pull the up-to-date mainnet genesis config file first. It's a MUST step for this upgrade. In addtion, note that this upgrade will result in db migration upon restart which could takes 30min to 1hr to complete. Therefore, please upgrade when the delegate node is not in the active consensus epoch.
- We have reset testnet, and deployed `v0.8.3`, and finally upgraded it to `v0.9.0`. The genesis config file has been updated as well.
- We have upgraded mainnet to `v0.8.3`. It contains breaking changes which will be activated on block height 1512001. Delegates must upgrade your node to the new version before that.
- We have upgraded testnet to `v0.8.4`.
- We have upgraded testnet to `v0.8.3`, which contains the new error code, and consensus improvement.

## Index

- [Release Status](#status)
- [Join MainNet Beta](#mainnet)
- [Join TestNet](#testnet)
- [Interact with Blockchain](#ioctl)
- [Operate Your Node](#ops)


## <a name="status"/>Release Status

Here are the software versions we use:

- MainNet: v0.9.1
- TestNet: v0.9.1

## <a name="mainnet"/>Join MainNet Beta

1. Pull the docker image:

```
docker pull iotex/iotex-core:v0.9.1
```

2. Set the environment with the following commands:

```
mkdir -p ~/iotex-var
cd ~/iotex-var

export IOTEX_HOME=$PWD

mkdir -p $IOTEX_HOME/data
mkdir -p $IOTEX_HOME/log
mkdir -p $IOTEX_HOME/etc

curl https://raw.githubusercontent.com/iotexproject/iotex-bootstrap/master/config_mainnet.yaml > $IOTEX_HOME/etc/config.yaml
curl https://raw.githubusercontent.com/iotexproject/iotex-bootstrap/master/genesis_mainnet.yaml > $IOTEX_HOME/etc/genesis.yaml
```

3. Edit `$IOTEX_HOME/etc/config.yaml`, look for `externalHost` and `producerPrivKey`, uncomment the lines and fill in your external IP and private key.

4. (Optional) If you prefer to start from a snapshot, run the following commands:

```
curl -L https://t.iotex.me/mainnet-data-latest > $IOTEX_HOME/data.tar.gz
tar -xzf data.tar.gz
```
We will update the snapshot once a day. If you plan to run your node as a gateway, please use the snapshot with index data:
https://t.iotex.me/mainnet-data-with-idx-latest.

5. Run the following command to start a node:

```
docker run -d --restart on-failure --name iotex \
        -p 4689:4689 \
        -p 8080:8080 \
        -v=$IOTEX_HOME/data:/var/data:rw \
        -v=$IOTEX_HOME/log:/var/log:rw \
        -v=$IOTEX_HOME/etc/config.yaml:/etc/iotex/config_override.yaml:ro \
        -v=$IOTEX_HOME/etc/genesis.yaml:/etc/iotex/genesis.yaml:ro \
        iotex/iotex-core:v0.9.1 \
        iotex-server \
        -config-path=/etc/iotex/config_override.yaml \
        -genesis-path=/etc/iotex/genesis.yaml
```

Now your node should be started successfully.

If you want to also make your node be a gateway, which could process API requests from users, use the following command instead:

```
docker run -d --restart on-failure --name iotex \
        -p 4689:4689 \
        -p 14014:14014 \
        -p 8080:8080 \
        -v=$IOTEX_HOME/data:/var/data:rw \
        -v=$IOTEX_HOME/log:/var/log:rw \
        -v=$IOTEX_HOME/etc/config.yaml:/etc/iotex/config_override.yaml:ro \
        -v=$IOTEX_HOME/etc/genesis.yaml:/etc/iotex/genesis.yaml:ro \
        iotex/iotex-core:v0.9.1 \
        iotex-server \
        -config-path=/etc/iotex/config_override.yaml \
        -genesis-path=/etc/iotex/genesis.yaml \
        -plugin=gateway
```

6. Make sure TCP ports 4689, 8080 (also 14014 if used) are open on your firewall and load balancer (if any).

## <a name="testnet"/>Join TestNet

There's almost no difference to join TestNet, but in step 2, you need to use the config and genesis files for TestNet:

```
curl https://raw.githubusercontent.com/iotexproject/iotex-bootstrap/master/config_testnet.yaml > $IOTEX_HOME/etc/config.yaml
curl https://raw.githubusercontent.com/iotexproject/iotex-bootstrap/master/genesis_testnet.yaml > $IOTEX_HOME/etc/genesis.yaml
```

In step 4, you need to use the snapshot for TestNet: https://t.iotex.me/testnet-data-latest and https://t.iotex.me/testnet-data-with-idx-latest. 

In step 5, you need to replace the docker image tag in the command with `v0.9.1`.

## <a name="ioctl"/>Interact with Blockchain


You can install `ioctl` (a command-line interface for interacting with IoTeX blockchain)

```
curl https://raw.githubusercontent.com/iotexproject/iotex-core/master/install-cli.sh | sh
```

You can point `ioctl` to your node (if you enable the gateway plugin):

```
ioctl config set endpoint localhost:14014 --insecure
```

Or you can point it to our nodes:

- MainNet secure: `api.iotex.one:443`
- MainNet insecure: `api.iotex.one:80`
- TestNet secure: `api.testnet.iotex.one:443`
- TestNet insecure: `api.testnet.iotex.one:80`

If you want to set an insecure endpoint, you need to add `--insecure` option.

Generate key:
```
ioctl account create
```

Get consensus delegates of current epoch:
```
ioctl node delegate
```

Refer to [CLI document](https://github.com/iotexproject/iotex-core/blob/master/ioctl/README.md) for more details.

### Other Commonly Used Commands

Claim reward:
```
ioctl action claim ${amountInIOTX} -l 10000 -p 1 -s ${ioAddress|alias}
```

Exchange IoTeX native token to ERC20 token on Ethereum via Tube service:
```
ioctl action invoke io1p99pprm79rftj4r6kenfjcp8jkp6zc6mytuah5 ${amountInIOTX} -s ${ioAddress|alias} -l 400000 -p 1 -b d0e30db0
```
Click [IoTeX Tube docs](https://github.com/iotexproject/iotex-bootstrap/blob/master/tube/tube.md) for detailed documentation of the tube service.

## <a name="ops"/>Operate Your Node

### Checking Node log

Container logs can be accessed with the following command. 

```
docker logs iotex
```

Content can be filtered with:

```
docker logs -f --tail 100 iotex |grep --color -E "epoch|height|error|rolldposctx"
```

### Stop and remove container

When starting the container with ```--name=iotex```, you must remove the old container before a new build.

```
docker stop iotex
docker rm iotex
```

### Pause and Restarting container

Container can be "stopped" and "restarted" with:

```
docker stop iotex
docker start iotex
```

# Docker Monerod

Run `monerod` using Docker.

This image clones `https://github.com/monero-project/monero` and builds it according to instructions.

## Usage

Append any `monerod` params after the image name. i.e:

```shell
docker run -it --rm lchaia/monerod --help
```

### Blockchain Volume

The blockchain will get downloaded at `/home/monero/.bitmonero`, so you should
volume-map that path for persistance

## Permission issues

If experiencing permission issues on the volume-mapped folder these can be solved
with a chown like below:

```shell
docker run -it --rm -v /etc/monero-blockchain:/home/monero/.bitmonero \
--entrypoint chown --user root lchaia/monerod monero:monero -R /home/monero/.bitmonero
```

## Docker Compose example

You'll need to modify it for your environment

```yaml
version: "3"

services:
  monerod:
    image: lchaia/monerod:v0.17.3.2 # or latest and use watchtower to keep up to date
    container_name: monerod
    volumes:
      - /home/my-user/.bitmonero:/home/monero/.bitmonero
    ports:
      - 18080:18080
      - 18089:18089
    command:
      - "--non-interactive"
      - "--restricted-rpc"
      - "--rpc-bind-ip=0.0.0.0"
      - "--rpc-bind-port=18089"
      - "--confirm-external-bind"
      - "--max-concurrency=4"
      - "--public-node"
```

## Copyrights

Code from this repository is released under MIT license.
[Monero License](https://github.com/monero-project/monero/blob/master/LICENSE)

## Donations

Please consider [donating to Monero](https://github.com/monero-project/monero/tree/release-v0.15#supporting-the-project)

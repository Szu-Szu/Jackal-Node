# Jackal-Node
This is how to build a Jackal Node


# Building a node for Jackal step by step

## Prep the server

First prep the node by updating.  I typically recommend you use the latest LTS of Ubuntu

`apt update`
`apt upgrade -y`

You can download pretty much all packages you need from this single line command. 

`sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y`

Next install Go.  This will remove any instances of Go on the server and install a fresh runtime.
```
rm -rf $HOME/go
rm -rf /usr/local/go
cd ~
curl https://dl.google.com/go/go1.21.1.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
```

Let's check to make sure you have Go installed correctly.

`go version`

You should see the installed version of Go here if you did it correctly. 

## Making the node

If you have any instances of the chain on the server already let's remove it.

`rm -rf canine-chain`

Clone the repo on to the server

`git clone https://github.com/JackalLabs/canine-chain`

Change into the canine directory

`cd canine-chain`

Checkout the latest version of canined

```
git fetch --all
git checkout v3.1.1 (or the latest version)
```
`make install`

Now check to make sure you have the correct version installed.  

`canined version`

Now we can initialize your node.  At this momement we'll also be naming your node

`canined init ["Your_node_name_here"] --chain-id=jackal-1`

Now let's replace our genesis file. We'll be using some Polkachu resources here.
[here is their website](https://polkachu.com/networks/jackal)


`wget -O genesis.json https://snapshots.polkachu.com/genesis/jackal/genesis.json --inet4-only`

Now let's move the genesis file in the correct location

`mv genesis.json ~/.canine/config`

Now, let's download some peers so our node can connect to the network.

`PEERS=ea35106e43dcec1e5c66319272da48df3dce7723@57.128.144.233:26656,bdb9cc3833ecba797cd2275e0bd81c2a7fc410b1@168.119.90.246:17556,fd9e27da1ebd67ea7577bf7840d3df8ea918daff@109.236.88.64:46656,3e90e3b191dd69213d820571fd654c3c60f8eee3@5.181.190.157:12856,34135f1a6585fa21ab313fb1be8fe537ff16e6bc@65.109.29.150:13756`

Now let's put those peers in the right location.

`sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.canine/config/config.toml`

Now let's import an address book.  Normally this step can be skipped as the node will build an address book by itself.  But we can speed up the process by just importing one.

`wget -O addrbook.json https://snapshots.polkachu.com/addrbook/jackal/addrbook.json --inet4-only`

Now let's move the file to the correct location.

`mv addrbook.json ~/.canine/config`

Now, we want to make sure the node starts at the correct place.  In order for the node to start it will need some data to kick it off.  Let's download a snapshot. Normally these are fairly small.

### Note if you haven't installed some prereqs to download this you should do this now.

```
sudo apt install snapd -y
sudo snap install lz4 -y
```

This can take awhile depending on the size of the file and internet speeds

`wget -O jackal_6211534.tar.lz4 https://snapshots.polkachu.com/snapshots/jackal/jackal_6211534.tar.lz4 --inet4-only`

Now let's decompress that file and place it in the right spot.

`lz4 -c -d jackal_6211534.tar.lz4  | tar -x -C $HOME/.canine`

Let's delete the file to save some space.

`rm -v jackal_6211534.tar.lz4`

Make sure you are back in your home directory `cd`

Now, let's see if the node is working.  We can do a quick check to see if the node will start up.

`canined start`

After a few moments you should see the node start to catch up with the rest of the network.

Let's stop the node

`Ctrl+C`

Let's make a service file in order to let this node work in the background.

```
sudo tee /etc/systemd/system/canined.service > /dev/null <<EOF
[Unit]
Description=canined Daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$(which canined) start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

`sudo systemctl enable canined`
`sudo systemctl daemon-reload`
`sudo systemctl start canined.service`

You should let this daemon run for a few moments.  

You can always check to see if the service is running correctly with some log files.  If you are ever having trouble and you ask on the discord to see what is wrong with your node you may be asked to provide some logss.  Here is how you do that.

`journalctl -u canined -f`

You can check to see if you node is all caught up with the network by checking the status

`canined status |grep catching`

Look at the highlighted word after the output.  It should be the "catching"  After taht you should see a true or false value.  If that value is **true** then the node has not finished catching up.  If that value is **false** then the node is finished catching up and it is completely synced with the network as of the current version.














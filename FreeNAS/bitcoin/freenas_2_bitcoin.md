[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

[ [Intro](README.md) ] - [ [Jail Creation](freenas_1_jail_creation.md) ] - [**Bitcoin**] - [ [Tor & i2p](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [loopd ](freenas_5a_loopd.md)] - [ [RTL](freenas_6_rtl.md) ] - [ [mempool](freenas_8_mempool.md) ] - [ [Extras](extras.md) ] 

## TrueNASnode - full bitcoin stack deployment guide ![BSDBTC60.png](images/BSDBTC60.png)

Join the chatroom on the matrix chat protocol: [#truenasnode:nym.im](https://matrix.to/#/#truenasnode:nym.im)

### Bitcoin Install

View the [Extras](extras.md) page if you want to manually compile bitcoind.

Secure Socket Shell into your truenas server. SSH is a way to manage your server remotely over a network. When you don’t plug in a monitor & keyboard directly into the server, it’s called a ‘headless’ server. The most popular SSH client is called [PuTTY, download it here](https://www.putty.org/). Connect to your trueNAS’ IP address, and log in with your root credentials.

```
# iocage list
```

You should see your bitcoin jail listed. Lets switch our console from our base system to our jail.

```
# iocage console bitcoin
```

You're in! Install `bitcoin-daemon`
```
# pkg install -y bitcoin-daemon bitcoin-utils nano
# sysrc bitcoind_enable="YES"
# cat <<EOT >> /usr/local/etc/bitcoin.conf
server=1
txindex=1
zmqpubrawblock=tcp://127.0.0.1:28332
zmqpubrawtx=tcp://127.0.0.1:28333
EOT
# nano /usr/local/etc/rc.d/bitcoind
```

Change the pid file in the rc.d startup script. 

Remove the `#` comment before `pidfile="/var/run/${name}.pid"`

Delete the entire line: `pidfile="${bitcoind_data_dir}/bitcoind.pid"`

Save (Ctrl+O, ENTER) and exit (Ctrl+X)

Some apps, like `lnd`, look for the config file in the bitcoin data directory. It is FreeBSD tradition to keep config files in `/usr/local/etc`. So lets make a hard link so the config file exists in both spots. Changing one will change the other. Then start bitcoind and check sync progress:

```
# ln /usr/local/etc/bitcoin.conf /var/db/bitcoin/bitcoin.conf
# service bitcoind start
# bitcoin-cli -datadir=/var/db/bitcoin getblockchaininfo
```

Wait until sync is complete, once blocks=headers you're good to go. Let this run overnight.

### Shell Alias
Typing in `-datadir=/var/db/bitcoin` everytime we want to run a bitcoin-cli command is cumbersome, lets make a shell alias.
```
# alias bitcoin-cli bitcoin-cli -datadir=/var/db/bitcoin
# bitcoin-cli getnetworkinfo
```
Success! Now lets make this survive a reboot, `nano ~/.cshrc` and add the following line:
```
alias bitcoin-cli bitcoin-cli -datadir=/var/db/bitcoin
```
Save (Ctrl+O, ENTER) and exit (Ctrl +X)

### How to upgrade bitcoind: 
```
# service bitcoind stop
# pkg update && pkg upgrade bitcoin-daemon bitcoin-utils
# nano /usr/local/etc/rc.d/bitcoind
```
Remove the `#` comment before `pidfile="/var/run/${name}.pid"`
Delete the entire line `pidfile="${bitcoind_data_dir}/bitcoind.pid"`

Save (Ctrl+O, ENTER) and exit (Ctrl+X)

Start bitcoin:
```
# service bitcoind start
```

Next: [ [Install Tor](freenas_3_tor.md) ]

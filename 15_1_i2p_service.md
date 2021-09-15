# 15.1: Bitcoin Core as an I2P (Invisible Internet Project) service

> :information_source: **NOTE:** This section has been recently added to the course and is an early draft that may still be awaiting review. Caveat reader.

Rather than using the proxy-based Tor service to ensure the privacy of your Bitcoin communications, you may instead wish to use i2p, which is designed to act as a private network within the internet, rather than simply offering private acecss to internet services.

## Understand the Differences

Tor and i2p both offer private access to online services, but with different routing and databases, and with different architectures for relays. Since hidden services (such as Bitcoin access) are core to the design of i2p, they have also been better optimized:

|  | Tor | i2p |
| :---         |     :---:      |          ---: |
| Routing   | [Onion](https://www.onion-router.net/)     | [Garlic](https://geti2p.net/en/docs/how/garlic-routing)    |
| Network Database     | Trusted [Directory Servers](https://blog.torproject.org/possible-upcoming-attempts-disable-tor-network)       | [Distributed network database](https://geti2p.net/en/docs/how/network-database)      |
| Relay     | **Two-way** encrypted connections between each Relay       | **One-way** connections between every server in its tunnels      |
| Hidden services     | Slow       | Fast      |

A more detailed comparison may be found at [geti2p.net](https://geti2p.net/en/comparison/tor).

### Understand Tradeoffs for Limiting Outgoing Connections

There are [tradeoffs](https://bitcoin.stackexchange.com/questions/107060/tor-and-i2p-tradeoffs-in-bitcoin-core) if you choose to support only i2p, only Tor, or both. These configurations, which limit outgoing clearnet connections, are made in Bitcoin Core using the `onlynet` argument in your `bitcoin.conf`.

* `onlynet=onion`, which limits outgoing connections to Tor, can expose a node to Sybil attacks and can create network partitioning, because of limited connections between Tornet and the clearnet.
* `onlynet=onion` and `onlynet=i2p` in conjunction, which runs Onion service with i2p service is experimental for now.

## Install i2p

To install i2p, you should make sure your ports are correctly set up and then you can continue with your setup process.

### Prepare Ports

To use i2p, you will need to open the following ports, which are required by i2p:

1. **Outbound (Internet facing):** a random port between 9000 and 31000 is selected. It is best if all these ports are open for outbound connections, which doesn't affect your security. 
- You can check firewall status using `sudo ufw status verbose`, which shouldn't deny outgoing connections by default.
2. Inbound (Internet facing): optional. A variety of inbound ports are listed in the [i2p docs](https://geti2p.net/en/faq#ports).
- For maximum privacy, it is preferable to disable accepting incoming connections.

### Run i2p

The following will run Bitcoin Core i2p services:

1. Install `i2pd` on Ubuntu:

   ```
   sudo add-apt-repository ppa:purplei2p/i2pd
   sudo apt-get update
   sudo apt-get install i2pd
   ```

   For installing on other OSes, see [these docs](https://i2pd.readthedocs.io/en/latest/user-guide/install/)

2. [Run](https://i2pd.readthedocs.io/en/latest/user-guide/run/) the i2p service:

   ```
   $ sudo systemctl start i2pd.service
   ```

3. Check that i2p is running. You should see it on port 7656:

   ```
   $ ss -nlt

   State   Recv-Q   Send-Q     Local Address:Port      Peer Address:Port  Process

   LISTEN  0        4096           127.0.0.1:7656           0.0.0.0:*
   ```

4. Add the following lines in `bitcoin.conf`: 

   ```
   i2psam=127.0.0.1:7656
   debug=i2p
   ```
   The logging option, `debug=i2p`, is used to record additional information in the debug log about your i2p configuration and connections. The default location for this debugging file on Linux is: `~/.bitcoin/bitcoin.conf`:

5. Restart `bitcoind`

   ```
   $ bitcoind
   ```

6. Check `debug.log` to see if i2p was setup correctly, or if any errors appeared in the logs.  
   ```
   2021-06-15T20:36:16Z i2paccept thread start
   2021-06-15T20:36:16Z I2P: Creating SAM session with 127.0.0.1:7656

   2021-06-15T20:36:56Z I2P: SAM session created: session id=3e0f35228b, my address=bmwyyuzyqdc5dcx27s4baltbu6zw7rbqfl2nmclt45i7ng3ul4pa.b32.i2p:18333
   2021-06-15T20:36:56Z AddLocal(bmwyyuzyqdc5dcx27s4baltbu6zw7rbqfl2nmclt45i7ng3ul4pa.b32.i2p:18333,4)
   ```
   
   The i2p address is mentioned in the logs, ending with _b32.i2p_. For example `bmwyyuzyqdc5dcx27s4baltbu6zw7rbqfl2nmclt45i7ng3ul4pa.b32.i2p:18333`.

7. Confirm `i2p_private_key` was created in the Bitcoin Core data directory. The first time Bitcoin Core connects to the i2p router, its i2p address (and corresponding private key) will be automatically generated and saved in a file named *i2p_private_key*:
   ```
   ~/.bitcoin/testnet3$ ls

   anchors.dat  chainstate         i2p_private_key  settings.json
   banlist.dat  debug.log          mempool.dat      wallets
   blocks       fee_estimates.dat  peers.dat
   ```

8. Check that `bitcoin-cli -netinfo` or `bitcoin-cli getnetworkinfo`returns the i2p address:

   ```
   Local addresses
   bmwyyuzyqdc5dcx27s4baltbu6zw7rbqfl2nmclt45i7ng3ul4pa.b32.i2p     port  18333    score      4
   ```
  
You now have your Bitcoin server accessible through the i2p network at your new local address.

## Summary: Bitcoin Core as an I2P (Invisible Internet Project) service

It is always good to have alternatives for privacy and not depend solely on Tor for running Bitcoin Core as a hidden service. Since i2p was recently added in Bitcoin Core, not many people use it. Experiment with it and report bugs if you find any issues.

> :information_source: **NOTE:** For the official i2prouter implementation in Java, visit the [i2p download page](https://geti2p.net/en/download) and follow the instructions for your Operating System. Once installed, open a terminal window and type `i2prouter start`. Then visit `127.0.0.1:7657` in your browser to enable SAM. To do so, select: "Configure Homepage", then "Clients", and finally select the "Play Button" next to SAM application Bridge. On the left side of the page, there should be a green light next to "Shared Clients".

Move on to "Programming with RPC" with [Chapter Sixteen: Talking to Bitcoind with C](16_0_Talking_to_Bitcoind.md).

Or, if you're not a programmer, you can skip to [Chapter Nineteen: Understanding Your Lightning Seutp](https://github.com/BlockchainCommons/Learning-Bitcoin-from-the-Command-Line/blob/master/19_0_Understanding_Your_Lightning_Setup.md) to continue your command-line education with the Lightning Network.
![documentation](http://getprowl.com/assets/images/documentation.png)
<h1 align="center">Managing Clusters for Prowl</h1>

With Prowl, in some cases we use Scaleway. Scaleway uses custom Kernel versions which makes the installation process a little more complex. Fortunately, they provide a [shell script](https://github.com/scaleway/kernel-tools#how-to-build-a-custom-kernel-module) to download the required headers without much hassle.

Alternatively, you can build a DKMS-based kernel module, for instance by running:

```
apt-get install zfsutils-linux
```

## WireGuard

Once WireGuard has been compiled, it's time to create the configuration files. Each host should connect to its peers to create a secure network overlay via a tunnel interface called wg0. Let's assume the setup consists of three hosts and each one will get a new VPN IP address in the 10.0.1.1/24 range:

| Host  | Private IP address  (ethN) | VPN IP address (wg0) |
| ----- | -------------------------- | -------------------- |
| prowlbox1 | 10.8.23.93                 | 10.0.1.1         |
| prowlbox2 | 10.8.23.94                 | 10.0.1.2         |
| prowlbox3 | 10.8.23.95                 | 10.0.1.3         |

In this scenario, a configuration file for prowlbox1 would look like this:

```sh
# /etc/wireguard/wg0.conf
[Interface]
Address = 10.0.1.1
PrivateKey = <PRIVATE_KEY_PROWLBOX1>
ListenPort = 51820

[Peer]
PublicKey = <PUBLIC_KEY_PROWLBOX2>
AllowedIps = 10.0.1.2/32
Endpoint = 10.8.23.94:51820

[Peer]
PublicKey = <PUBLIC_KEY_PROWLBOX3>
AllowedIps = 10.0.1.3/32
Endpoint = 10.8.23.95:51820
```

To simplify the creation of private and public keys, the following command can be used to generate and print the necessary key-pairs:

```sh
for i in 1 2 3; do
  private_key=$(wg genkey)
  public_key=$(echo $private_key | wg pubkey)
  echo "Host $i private key: $private_key"
  echo "Host $i public key:  $public_key"
done
```



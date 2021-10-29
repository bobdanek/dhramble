Extremely WIP.

1. Set up a database on a database server external to the cluster (NAS in my case)
1. Install k3s on some pis
  * `curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable servicelb --disable traefik --write-kubeconfig-mode 644 --token=<some random string>" sh -s - server --datastore-endpoint="mysql://<db username>:<db user password>@tcp(<db server hostname>:<db server port>)/<db name>"`
  * Note: You can run this exact same command on multiple pis. As long as the token and the db details are the same, they should function as multi-master.
3. Copy the kubeconfig from a node to your local machine, so you can run kubectl commands locally
  * `scp pi@<node hostname or IP>:/etc/rancher/k3s/k3s.yaml $HOME/.kube/config && sed -i 's/127.0.0.1/<node hostname or IP>/g' .kube/config`
2. Install [metallb](https://metallb.org/installation/) on the cluster
3. Install [longhorn](https://longhorn.io/docs/1.2.2/deploy/install/install-with-kubectl/) on the cluster
6. Go through the yaml files and change stuff to suit your network
  * metallb.yaml needs a range of IP addresses on your network that won't be allocated by DHCP or used by any devices not managed by metallb
  * pihole.yaml has a hardcoded IP address, replace that with an IP within the metallb IP range
  * same with deployment.yaml
  * probably some other things
4. `kubectl apply -f k8s/`
  * you may need to run this a few times if stuff doesn't come up before other stuff needs to come up first
5. `kubectl apply -f pihole-secondary`
6. Wait for stuff to come up. I like to watch [🐺 k9s 🐺 ](https://k9scli.io/)
7. Get the IP for the primary pihole:
  * kubectl describe service -n pihole pihole-dns-udp | grep Ingress
7. Get the IP for the secondary pihole:
  * kubectl describe service -n pihole-2 pihole-dns-service | grep Ingress

Enjoy! Make config changes for the secondary pihole in 01-private.yaml and 02-configmaps.yaml

TODO: lots, like https


Sources of inspiration:
* https://github.com/ivanmorenoj/k3s-pihole-wireguard
* https://gist.github.com/troyharvey/4506472732157221e04c6b15e3b3f094
* https://github.com/David-VTUK/piholev2

# Purpose

Upgrading the K3s image from `rancher/k3s:v1.31.2-k3s1` to `rancher/k3s:v1.31.3-k3s1` to causes the following error inside the [gluetun](https://github.com/qdm12/gluetun) container:

``` text
2024-12-08T12:51:29Z INFO [routing] default route found: interface eth0, gateway 10.42.0.1, assigned IP 10.42.0.15 and family v4
2024-12-08T12:51:29Z INFO [routing] adding route for 0.0.0.0/0
2024-12-08T12:51:29Z INFO [firewall] setting allowed subnets...
2024-12-08T12:51:29Z INFO [routing] default route found: interface eth0, gateway 10.42.0.1, assigned IP 10.42.0.15 and family v4
2024-12-08T12:51:29Z INFO TUN device is not available: open /dev/net/tun: no such file or directory; creating it...
2024-12-08T12:51:29Z INFO [routing] routing cleanup...
2024-12-08T12:51:29Z INFO [routing] default route found: interface eth0, gateway 10.42.0.1, assigned IP 10.42.0.15 and family v4
2024-12-08T12:51:29Z INFO [routing] deleting route for 0.0.0.0/0
2024-12-08T12:51:29Z ERROR creating tun device: unix opening TUN device file: operation not permitted (did you specify --device /dev/net/tun to your container command?)
2024-12-08T12:51:29Z INFO Shutdown successful
```
This repo exsts to reproduce the error.

# Reproduce steps

1. Populate appropriate `mullvad` username as the value of the `OPENVPN_USER` environment variable in `templates/gluetun.yaml`
1. Run `k3d cluster create --config k3d-config.working.yaml`
1. Run `helm upgrade --install k3s-1.31.3-test .`
1. Wait for the `glueton-0` pod to `Running`
1. Run `kubectl logs gluetun-0` and note successfully running
1. Run `k3d cluster create --config k3d-config.broken.yaml`
1. Run `helm upgrade --install k3s-1.31.3-test .`
1. Wait for the `glueton-0` pod to `Error`
1. Run `kubectl logs gluetun-0` and note permission denied error for `/dev/net/tun`

# Cleanup steps

1. `k3d cluster delete --config k3d-config.working.yaml`
1. `k3d cluster delete --config k3d-config.broken.yaml`
# Download a precompiled tailscale binary build 

We recommend to use version 1.68.2 since newer builds have issues on RAK gateways.

```
cd /mnt/mmcblk0p1
wget https://pkgs.tailscale.com/stable/tailscale_1.68.2_mipsle.tgz
tar zxvf tailscale_1.68.2_mipsle.tgz
mv tailscale_1.68.2_mipsle tailscale
```

# Add a /etc/init.d/tailscale

Open the VIM text editor and copy the lines below to `/etc/init.d/tailscale`

```
#!/bin/sh /etc/rc.common

# Copyright 2020 Google LLC.
# Copyright (C) 2021 CZ.NIC z.s.p.o. (https://www.nic.cz/)
# SPDX-License-Identifier: Apache-2.0

USE_PROCD=1
START=99

start_service() {
  local state_file
  local port
  local std_err std_out

  config_load tailscale
  config_get_bool std_out "settings" log_stdout 1
  config_get_bool std_err "settings" log_stderr 1
  config_get port "settings" port 41641
  config_get state_file "settings" state_file /mnt/mmcblk0p1/tailscale/tailscaled.state

  /mnt/mmcblk0p1/tailscale/tailscaled --cleanup

  procd_open_instance
  procd_set_param command /mnt/mmcblk0p1/tailscale/tailscaled

  # Set the port to listen on for incoming VPN packets.
  # Remote nodes will automatically be informed about the new port number,
  # but you might want to configure this in order to set external firewall
  # settings.
  procd_append_param command --port "$port"
  procd_append_param command --state "$state_file"

  procd_set_param respawn
  procd_set_param stdout "$std_out"
  procd_set_param stderr "$std_err"

  procd_close_instance
}

stop_service() {
  /mnt/mmcblk0p1/tailscale/tailscaled --cleanup
}
```

# Make the init.d file executable

`chmod +x /etc/init.d/tailscale`

# Enable the service 

`/etc/init.d/tailscale enable`
`/etc/init.d/tailscale start`


# Join the network
If using Tailscale's coordination server, you don't need to specify a `{HEADSCALE_SERVER_IP}`
`cd /mnt/mmcblk0p1`
`./tailscale up --hostname={RAK_GW_HOSTNAME} --login-server={HEADSCALE_SERVER_IP} --advertise-tags=tag:lorawan --authkey={TAILSCALE_AUTHKEY}`

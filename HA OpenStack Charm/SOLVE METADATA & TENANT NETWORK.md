# SOLVE METADATA & TENANT NETWORK

==KASUS==: Cloud-init tidak suskses ketika tidak di centang (--config-drive true/disk lokal dalam vm)

### OPENSTACK CHARM

```bash
juju remove-relation neutron-gateway nova-cloud-controller
juju integrate neutron-gateway nova-cloud-controller
```

cek ulang secret setelah relation dibuat ulang

```bash
juju exec --application neutron-gateway 'sudo egrep -i "metadata_proxy_shared_secret|nova_metadata_host" /etc/neutron/metadata_agent.ini'
juju exec --application nova-cloud-controller 'sudo egrep -i "service_metadata_proxy|metadata_proxy_shared_secret" /etc/nova/nova.conf'
```

- semua unit `neutron-gateway` harus punya secret yang sama
- nilainya harus sama persis dengan semua unit `nova-cloud-controller`

:::warning
Jika secret data belum sama harus di relasikan ulang(hapus relasi lama dan buat relasi baru neutron).
:::

Restart service jika decretkey dah sama

```bash
juju exec --application neutron-gateway 'sudo systemctl restart neutron-metadata-agent neutron-dhcp-agent'
juju exec --application nova-cloud-controller 'sudo systemctl restart apache2'
```

Karena pada unit API Nova dilayani Apache, bukan `nova-api.service`.

---

- error relation hook
- secret generation yang tidak konsisten
- kegagalan render config setelah relation-changed

```bash
juju debug-log --replay --include neutron-gateway,nova-cloud-controller | egrep -i "metadata|secret|quantum-network-service|relation"
juju exec --application neutron-gateway 'sudo journalctl -u neutron-metadata-agent -n 200 --no-pager'
```

---

---

# ERROR TENANT NETWORK

```bash
juju integrate neutron-gateway:neutron-plugin-api neutron-api:neutron-plugin-api
```

```bash
juju ssh 0 sudo cat /etc/neutron/plugins/ml2/openvswitch_agent.ini
juju ssh 1 sudo cat /etc/neutron/plugins/ml2/openvswitch_agent.ini
juju ssh 2 sudo cat /etc/neutron/plugins/ml2/openvswitch_agent.ini
```

:::warning
Pastikan semua controller sudah

tunnel_types = vxlan  
l2_population = True
:::

```bash
# Verifikasi nilai saat ini
juju config neutron-api overlay-network-type
juju config neutron-api default-tenant-network-type
juju config neutron-api l2-population
juju config neutron-api vni-ranges
```

Jika masih gre ubah ke vxlan

```bash
juju config neutron-api overlay-network-type=vxlan
juju config neutron-api default-tenant-network-type=vxlan
juju config neutron-api l2-population=true

# opsional/sesuai desain
juju config neutron-api vni-ranges=1:5000
```

:::info
vni-ranges adalah rentang ID VXLAN (VNI, Virtual Network Identifier) yang boleh dipakai Neutron saat membuat jaringan tenant/self-service bertipe VXLAN. misal nilai 1:5000 berarti Neutron dapat mengalokasikan VNI 1 sampai 5000 untuk network VXLAN baru.
:::

Verifikasi Bridge tunnel (Bridge br-tun)

```bash
juju ssh 0 sudo ovs-vsctl show
juju ssh 1 sudo ovs-vsctl show
juju ssh 2 sudo ovs-vsctl show
```

```bash
openstack network agent list --long
```

### Target konfigurasi OVS agent

```bash
[ovs]
enable_tunneling = True
local_ip = 172.16.2.X
bridge_mappings = physnet1:br-provider

[agent]
tunnel_types = vxlan
l2_population = True
```

---

**Advance Step | →**
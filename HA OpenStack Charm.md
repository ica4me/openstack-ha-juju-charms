# HA OpenStack Charm

by @wahyudi

### Status: SELESAI👍

:::info
Caracal 2024.1/stable (Ubuntu 22.04)
:::

[MAAS Configure](HA%20OpenStack%20Charm/MAAS%20Configure.md)

[Topologi](HA%20OpenStack%20Charm/Topologi.md)

[Bootstrap Juju Controller](HA%20OpenStack%20Charm/Bootstrap%20Juju%20Controller.md)

[Deployment Charm](HA%20OpenStack%20Charm/Deployment%20Charm.md)

[EndPoint HTTP to HTTPS](HA%20OpenStack%20Charm/EndPoint%20HTTP%20to%20HTTPS.md)

[Client & Admin Dashboard](HA%20OpenStack%20Charm/Client%20%26%20Admin%20Dashboard.md)

[Expose Domain (Proxy)](HA%20OpenStack%20Charm/Expose%20Domain%20%28Proxy%29.md)

[Ubah Endpoint public jadi murni HTTPS](HA%20OpenStack%20Charm/Ubah%20Endpoint%20public%20jadi%20murni%20HTTPS.md)

[Operasional CLI](HA%20OpenStack%20Charm/Operasional%20CLI.md)

[SOLVE METADATA & TENANT NETWORK](HA%20OpenStack%20Charm/SOLVE%20METADATA%20%26%20TENANT%20NETWORK.md)

### Network

| No  | Space | Kegunaan utama |
| --- | --- | --- |
| 1   | `management` | Jaringan manajemen/infrastruktur untuk MAAS, Juju controller/agent, SSH admin, provisioning, dan komunikasi operasional dasar antar node. |
| 2   | `external-openstack` | Jaringan endpoint API eksternal/public untuk service OpenStack seperti Horizon, Keystone, Nova, Neutron, dan Glance agar dapat diakses dari luar. |
| 3   | `internal-openstack` | Jaringan endpoint API internal antar service OpenStack untuk komunikasi service-to-service secara terpisah dari akses public. |
| 4   | `storage-network` | Jaringan storage internal untuk trafik Ceph. |
| 5   | `provider-network` | Jaringan provider/physnet untuk floating IP, akses external VM, dan konektivitas jaringan tenant ke luar OpenStack. |

---

### MAIN COMPONENT

| No  | Komponen | Kegunaan utama di OpenStack |
| --- | --- | --- |
| 1   | RabbitMQ | Message broker / antrean pesan untuk komunikasi RPC dan notifikasi antar layanan OpenStack. |
| 2   | Database | Menyimpan data persisten dan metadata layanan, seperti user, instance, image, network, dan konfigurasi service. |
| 3   | Keystone | Layanan identitas untuk autentikasi, otorisasi, token, dan katalog endpoint service OpenStack. |
| 4   | Neutron | Mengelola jaringan cloud: network, subnet, router, port, floating IP, dan security group. |
| 5   | Glance | Menyimpan, mendaftarkan, dan menyediakan image VM yang dipakai saat membuat instance. |
| 6   | Nova | Layanan compute untuk membuat, menjalankan, menghentikan, memindahkan, dan mengelola VM/instance. |
| 7   | Memcached | Cache in-memory untuk mempercepat akses data tertentu, misalnya token, session, atau cache service. |
| 8   | Placement | Melacak inventori dan alokasi resource seperti vCPU, RAM, dan disk untuk membantu penjadwalan instance. |

### Telemetry/monitoring

| No  | Komponen | Kegunaan utama di OpenStack |
| --- | --- | --- |
| 1   | Gnocchi | Menyimpan metrics time-series (data metrik historis) seperti penggunaan resource untuk monitoring, analisis kapasitas, dan dasar evaluasi alarm. |
| 2   | Aodh | Menyediakan alarm service, yaitu memicu notifikasi atau aksi otomatis ketika metric atau event memenuhi ambang batas tertentu. |
| 3   | Ceilometer | Mengumpulkan telemetry/metering data dari layanan OpenStack, seperti penggunaan CPU, network, dan resource lain, untuk monitoring, billing, dan integrasi ke sistem metrics/alarm. |

### EnterPrise Feature (HA)

| No  | Komponen | Kegunaan utama di OpenStack |
| --- | --- | --- |
| 1   | Horizon | Dashboard web OpenStack untuk mengelola resource cloud seperti instance, network, image, project, dan user melalui antarmuka browser. |
| 2   | Masakari | Layanan high availability untuk instance/VM yang membantu mendeteksi kegagalan host atau service compute dan melakukan pemulihan otomatis workload. |
| 3   | Barbican | Layanan Secrets management untuk menyimpan dan mengelola key, password, sertifikat, dan data sensitif lain secara terpusat. |
| 4   | Vault | Layanan pengelolaan secrets dan PKI/TLS untuk distribusi sertifikat, enkripsi, dan penyimpanan kredensial secara aman. |
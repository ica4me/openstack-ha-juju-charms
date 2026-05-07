# HA OpenStack Charm

![OpenStack](https://img.shields.io/badge/OpenStack-Caracal%202024.1-red)
![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04%20LTS-orange)
![Deployment](https://img.shields.io/badge/Deployment-MAAS%20%2B%20Juju%20%2B%20Charms-blue)

Dokumentasi implementasi **High Availability OpenStack** menggunakan **MAAS**, **Juju**, dan **OpenStack Charms** pada **OpenStack Caracal 2024.1/stable** berbasis **Ubuntu 22.04 LTS**.

Repo ini berfungsi sebagai catatan teknis deployment dari tahap desain topologi, konfigurasi MAAS, bootstrap Juju controller, deployment charm, konfigurasi endpoint HTTPS, dashboard, proxy domain, operasional CLI, sampai troubleshooting metadata dan tenant network.

---

## Gambaran arsitektur

Deployment ini menggunakan pemisahan jaringan untuk manajemen, API public, API internal, storage Ceph, dan provider/physnet. Control plane berjalan secara HA pada node controller, sedangkan compute node juga membawa peran storage Ceph OSD secara hyperconverged.

### Topologi visual

![Topologi HA OpenStack Charm](HA%20OpenStack%20Charm/files/019da505-f6d7-7199-bf04-f8f9f3977f1d/topologi.drawio.png)

Detail topologi, subnet, IP VIP, hostname, spesifikasi node, dan catatan boot order tersedia di:

➡️ [Topologi](HA%20OpenStack%20Charm/Topologi.md)

---

## Ringkasan deployment

| Item              | Nilai                                                                               |
| ----------------- | ----------------------------------------------------------------------------------- |
| OpenStack release | Caracal 2024.1/stable                                                               |
| Base OS           | Ubuntu 22.04 LTS                                                                    |
| Provisioning      | MAAS                                                                                |
| Orchestrator      | Juju                                                                                |
| Deployment method | OpenStack Charms                                                                    |
| HA focus          | Control plane, database, message broker, endpoint, dashboard, dan workload recovery |
| Storage           | Ceph melalui storage-network                                                        |
| Public access     | Endpoint HTTPS dan proxy domain                                                     |
| Operasional       | OpenStack CLI, Horizon dashboard, troubleshooting metadata/tenant network           |

---

## Full dokumentasi

| Urutan | Dokumen                                                                                                            | Fungsi                                                                              |
| ------ | ------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------- |
| 1      | [Topologi](HA%20OpenStack%20Charm/Topologi.md)                                                                     | Desain jaringan, node, subnet, VIP, hostname, dan spesifikasi lab/deployment.       |
| 2      | [MAAS Configure](HA%20OpenStack%20Charm/MAAS%20Configure.md)                                                       | Konfigurasi MAAS untuk provisioning node fisik/VM, jaringan, PXE, dan image OS.     |
| 3      | [Bootstrap Juju Controller](HA%20OpenStack%20Charm/Bootstrap%20Juju%20Controller.md)                               | Inisialisasi Juju controller dan integrasi Juju dengan MAAS.                        |
| 4      | [Full Step Deployment Charm](HA%20OpenStack%20Charm/Deployment%20Charm.md)                                         | Deployment service OpenStack menggunakan Juju charms.                               |
| 5      | [EndPoint HTTP to HTTPS](HA%20OpenStack%20Charm/EndPoint%20HTTP%20to%20HTTPS.md)                                   | Migrasi endpoint OpenStack dari HTTP ke HTTPS.                                      |
| 6      | [Client & Admin Dashboard](HA%20OpenStack%20Charm/Client%20%26%20Admin%20Dashboard.md)                             | Akses client, Horizon dashboard, dan kebutuhan admin.                               |
| 7      | [Expose Domain (Proxy)](HA%20OpenStack%20Charm/Expose%20Domain%20%28Proxy%29.md)                                   | Ekspos endpoint/dashboard melalui domain dan reverse proxy.                         |
| 8      | [Ubah Endpoint public jadi murni HTTPS](HA%20OpenStack%20Charm/Ubah%20Endpoint%20public%20jadi%20murni%20HTTPS.md) | Finalisasi public endpoint agar sepenuhnya menggunakan HTTPS.                       |
| 9      | [Vault CA ➡️ Let's Encrypt](HA%20OpenStack%20Charm/Vault%20CA%20%E2%9E%A1%EF%B8%8F%20Let%27s%20Encrypt.md)         | Integrasi Vault/CA dan sertifikat Let's Encrypt untuk TLS.                          |
| 10     | [Operasional CLI](HA%20OpenStack%20Charm/Operasional%20CLI.md)                                                     | Operasi harian OpenStack menggunakan CLI.                                           |
| 11     | [SOLVE METADATA & TENANT NETWORK](HA%20OpenStack%20Charm/SOLVE%20METADATA%20%26%20TENANT%20NETWORK.md)             | Troubleshooting metadata service, tenant network, floating IP, dan konektivitas VM. |

> [!IMPORTANT]
> Full Documentation Step-by-step Deploy: (HA%20OpenStack%20Charm/Deployment%20Charm.md)

---

## Network spaces

| No  | Space                | Kegunaan utama                                                                                                                                    |
| --- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | `management`         | Jaringan manajemen/infrastruktur untuk MAAS, Juju controller/agent, SSH admin, provisioning, dan komunikasi operasional dasar antar node.         |
| 2   | `external-openstack` | Jaringan endpoint API eksternal/public untuk service OpenStack seperti Horizon, Keystone, Nova, Neutron, dan Glance agar dapat diakses dari luar. |
| 3   | `internal-openstack` | Jaringan endpoint API internal antar service OpenStack untuk komunikasi service-to-service secara terpisah dari akses public.                     |
| 4   | `storage-network`    | Jaringan storage internal untuk trafik Ceph.                                                                                                      |
| 5   | `provider-network`   | Jaringan provider/physnet untuk floating IP, akses external VM, dan konektivitas jaringan tenant ke luar OpenStack.                               |

### Alokasi NIC dan subnet

| No  | NIC fisik | Alokasi                   | Subnet          |
| --- | --------- | ------------------------- | --------------- |
| 1   | NIC-1     | Management khusus         | `172.16.1.0/24` |
| 2   | NIC-2     | External OpenStack khusus | `172.16.4.0/24` |
| 3   | NIC-3     | Internal OpenStack khusus | `172.16.2.0/24` |
| 4   | NIC-4     | Storage khusus            | `172.16.5.0/24` |
| 5   | NIC-5     | Provider / physnet khusus | `172.16.3.0/24` |

---

## Node dan Rekoemndasi Spek(Minimun)

| Node     | vCPU |   RAM | Disk OS | Disk Ceph | Peran utama                            |
| -------- | ---: | ----: | ------: | --------: | -------------------------------------- |
| MAAS     |    4 | 12 GB |  100 GB |         - | PXE Boot, DHCP, DNS, OS Provisioning   |
| Deployer |    8 | 16 GB |  100 GB |         - | Juju Controller dan CLI Command Center |
| Ctrl-01  |    8 | 16 GB |  100 GB |         - | API OpenStack, MySQL/Galera, Vault     |
| Ctrl-02  |    8 | 16 GB |  100 GB |         - | High Availability quorum               |
| Ctrl-03  |    8 | 16 GB |  100 GB |         - | High Availability quorum               |
| Comp-01  |   16 | 24 GB |  100 GB | 2× 200 GB | Hyperconverged Compute dan Ceph OSD    |
| Comp-02  |   16 | 24 GB |  100 GB | 2× 200 GB | Hyperconverged Compute dan Ceph OSD    |

> [!IMPORTANT]
> Semua node compute harus mendukung KVM. Validasi dengan `kvm-ok` dan pastikan `/dev/kvm` tersedia sebelum menjalankan workload VM.

---

## Komponen utama OpenStack

| No  | Komponen  | Kegunaan utama di OpenStack                                                                                     |
| --- | --------- | --------------------------------------------------------------------------------------------------------------- |
| 1   | RabbitMQ  | Message broker / antrean pesan untuk komunikasi RPC dan notifikasi antar layanan OpenStack.                     |
| 2   | Database  | Menyimpan data persisten dan metadata layanan, seperti user, instance, image, network, dan konfigurasi service. |
| 3   | Keystone  | Layanan identitas untuk autentikasi, otorisasi, token, dan katalog endpoint service OpenStack.                  |
| 4   | Neutron   | Mengelola jaringan cloud: network, subnet, router, port, floating IP, dan security group.                       |
| 5   | Glance    | Menyimpan, mendaftarkan, dan menyediakan image VM yang dipakai saat membuat instance.                           |
| 6   | Nova      | Layanan compute untuk membuat, menjalankan, menghentikan, memindahkan, dan mengelola VM/instance.               |
| 7   | Memcached | Cache in-memory untuk mempercepat akses data tertentu, misalnya token, session, atau cache service.             |
| 8   | Placement | Melacak inventori dan alokasi resource seperti vCPU, RAM, dan disk untuk membantu penjadwalan instance.         |

---

## Telemetry dan monitoring

| No  | Komponen   | Kegunaan utama di OpenStack                                                                                                                                                        |
| --- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Gnocchi    | Menyimpan metrics time-series seperti penggunaan resource untuk monitoring, analisis kapasitas, dan dasar evaluasi alarm.                                                          |
| 2   | Aodh       | Menyediakan alarm service untuk memicu notifikasi atau aksi otomatis ketika metric atau event memenuhi ambang batas tertentu.                                                      |
| 3   | Ceilometer | Mengumpulkan telemetry/metering data dari layanan OpenStack, seperti penggunaan CPU, network, dan resource lain, untuk monitoring, billing, dan integrasi ke sistem metrics/alarm. |

---

## Enterprise features dan HA

| No  | Komponen | Kegunaan utama di OpenStack                                                                                                                         |
| --- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Horizon  | Dashboard web OpenStack untuk mengelola resource cloud seperti instance, network, image, project, dan user melalui antarmuka browser.               |
| 2   | Masakari | Layanan high availability untuk instance/VM yang membantu mendeteksi kegagalan host atau service compute dan melakukan pemulihan otomatis workload. |
| 3   | Barbican | Layanan secrets management untuk menyimpan dan mengelola key, password, sertifikat, dan data sensitif lain secara terpusat.                         |
| 4   | Vault    | Layanan pengelolaan secrets dan PKI/TLS untuk distribusi sertifikat, enkripsi, dan penyimpanan kredensial secara aman.                              |

---

## Checklist cepat

Gunakan urutan berikut saat mengikuti dokumentasi:

- [ ] Pahami desain jaringan dan IP mapping di [Topologi](HA%20OpenStack%20Charm/Topologi.md).
- [ ] Siapkan MAAS, image Ubuntu 22.04, subnet, VLAN/space, dan PXE boot.
- [ ] Bootstrap Juju controller dan buat model OpenStack.
- [ ] Deploy service OpenStack menggunakan charm sesuai bundle/command yang disiapkan.
- [ ] Validasi status Juju sampai semua unit `active/idle` atau sesuai kondisi yang diharapkan.
- [ ] Konfigurasi endpoint API, Horizon, dan domain public.
- [ ] Migrasikan endpoint public ke HTTPS.
- [ ] Validasi akses dashboard, OpenStack CLI, image, network, subnet, router, floating IP, dan instance.
- [ ] Uji metadata service dan konektivitas tenant network.
- [ ] Dokumentasikan perubahan endpoint, DNS, sertifikat, dan kredensial operasional secara aman.

---

## Struktur repo

```text
.
├── README.md
├── HA OpenStack Charm.md
└── HA OpenStack Charm/
    ├── MAAS Configure.md
    ├── Topologi.md
    ├── Bootstrap Juju Controller.md
    ├── Deployment Charm.md
    ├── EndPoint HTTP to HTTPS.md
    ├── Client & Admin Dashboard.md
    ├── Expose Domain (Proxy).md
    ├── Ubah Endpoint public jadi murni HTTPS.md
    ├── Vault CA ➡️ Let's Encrypt.md
    ├── Operasional CLI.md
    ├── SOLVE METADATA & TENANT NETWORK.md
    └── files/
```

---

## Dokumentasi resmi

Referensi berikut berguna untuk validasi konsep, parameter, kompatibilitas, dan praktik operasional:

| Topik                                                | Link resmi                                                                                        |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| OpenStack 2024.1 Documentation                       | https://docs.openstack.org/2024.1/                                                                |
| OpenStack Caracal 2024.1 Release                     | https://www.openstack.org/software/openstack-caracal/                                             |
| OpenStack packages for Ubuntu / Ubuntu Cloud Archive | https://docs.openstack.org/install-guide/environment-packages-ubuntu.html                         |
| OpenStack Charm Guide                                | https://docs.openstack.org/charm-guide/latest/                                                    |
| OpenStack Charms Deployment Guide                    | https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/                    |
| OpenStack Charms 2024.1 Caracal release note         | https://discourse.charmhub.io/t/openstack-charms-2024-1-caracal-release-is-available-now/17042    |
| MAAS Documentation                                   | https://canonical.com/maas/docs                                                                   |
| Juju Documentation                                   | https://documentation.ubuntu.com/juju/                                                            |
| Juju bootstrap command                               | https://documentation.ubuntu.com/juju/3.6/reference/juju-cli/list-of-juju-cli-commands/bootstrap/ |
| Charmhub                                             | https://charmhub.io/                                                                              |
| HashiCorp Vault Documentation                        | https://developer.hashicorp.com/vault/docs                                                        |
| Let's Encrypt Documentation                          | https://letsencrypt.org/docs/                                                                     |

---

## Author

Created and maintained by **@wahyudi**.

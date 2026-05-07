# MAAS Configure

API Proxmox Info for MAAS

```bash
TOKEN_SECRET:
Token_id: yezato@pve!yudi-maas
API token secret: f51c6129-6a93-4b9f-a316-ec04ff2a5718
```

### NETWORK INTERFACE

| No  | NIC fisik | Alokasi | Alamat subnet |
| --- | --- | --- | --- |
| 1   | NIC-1 | Management khusus | `172.16.1.0/24` |
| 2   | NIC-2 | External OpenStack khusus | `172.16.4.0/24` |
| 3   | NIC-3 | Internal OpenStack khusus | `172.16.2.0/24` |
| 4   | NIC-4 | Storage khusus | `172.16.5.0/24` |
| 5   | NIC-5 | Provider / physnet khusus | `172.16.3.0/24` |

## Persiapan Infrastruktur MAAS (Metal As A Service)

Target → Vm MAAS

### Instalasi MAAS & Database Lokal

```bash
# 1. Update sistem dasar
sudo apt update && sudo apt upgrade -y

# 2. Instal MAAS dan database testing/lokal
sudo snap install maas
sudo snap install maas-test-db

# 3. Inisialisasi MAAS (sebagai Region & Rack controller)
sudo maas init region+rack --database-uri maas-test-db:///

# 4. Buat akun administrator untuk login ke Web UI
sudo maas createadmin
```

![](files/019da505-f0a2-703a-b3fd-4250daedf7c5/image.png)

### Konfigurasi MAAS via Web Dashboard

```bash
http://172.16.1.2:5240/MAAS
```

![](files/019da505-f0a3-71b6-afc3-87720302eceb/image.png)![](files/019da505-f0a4-73e7-8931-e6a037c7ec1c/image.png)

### Mengaktifkan DHCP & Konfigurasi Network

:::info
MAAS harus menjadi prioritas di jaringan `net0` agar bisa melakukan _PXE Boot_ ke VM lain.
:::

![](files/019da505-f0a4-73e7-8931-e8054eebf0e0/image.png)![](files/019da505-f0a6-768a-9a0b-64b1cea13ac2/image.png)![](files/019da505-f0a6-768a-9a0b-6a97716a6c8e/image.png)![](files/019da505-f0a7-70de-9135-243e88b049ef/image.png)

### SPACE (ALL SUBNET)

![](files/019da505-f0a8-75ba-a728-c16b8f7746b0/image.png)

### MAAS Enlisting Node

:::info
- `http://172.16.1.2:5240/MAAS`
:::

![](files/019da505-f0a8-75ba-a728-c52b87dae67a/image.png)

:::success
kelima mesin tersebut akan muncul secara otomatis di daftar dengan status `New`
:::

### Ganti Nama & Pemberian Tag (untuk Juju)

:::warning
Example ⬇️
:::

![](files/019da505-f0a8-75ba-a728-c885420470cc/image.png)

### Power configuration(Proxmox)

:::info
Power type: Proxmox

Proxmox host name or IP: 172.16.1.1

Proxmox username: yezato@pve

Proxmox password: \*\*\*\*\*

Proxmox API token name: yudi-maas

Proxmox API token secret: f51c6129-6a93-4b9f-a316-ec04ff2a5718

Node ID: (sesuaikan)

Verify SSL: no
:::

![](files/019da505-f0a9-740e-a5f3-e6295648518c/image.png)

### Proxy MAAS

![](files/019da505-f0ab-77e1-8488-dc3e94c92371/image.png)

### Repo MAAS

```bash
http://mirrors.ubuntu.com/ID.txt
```

![](files/019da505-f0ab-77e1-8488-e221fa2491a4/image.png)

```bash
sudo snap restart maas
```

### Proses _Commissioning_ (Pengujian _Hardware_)

:::info
Menguji _hardware_, memetakan _disk_, dan menyiapkan _node_ agar berstatus `Ready` (siap diinstal OS oleh Juju).
:::

![](files/019da505-f0ac-7117-973e-9bf1c1948908/image.png)![](files/019da505-f0ac-7117-973e-9cee3e617e8f/image.png)

:::warning
VM akan melakukan _PXE Boot_, menjalankan _script_ pengujian MAAS, dan otomatis mati sendiri jika sudah selesai.
:::

:::success
Jika sudah mati, Status kuning `Commissioning` seharusnya akan berubah menjadi hijau `Ready`.
:::

![](files/019da505-f0ad-7659-af8e-585687e84806/image.png)

### Verifikasi Storage untuk Ceph

:::warning
Karena menggunakan skema _Hyperconverged_ Ceph, wajib memastikan MAAS mendeteksi _disk_ kosong dengan benar tanpa memformatnya.
:::

:::info
jika mengklik nama `comp-01 dan comp-02` dan masuk ke tab Storage, pastikan dua _disk_ tambahan (yang 100GB) berstatus Unused.
:::

:::success
Available disks and partitions disk tersebut Unused dan tidak memiliki partisi (_raw_).
:::

![](files/019da505-f0ad-7659-af8e-5ed1ad7d51f8/image.png)

### Verifikasi Interface Network semua NOde

Misal Node comp-01 (5 interface)

![](files/019da505-f0ad-7659-af8e-620ad31d0e1a/image.png)

**Next →**
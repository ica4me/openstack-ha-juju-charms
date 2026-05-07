# EndPoint HTTP to HTTPS

:::info
Sebelum Vault diisntal Endpoint masih pakai HTTP, setelah ada Vault Endpoint harus https dengan CA vault.
:::

```bash
juju ssh glance/0 "sudo egrep -Rni '35357|http://identity-int|auth_url|www_authenticate_uri' /etc/glance"

juju ssh nova-cloud-controller/0 "sudo egrep -Rni '35357|http://identity-int|auth_url|www_authenticate_uri' /etc/nova"

juju ssh neutron-api/0 "sudo egrep -Rni '35357|http://identity-int|auth_url|www_authenticate_uri' /etc/neutron"

juju ssh cinder/0 "sudo egrep -Rni '35357|http://identity-int|auth_url|www_authenticate_uri' /etc/cinder"
```

```bash
juju remove-relation glance:identity-service keystone:identity-service --force

juju remove-relation nova-cloud-controller:identity-service keystone:identity-service --force

juju remove-relation neutron-api:identity-service keystone:identity-service --force

juju remove-relation cinder:identity-service keystone:identity-service --force

juju remove-relation placement:identity-service keystone:identity-service --force
```

Add ulang Relasi (Reload Service)

```bash
juju integrate glance:identity-service keystone:identity-service

juju integrate nova-cloud-controller:identity-service keystone:identity-service

juju integrate neutron-api:identity-service keystone:identity-service

juju integrate cinder:identity-service keystone:identity-service

juju integrate placement:identity-service keystone:identity-service
```

Tunggu semua Service active

```bash
watch -c "juju status --color | grep -vE 'active|started'"
# Aatau
watch -n 2 juju status --color
```

### Restart service yang masih pegang config lama

:::warning
Sesudah relasi di-refresh, restart service API utama
:::

```bash
juju ssh glance/0 "sudo systemctl restart glance-api"
juju ssh glance/1 "sudo systemctl restart glance-api"
juju ssh glance/2 "sudo systemctl restart glance-api"

juju ssh nova-cloud-controller/0 "sudo systemctl restart nova-api-os-compute nova-scheduler nova-conductor"
juju ssh nova-cloud-controller/1 "sudo systemctl restart nova-api-os-compute nova-scheduler nova-conductor"
juju ssh nova-cloud-controller/2 "sudo systemctl restart nova-api-os-compute nova-scheduler nova-conductor"

juju ssh neutron-api/0 "sudo systemctl restart neutron-server"
juju ssh neutron-api/1 "sudo systemctl restart neutron-server"
juju ssh neutron-api/2 "sudo systemctl restart neutron-server"

juju ssh cinder/0 "sudo systemctl restart apache2 cinder-scheduler"
juju ssh cinder/1 "sudo systemctl restart apache2 cinder-scheduler"
juju ssh cinder/2 "sudo systemctl restart apache2 cinder-scheduler"

juju ssh placement/0 "sudo systemctl restart apache2"
juju ssh placement/1 "sudo systemctl restart apache2"
juju ssh placement/2 "sudo systemctl restart apache2"
```

### Perbaiki Endpoint Akses Horizon http → https

Cek apakah Horizon masih memakai HTTP ke Keystone

```bash
juju ssh openstack-dashboard/1 "sudo egrep -Rni '35357|identity-int|OPENSTACK_KEYSTONE_URL|OPENSTACK_HOST|https://|http://' /etc/openstack-dashboard /etc/apache2"
```

Ubah `OPENSTACK_KEYSTONE_URL` dari HTTP ke HTTPS

```bash
for i in 0 1 2; do
  juju ssh openstack-dashboard/$i "sudo sed -i 's|OPENSTACK_KEYSTONE_URL = \"http://%s:5000/v3\" % OPENSTACK_HOST|OPENSTACK_KEYSTONE_URL = \"https://%s:5000/v3\" % OPENSTACK_HOST|g' /etc/openstack-dashboard/local_settings.py"
done
```

Hapus endpoint HTTP lama dari `AVAILABLE_REGIONS`

```bash
for i in 0 1 2; do
  juju ssh openstack-dashboard/$i "sudo sed -i \"/http:\\/\\/identity-api\\.projx\\.my\\.id:5000\\/v3/d\" /etc/openstack-dashboard/local_settings.py"
done
```

Verifikasi hasil perubahan

```bash
for i in 0 1 2; do
  juju ssh openstack-dashboard/$i "sudo egrep -n 'AVAILABLE_REGIONS|OPENSTACK_HOST|OPENSTACK_KEYSTONE_URL|identity-api' /etc/openstack-dashboard/local_settings.py"
done
```

Restart Apache di semua unit dashboard

```bash
for i in 0 1 2; do
  juju ssh openstack-dashboard/$i "sudo systemctl restart apache2"
done
```

Uji endpoint Horizon

```bash
curl -kI https://horizon-api.projx.my.id/
curl -kI https://172.16.4.50
```

https://172.16.4.50/

https://horizon-api.projx.my.id/ (Domain by lokal etc/hosts)

**Next →**
# Role estclient

Use `estclient` role to request certificate from internal CA via the EST protocol
(RFC 7030). All operations are done by `estclient` user. EST client is run daily for each
certificate.

## Variables

Main configuration is done using the `est_certs` and `est_certs_restart_units` arrays
of dictionaries.

`est_certs` is an array of dictionaries with following values:

| Variable                  | Mandatory | Default value | Description |
| ------------------------- | --------- | ------------- | ----------- |
| name                      | yes       |               | Name of the certificate. Not used withing the certificate itself |
| site_name                 | yes       |               | Common name of the certificate subject |
| aliases                   | no        |               | List of additional domain name to be present in the certificate SAN |
| rsa                       | no        | false         | Use RSA algorithm for the private key. Default is ECDSA. |


`est_certs_restart_units` is an array of dictionaries with following values:

| Variable | Mandatory | Default value | Description |
| -------- | --------- | ------------- | ----------- |
| .name    | yes       |               | Systemd unit name (f.e. `apache2.service`) |
| .action  | no        | restart       | Type of action `systemctl` is going to execute (f.e. `restart` or `reload`) |

Other variables:

| Variable                  | Mandatory | Default value            | Description |
| ------------------------- | --------- | ------------------------ | ----------- |
| est_server_url            | yes       |                          | Full HTTPS URL of EST server endpoint |
| estclient_home_dir        | no        | /var/lib/estclient       | Home directory of estclient user |
| estclient_cert_dir        | no        | /var/lib/estclient/certs | Directory containing all certificate directories |
| estclient_renew_threshold | no        | 31                       | Days remaining until expiry since when to atte mpt the certificate renewal |


## HA

Role is adapted to the master-backup HA scenario. `estclient` is run on the master node.
Certificate files are copied to the backup node via the SSH. Services on backup node
are also restarted via the SSH.

Required variables for each host in the HA mode:

- `failover_role`: `master` or `backup`
- `failover_mirror`: Ansible inventory hostname of the other node.


# Examples

```yaml
est_server_url: 'https:/pki.someca.tld/.well-known/est/url'
est_certs:
  - name: 'testcert'
    rsa: true
    site_name: 'commonname.company.tld'
    aliases:
      - 'san2.company.tld'
      - 'san3.company.tld''

  - name: 'testcert2'
    site_name: 'testcert'

est_certs_restart_units:
  - name: 'exim4.service'
    action: 'restart'
```

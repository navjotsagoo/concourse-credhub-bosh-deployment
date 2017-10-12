#Deploying Concourse with Credhub Integration

The BOSH Director installation is based off of the bosh-deployment repository. The BOSH team keeps this repository up to date with the latest bosh and credhub advances. In order to modify the installation we modify variables and add [BOSH operations files](https://bosh.io/docs/cli-ops-files.html). This installation also requires the use of the [BOSHv2 CLI](https://bosh.io/docs/cli-v2.html)

```bash
git clone https://github.com/cloudfoundry/bosh-deployment.git
```

Run the following command to create or update a BOSH Director
Adding `uaa` and `credhub` yaml ops files during deployment of Bosh Director

```bash
bosh create-env bosh-deployment/bosh.yml \
    --state=state.json \
    --vars-store=creds.yml \
    -o bosh-deployment/aws/cpi.yml \
    -o bosh-deployment/uaa.yml \
    -o bosh-deployment/credhub.yml \
    -v director_name=boshd \
    -v internal_cidr=10.0.0.0/24 \
    -v internal_gw=10.0.0.1 \
    -v internal_ip=10.0.0.6 \
    -v access_key_id=#CHANGE_ME \
    -v secret_access_key=#CHANGE_ME \
    -v region=#CHANGE_ME \
    -v az=#CHANGE_ME \
    -v default_key_name=bosh \
    -v default_security_groups=[bosh-director] \
    --var-file private_key=#CHANGE_ME mykey.pem \
    -v subnet_id=CHANGE_ME
  ```

#Download Credhub CLI
https://github.com/cloudfoundry-incubator/credhub-cli/releases

Verify connection to Credhub
```bash
  credhub api \
--ca-cert=<(bosh int ./creds.yml --path /credhub_ca/ca) \
--ca-cert=<(bosh int ./creds.yml --path /uaa_ssl/ca) \
--server=<BOSH_DIRECTOR_IP>:8844
```

Verify login to Credhub
```bash
credhub login \
--client-name=director_credhub_client \
--client-secret=my_secret # get value of (bosh int ./creds.yml --path /uaa_clients_director_to_credhub)
This is the value that gets placed in concourse.yml file under credhub properties.
```

Test Credhub with Concourse Integration
```bash
credhub set -t value -n /concourse/main/testkey -v testvalue
See sample-pipeline _for_ configuration
```

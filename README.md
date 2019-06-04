# azure-secure-engine-for-azurechinacloud
# steps of customized build
apt install zip

wget https://dl.google.com/go/go1.12.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf ./go1.12.5.linux-amd64.tar.gz
PATH=$PATH:/usr/local/go/bin

cd /
mkdir gocode
export GOPATH="/gocode"

mkdir -p /gocode/src/github.com/hashicorp/vault-plugin-secrets-azure

/home/vmadmin/go/src/github.com/hashicorp/vault-plugin-secrets-azure

go get -u https://github.com/hashicorp/vault-plugin-secrets-azure

<update content. need to replace the dault fine from azure-sdk-for-go and go-autorest project>

# build boot strap
make bootstrap
cp /gocode/bin/gox /usr/local/go/bin
cp /gocode/bin/govendor /usr/local/go/bin

# build plugin
make 

# now we get the whole plugin file


# sample steps to build custom plugin
# https://www.hashicorp.com/blog/building-a-vault-secure-plugin

# on each vault server, copy the re-complied plugin vault-plugin-secrets-azure
# for exmaple: 
mkdir -p /hashicorp/vault/plugin
cd 
scp <user>@<source_server>:/gocode/bin/vault-plugin-secrets-azure /hashicorp/vault/plugin

# add permission. If it is a vault service running under user vault, set file owner as vault. 
chown rootL:root /hashicorp/vault/plugin/vault-plugin-secrets-azure
chmod +x vault-plugin-secrets-azure

# add to configure file
tee -a vault.hcl << EOF

plugin_directory = "/hashicorp/vault/plugin"
EOF


# start vault server：

vault server -dev -log-level="debug" -config=./vault.hcl

# in another terminal, login with root and registeter custom plugin

SHASUM=$(shasum -a 256 "/hashicorp/vault/plugin/vault-plugin-secrets-azure" | cut -d " " -f1)
 vault write sys/plugins/catalog/secret/azure \
   sha_256="$SHASUM" \
   command="vault-plugin-secrets-azure"

# ensure if it is a custom plugin now. look at attribute builtin is false. 
vault read sys/plugins/catalog/secret/azure
Key        Value
---        -----
args       []
builtin    false
command    vault-plugin-secrets-azure
name       azure
sha256     ffac49a7ceb3dcc683bd470259a33aadffb815c5051e690b410c476d04fb7d27

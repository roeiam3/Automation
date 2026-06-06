Architecture - 

Netbox as SoT (Desired State) 

Ansible as Automation

Goal - Ansible playbooks query the Netbox which is the single source of truth (The desired state of the network) and ideompotently make sure the desired state is enforced.




Step 1 - Netbox as docker



apt update 

sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

git clone -b release https://github.com/netbox-community/netbox-docker.git

# Then

cd netbox-docker

# Copy the example override file
cp docker-compose.override.yml.example docker-compose.override.yml

# Read and edit the file to your liking

docker compose pull

docker compose up

If docker compose pull fails - do this 

{
  "dns": ["8.8.8.8", "1.1.1.1"],
  "mtu": 1450,
  "max-concurrent-downloads": 1,
  "registry-mirrors": ["https://mirror.gcr.io"]
}

Afterwards, available via localhost:8000

# Add a new superuser.

docker compose exec netbox /opt/netbox/netbox/manage.py createsuperuser




Kill instance -

docker compose down 

Bring instance up - 

docker compose up # add -d to run in background
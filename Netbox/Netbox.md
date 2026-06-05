Step 1 - Netbox as docker

git clone -b release https://github.com/netbox-community/netbox-docker.git
cd netbox-docker
# Copy the example override file
cp docker-compose.override.yml.example docker-compose.override.yml
# Read and edit the file to your liking
docker compose pull

docker compose up

Afterwards, available via localhost:8000

docker compose exec netbox /opt/netbox/netbox/manage.py createsuperuser

# Add a new superuser.


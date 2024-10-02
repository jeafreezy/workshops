### Building a Custom Geocoding Service using Open Source Tools

GISfest 2022 Conference Workshop: [Google Slides](https://docs.google.com/presentation/d/1v2oTioG59r1u6pkj7VHBWCZMB3rVPEo1EvhlnhsvFMI/edit?usp=sharing)

### Pre-requisites

- Docker
- Linux
- Command Line
- Linux server e.g Digital Ocean/AWS etc
- OSM knowledge
- IDE - VSCode

### Steps

1. **Server Setup- Digital Ocean**

- Create a droplet
- SSH into the droplet
- Create a new user
- Give super user privilege: [learn how](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04)
  - `adduser <user>`
  - `Password: <securepassword>`
- Grant privileges : `usermod -aG sudo <user>`
- Setup firewall

      #ufw app list
      #ufw allow OpenSSH
      #ufw enable
      #ufw status

- Login as root then switch user using: `su <user>`

2.  Install Docker and Docker compose. [learn how](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04)

        #sudo apt update
        #sudo apt install apt-transport-https ca-certificates curl software-properties-common
        #curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
        #sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
        #apt-cache policy docker-ce
        #sudo apt install docker-ce
        #sudo systemctl status docker

3.  Clone the [repo](https://github.com/mediagis/nominatim-docker):

- `git clone git@github.com:mediagis/nominatim-docker.git`
- `cd nominatim-docker/<version>`

4.  run the docker command:

        docker run -it \
        -e PBF_URL=https://download.geofabrik.de/europe/monaco-latest.osm.pbf \
        -e REPLICATION_URL=https://download.geofabrik.de/europe/monaco-updates/ \
        -p 8080:8080 \
        --name nominatim \
        mediagis/nominatim:4.1

5.  Test the deployment

- **BASE URL: server ipaddress**
- Check API Status - important in case of server failure. You can update user of failure/maintenance:

  - BASE URL/status?format=json|text|xml

- Forward Geocode with address:

  - BASE URL/search?q=address_to_lookup&limit=integer&format=json|text|xml

- Reverse geocode with coordinate:

  - BASE URL/reverse?lat=<value>&lon=<value>&format=json|text|xml

- lat = latitude and lon= longitude

- Reference: https://nominatim.org/release-docs/latest/api/Overview/

6. Reverse Proxy,Domain and SSL

- [Caddy](https://caddyserver.com/docs/install#digitalocean)
- Install caddy

      sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
      curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
      curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
      sudo apt update
      sudo apt install caddy

- Domain: point the A record in the domain provider to the server ip
- Add caddy to firewall before it can allow caddy to serve http and https to the server using this command: `sudo ufw allow proto tcp from any to any port 80,443`
- Use the one-liner command to reverse from the website address to localhost:
- `sudo caddy reverse-proxy --from domain.com --to localhost:8080`

7. Deployment considerations & Customization

- For high traffics and country/planet wide deployment
  - Use high server resources
  - Prepare for days of waiting e.g Planet import can take about 5-10 days on 4 cores and 64GB ram server.
- Customize & tune

        docker run -it \
        -e PBF_URL=https://download.geofabrik.de/africa/nigeria-latest.osm.pbf \
        -e REPLICATION_URL=https://download.geofabrik.de/africa/nigeria-updates/ \
        -p 8080:8080 \
        --name nominatim mediagis/nominatim:4.1
        -e REPLICATION_UPDATE_INTERVAL=604800 \
        -e POSTGRES_SHARED_BUFFER=500MB \
        -e POSTGRES_AUTOVACUUM_WORK_MEM=500MB \
        -e POSTGRES_EFFECTIVE_CACHE_SIZE=15GB \

8. Shortcomings

- Dependent on OSM data i.e if address is not mapped on OSM, the geocoding will return no result.
- Solve by contributing to OSM?

### Next Steps

- Abstract behind an API to provide support for:
  - rate limiting
  - tokens and authentication etc

### Credits

- [Nominatim community](https://github.com/osm-search/Nominatim)
- [Mediagis community](https://github.com/mediagis)

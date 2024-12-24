#!/bin/bash

# Check input files
if [ ! -f "kadena_address.txt" ] || [ ! -f "kadena_privatekey.txt" ]; then
  echo "Error: File kadena_address.txt or kadena_privatekey.txt does not exist!"
  exit 1
fi

# Read files
kadena_addresses=($(cat kadena_address.txt))
kadena_private_keys=($(cat kadena_privatekey.txt))

# Check the number of lines
if [ ${#kadena_addresses[@]} -ne ${#kadena_private_keys[@]} ]; then
  echo "Error: The number of Kadena addresses and private keys do not match!"
  exit 1
fi

# Create folders and docker-compose.yml files for each container
for i in "${!kadena_addresses[@]}"; do
  index=$((i + 1))
  address=${kadena_addresses[$i]}
  priv_key=${kadena_private_keys[$i]}
  base_port=$((31000 + i * 10))

  # Folder name
  folder="cyberfly/cyberfly_$index"

  # Create folder if it doesn't exist
  mkdir -p "$folder"

  # Create docker-compose.yml file in the folder
  cat <<EOL > "$folder/docker-compose.yml"
version: '3.8'

services:
  cyberflynodeui:
    image: "cyberfly/cyberfly_node_ui:latest"
    restart: always
    ports:
      - "$((base_port)):80" #nginx server port
    depends_on:
      - cyberflynode
    deploy:
      resources:
        limits:
          cpus: "1"

  cyberflynode:
    image: "cyberfly/cyberfly_node:latest"
    restart: always
    ports:
      - "$((base_port + 1)):31001" #libp2p tcp port
      - "$((base_port + 2)):31002" #libp2p websocket port
      - "$((base_port + 3)):31003" #Cyberfly API port
    volumes:
      - ./data:/usr/src/app/data
    environment:
      - KADENA_ACCOUNT=$address
      - NODE_PRIV_KEY=$priv_key
      - MQTT_HOST=mqtt://cyberflymqtt
      - REDIS_HOST=redisstackserver
    depends_on:
      - cyberflymqtt
      - redisstackserver
    deploy:
      resources:
        limits:
          cpus: "1"

  cyberflymqtt:
    image: "cyberfly/cyberfly_mqtt:latest"
    restart: always
    ports:
      - "$((base_port + 4)):1883" #mqtt tcp port
      - "$((base_port + 5)):9001" #mqtt websocket port
    deploy:
      resources:
        limits:
          cpus: "1"

  redisstackserver:
    image: "redis:8.0-M02-alpine3.20"
    restart: always
    volumes:
      - ./redis-data:/data
    deploy:
      resources:
        limits:
          cpus: "1"

  watchtower:
    image: "containrrr/watchtower"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    restart: always
    environment:
      - WATCHTOWER_POLL_INTERVAL=30
      - WATCHTOWER_CLEANUP=true
    deploy:
      resources:
        limits:
          cpus: "1"
EOL

  echo "Created $folder/docker-compose.yml"

  # Navigate to the folder and run docker-compose up -d
  (cd "$folder" && docker-compose up -d)
  echo "Started container in $folder"
done

echo "All containers have been created and started successfully!"

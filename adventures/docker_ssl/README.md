## Docker SSH
Create two ubuntu with add user and ssl

### Simple Docker SSH Setup

```yaml
#!/bin/bash
# simple_setup.sh - Create two Ubuntu containers with SSH access

# Create a simple Dockerfile with SSH pre-configured
cat > Dockerfile << 'EOF'
FROM ubuntu:22.04

# Install SSH and required packages
RUN apt-get update && apt-get install -y openssh-server iproute2 sudo && \
    mkdir -p /run/sshd

# Create a user with password-less sudo
RUN useradd -m -s /bin/bash user && \
    echo "user ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/user && \
    chmod 440 /etc/sudoers.d/user

# Set up SSH to allow password authentication (for simplicity)
RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config

# Set a simple password for the user
RUN echo 'user:password' | chpasswd

# Start SSH server
CMD ["/usr/sbin/sshd", "-D"]
EOF

# Create a simple docker-compose file
cat > docker-compose.yml << 'EOF'
version: '3'
services:
  ubuntu1:
    build: .
    container_name: ubuntu1
    networks:
      - ssh_network
  
  ubuntu2:
    build: .
    container_name: ubuntu2
    networks:
      - ssh_network

networks:
  ssh_network:
    driver: bridge
EOF

# Build and start the containers
docker-compose down -v 2>/dev/null || true
docker-compose build
docker-compose up -d

# Get container IPs
UBUNTU1_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ubuntu1)
UBUNTU2_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ubuntu2)

echo "==============================="
echo "Setup Complete!"
echo "==============================="
echo "Container ubuntu1 IP: $UBUNTU1_IP"
echo "Container ubuntu2 IP: $UBUNTU2_IP"
echo
echo "To SSH between containers:"
echo "  From ubuntu1 to ubuntu2: docker exec -it ubuntu1 ssh user@$UBUNTU2_IP"
echo "  From ubuntu2 to ubuntu1: docker exec -it ubuntu2 ssh user@$UBUNTU1_IP"
echo
echo "SSH credentials:"
echo "  Username: user"
echo "  Password: password"
echo "==============================="
```

### Secure Docker Containers with SSH Keys
We can secure the SSH access by using SSH keys instead of password authentication. 

```yaml
#!/bin/bash
# simple_setup.sh - Create two Ubuntu containers with SSH access

# Create a simple Dockerfile with SSH pre-configured
cat > Dockerfile << 'EOF'
FROM ubuntu:22.04

# Install SSH and required packages
RUN apt-get update && apt-get install -y openssh-server iproute2 sudo && \
    mkdir -p /run/sshd

# Create a user with password-less sudo
RUN useradd -m -s /bin/bash user && \
    echo "user ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/user && \
    chmod 440 /etc/sudoers.d/user

# Set up SSH to allow key-based authentication and disable password authentication
RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config && \
    sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config

# Create .ssh directory for the user
RUN mkdir -p /home/user/.ssh && \
    chown user:user /home/user/.ssh && \
    chmod 700 /home/user/.ssh

# Start SSH server
CMD ["/usr/sbin/sshd", "-D"]
EOF

# Create a simple docker-compose file
cat > docker-compose.yml << 'EOF'
version: '3'
services:
  ubuntu1:
    build: .
    container_name: ubuntu1
    networks:
      - ssh_network
    volumes:
      - ~/.ssh/id_rsa.pub:/tmp/id_rsa.pub
    command: sh -c "cat /tmp/id_rsa.pub >> /home/user/.ssh/authorized_keys && /usr/sbin/sshd -D"

  ubuntu2:
    build: .
    container_name: ubuntu2
    networks:
      - ssh_network
    volumes:
      - ~/.ssh/id_rsa.pub:/tmp/id_rsa.pub
    command: sh -c "cat /tmp/id_rsa.pub >> /home/user/.ssh/authorized_keys && /usr/sbin/sshd -D"

networks:
  ssh_network:
    driver: bridge
EOF

# Build and start the containers
docker-compose down -v 2>/dev/null || true
docker-compose build
docker-compose up -d

# Get container IPs
UBUNTU1_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ubuntu1)
UBUNTU2_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ubuntu2)

echo "==============================="
echo "Setup Complete!"
echo "==============================="
echo "Container ubuntu1 IP: $UBUNTU1_IP"
echo "Container ubuntu2 IP: $UBUNTU2_IP"
echo
echo "To SSH between containers:"
echo "  From ubuntu1 to ubuntu2: docker exec -it ubuntu1 ssh user@$UBUNTU2_IP"
echo "  From ubuntu2 to ubuntu1: docker exec -it ubuntu2 ssh user@$UBUNTU1_IP"
echo
echo "SSH credentials:"
echo "  Username: user"
echo "  Password: password"
echo "==============================="
```

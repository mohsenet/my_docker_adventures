### Install docker with snap in Ubuntu 22.04:

```bash
# Install Snap
sudo apt update
sudo apt install snapd

# Insall Docker with Snap
sudo snap install docker

# Verify installation
docker --version

# Add your user to the docker group
sudo usermod -aG docker $USER
```
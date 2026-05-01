#Sample 

#SSH
ssh-keygen -t ed25519 -C "your_email@example.com-2026"

chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
cat ~/.ssh/id_ed25519




# Update and install prerequisites
echo "Installing prerequisites..."
sudo apt update && sudo apt install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  software-properties-common \
  tree\
  git
  
  
# Set email for git commits and ssh key for the above automation
  git config --global user.email "mosesimbahale0@gmail.com"
  git config --global user.name "Moses Imbahale"
  
  
  
  

# Docker and Docker Compose
#!/bin/bash

# Function to run the setup
run_setup() {
    echo "🔎 Performing detailed environment audit..."

    # --- 1. Version Detection ---
    DOCKER_V=$(docker --version 2>/dev/null | awk '{print $3}' | sed 's/,//' || echo "NOT INSTALLED")
    COMPOSE_V=$(docker compose version 2>/dev/null | awk '{print $4}' || echo "NOT INSTALLED")
    
    # Extracting plugin version from dpkg for precision
    MODEL_V=$(dpkg-query -W -f='${Version}' docker-model-plugin 2>/dev/null || echo "NOT INSTALLED")
    
    # GPU detection
    if command -v nvidia-smi &>/dev/null; then
        GPU_NAME=$(nvidia-smi --query-gpu=name --format=csv,noheader 2>/dev/null)
        GPU_DRV=$(nvidia-smi --query-gpu=driver_version --format=csv,noheader 2>/dev/null)
        GPU_STATUS="DETECTED ($GPU_NAME | Driver: $GPU_DRV)"
        GPU_FLAG="--gpu cuda"
    else
        GPU_STATUS="NOT FOUND (Falling back to CPU)"
        GPU_FLAG="--gpu none"
    fi

    # Service & Runner Status
    DOCKER_SVC=$(systemctl is-active docker 2>/dev/null || echo "inactive")
    RUNNER_SVC=$(docker model status 2>/dev/null | grep -i "running" >/dev/null && echo "active" || echo "inactive")

    # --- 2. Verbose Status Report ---
    echo "------------------------------------------------"
    echo "📦 Docker Engine:        $DOCKER_V"
    echo "📦 Docker Compose:       $COMPOSE_V"
    echo "📦 Model Plugin:         $MODEL_V"
    echo "🖥️  GPU Hardware:         $GPU_STATUS"
    echo "⚙️  Docker Daemon:        $DOCKER_SVC"
    echo "🤖 Model Runner:         $RUNNER_SVC"
    echo "------------------------------------------------"

    # --- 3. Idempotent Check ---
    if [[ "$DOCKER_V" != "NOT INSTALLED" && "$MODEL_V" != "NOT INSTALLED" && "$DOCKER_SVC" == "active" && "$RUNNER_SVC" == "active" ]]; then
        echo "🚀 All systems nominal. Environment is fully optimized."
        return 0
    fi

    echo "⚠️  Incomplete stack detected. Initializing setup/repair..."

    # --- 4. Installation ---
    sudo apt update && sudo apt install -y ca-certificates curl gnupg lsb-release tree git

    # Setup GPG & Repo
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://docker.com | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg --yes
    
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://docker.com \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    # Install everything
    sudo apt update
    sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-model-plugin

    # Start services
    sudo systemctl enable --now docker
    
    echo "⚙️  Configuring Model Runner for Hardware: $GPU_FLAG"
    docker model install-runner $GPU_FLAG 2>/dev/null || docker model start-runner

    # User Permissions
    if ! groups $USER | grep &>/dev/null "\bdocker\b"; then
        sudo usermod -aG docker $USER
        echo "✅ Added $USER to docker group."
    fi

    echo "✨ VERBOSE SETUP COMPLETED SUCCESSFULLY!"
    echo "💡 Note: If this is a fresh install, run 'newgrp docker' to use Docker without sudo."
}

run_setup




# Other setups



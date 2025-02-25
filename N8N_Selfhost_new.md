# n8n Self-Hosting with Ngrok

## Introduction

[n8n](https://n8n.io/) is a powerful workflow automation tool that allows you to connect various services and automate tasks. This guide provides step-by-step instructions on how to expose an existing n8n self-hosted instance to the internet securely with Ngrok.

## What is Ngrok?
Ngrok is a tool that creates secure tunnels to your localhost, allowing you to expose local web servers behind NATs and firewalls to the internet.


## Prerequisites

Before getting started, ensure you have the following installed on your system:

- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [n8n self-hosted-ai-starter-kit](https://github.com/n8n-io/self-hosted-ai-starter-kit)
- [Ngrok account](https://dashboard.ngrok.com/signup) (free plan works for testing)
- [Ngrok CLI](https://ngrok.com/download)


## Docker Installation
### Download Docker Desktop

1. **Visit the Official Download Page:**
   - Navigate to the [Docker Desktop download page](https://www.docker.com/products/docker-desktop).

2. **Select Your Operating System:**
   - Click on the **Download** button for your OS (Windows or macOS).

3. **Save the Installer:**
   - Once downloaded, locate the installer file on your computer.

### Install Docker Desktop
#### For Windows
1. **Run the Installer:**
   - Double-click the downloaded `Docker Desktop Installer.exe` file.

2. **Follow the Installation Prompts:**
   - Accept the license agreement and follow the on-screen instructions.
   - If prompted, grant administrative privileges.

3. **Enable WSL 2 (if using Windows Home):**
   - Windows Home users need to have WSL 2 installed. Follow the [WSL 2 Installation Guide](https://docs.microsoft.com/en-us/windows/wsl/install) if you haven’t already set it up.

4. **Complete Installation:**
   - Once installation is complete, Docker Desktop will start automatically.

#### For macOS
1. **Open the Installer:**
   - Open the downloaded `.dmg` file.

2. **Install Docker Desktop:**
   - Drag and drop the Docker icon into the **Applications** folder.

3. **Launch Docker Desktop:**
   - Open the **Applications** folder and double-click **Docker** to start the application.
   - You may be prompted to authorize Docker with your system password.

4. **Complete Setup:**
   - Follow any additional on-screen prompts to finalize the setup.

### Verify the Installation
1. **Open a Terminal:**
   - On **Windows**, use Command Prompt or PowerShell.
   - On **macOS**, use the Terminal application.

2. **Check Docker Version:**
   ```bash
   docker --version
   ```
   
## n8n Installation
### Cloning the Repository

```bash
git clone https://github.com/n8n-io/self-hosted-ai-starter-kit.git
cd self-hosted-ai-starter-kit
```

### Running n8n using Docker Compose
#### For Nvidia GPU users

```
docker compose --profile gpu-nvidia up
```

#### For AMD GPU users on Linux

```
docker compose --profile gpu-amd up
```

#### For Mac / Apple Silicon users
If you’re using a Mac with an M1 or newer processor, you can't expose your GPU
to the Docker instance, unfortunately. There are two options in this case:

1. Run the starter kit fully on CPU, like in the section "For everyone else"
   below
2. Run Ollama on your Mac for faster inference, and connect to that from the
   n8n instance

If you want to run Ollama on your mac, check the
[Ollama homepage](https://ollama.com/)
for installation instructions, and run the starter kit as follows:

```
docker compose up
```

#### For Mac users running OLLAMA locally

If you're running OLLAMA locally on your Mac (not in Docker), you need to modify the OLLAMA_HOST environment variable
in the n8n service configuration. Update the x-n8n section in your Docker Compose file as follows:

```yaml
x-n8n: &service-n8n
  # ... other configurations ...
  environment:
    # ... other environment variables ...
    - OLLAMA_HOST=host.docker.internal:11434
```

#### For everyone else

```
docker compose --profile cpu up
```
### Verify the Installation
1. **Open a Terminal:**
   - On **Windows**, use Command Prompt or PowerShell.
   - On **macOS**, use the Terminal application.

2. **Check running containers:**
   ```
   docker ps
   ```

## Ngrok Installation
#### Download Ngrok

1. Go to the [official ngrok download page](https://ngrok.com/download).
2. Download the version for your operating system (Windows, macOS, Linux).
3. Unzip the downloaded file.
4. Move the executable to a directory included in your system's PATH.

#### Install via Package Manager

- **Windows (using Chocolatey):**
  ```powershell
  choco install ngrok
  ```

- **macOS (using Homebrew):**
  ```bash
  brew install --cask ngrok
  ```

- **Linux (using Snap):**
  ```bash
  sudo snap install ngrok
  ```

#### Authenticate ngrok

1. Sign up or log in to your [ngrok dashboard](https://dashboard.ngrok.com/).
2. Copy your **Authtoken** from the dashboard.
3. Run the following command to authenticate:
   
   ```bash
   ngrok config add-authtoken <YOUR_AUTHTOKEN>
   ```

## Expose n8n to the Internet Using Ngrok

1. Start Ngrok and expose port 5678:
   ```sh
   ngrok http http://localhost:5678
   ```
   ![image](https://github.com/user-attachments/assets/b91179f9-e747-46a8-b498-fa878b49d50c)

2. Ngrok will provide a forwarding URL (e.g., `https://randomsub.ngrok.io`). Copy this URL.
   
   ![image](https://github.com/user-attachments/assets/c1a4dd36-739b-4f43-8807-af44e20e5775)

4. Stop the running n8n container:

    ```sh
   docker compose down
   docker compose --profile cpu down
   docker compose --profile gpu-amd down
     ```

6. Update your n8n environment variables to include the Ngrok URL:

    ```sh
   WEBHOOK_URL=https://20b3-197-211-58-214.ngrok-free.app
   ```

8. Restart n8n with the updated configuration:

    ```sh
   docker-compose up -d
   docker compose --profile gpu-amd up -d
   docker compose --profile cpu up -d
   ```

## Access n8n

1. Open your browser and visit `http://localhost:5678`
2. Log in with your existing credentials.
3. To use webhooks in workflows, ensure that external services send requests to the Ngrok URL (`https://20b3-197-211-58-214.ngrok-free.app`).
   
   ![image](https://github.com/user-attachments/assets/c5e1a0c1-46b1-44e1-93df-ab1c3641fa4c)

## Set Up a Static Domain (Free Plan)

Ngrok now allows the use of static domains even on the free plan. This provides a persistent URL that doesn't change every time you restart the tunnel.

### Steps to Get a Static Domain:

1. Go to your ngrok dashboard.
2. Navigate to the Deploy your app online section.
3. Under the Static Domain tab, copy the provided URL and command.
   ![image](https://github.com/user-attachments/assets/843975ef-62da-470f-8cff-111aab174aa2)
4. Replace the temporary url with the static url provided to you earlier in the environment variables

   ```sh
   WEBHOOK_URL=https://lark-striking-wholly.ngrok-free.app
   ```

5. Run the command in your terminal.

    ```sh
   ngrok http --url=lark-striking-wholly.ngrok-free.app 5678
   ```

Now, you have a permanent/static link that will remain the same as long as you use this command.

## Troubleshooting

- If n8n doesn’t start correctly, check the logs:
  ```sh
  docker-compose logs -f
  ```
- Ensure your Ngrok session is running; otherwise, restart it and update the `WEBHOOK_URL`.

## Conclusion

You have successfully exposed your self-hosted n8n instance to the internet securely using Ngrok. This setup enables you to develop and test workflows that require external webhook connections.

For more details, visit the [n8n documentation](https://docs.n8n.io/), [Ngrok Documentation](https://ngrok.com/docs), [Docker Documentation](https://docs.docker.com).

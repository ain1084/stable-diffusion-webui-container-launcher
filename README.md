# stable-diffusion-webui-container

[📖 日本語版 (README_ja.md)](./README_ja.md)

An environment for running Stable Diffusion WebUI in a Docker container using VSCode Dev Containers.

## Features

* Support for recent NVIDIA GPUs (Blackwell generation)
* Uses SDP Attention (xformers not required)
* VSCode Dev Containers workflow - no Dockerfile customization needed
* Bind-mount for models and outputs
* Minimal setup - only devcontainer.json configuration required

This is not a beginner-friendly one-click installer. At least basic knowledge of Docker, VSCode Dev Containers, and Linux command-line operations is required.

---

## Requirements

This repository assumes basic knowledge of the following:

* Docker
* VSCode Dev Containers
* Linux command-line operations

### Host (GPU-equipped PC) Requirements

* NVIDIA GPU
* NVIDIA Driver (latest)
* NVIDIA Container Toolkit
* Docker

### Local PC (VSCode Execution Environment) Requirements

* VSCode
* Dev Containers extension

**For Remote Connections:** If the host and local PC are different machines, you can use this setup as long as the local PC runs VSCode and the Dev Containers extension, and the host has Docker and NVIDIA Container Toolkit running.

---

## Test Environment

This container has been tested on the following host:

| Component | Version |
| --- | --- |
| Host OS | Ubuntu Server 26.04 LTS |
| GPU | NVIDIA GeForce RTX 5080 16GB |
| NVIDIA Driver | 595.71.05 |
| Docker | 29.5.0 build 98f1464 |
| NVIDIA Container Toolkit | Installed |
| VSCode | Latest |
| Dev Containers Extension | Latest |

---

## GPU / CUDA Notes

This container depends on the NVIDIA driver installed on the host.

Please verify the following:

* NVIDIA driver is correctly installed
* NVIDIA Container Toolkit is configured
* Docker can access the GPU

You can verify this using:

```bash
docker run --rm --gpus all debian:stable-slim nvidia-smi
```

If GPU access is working correctly, `nvidia-smi` in the container will display GPU information.

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/ain1084/stable-diffusion-webui-container-launcher
cd stable-diffusion-webui-container-launcher
```

### 2. Create devcontainer.json

#### 2.1 Copy devcontainer.json

A [`devcontainer.example.json`](./.devcontainer/devcontainer.example.json) is provided in the `.devcontainer` directory. Copy it to `.devcontainer/devcontainer.json`.

**Important:** Creating `devcontainer.json` is required. Without this configuration file, VSCode cannot launch the Dev Container.

#### 2.2 Verify/Customize Mount Settings (Optional)

By default, `devcontainer.json` includes the following mount settings. These are examples of recommended directories to persist on the host:

```text
Repository Root
└── stable-diffusion-webui/
    ├── models/Stable-diffusion/   # Model checkpoints
    ├── models/Lora/               # LoRA models
    ├── extensions/                # Extensions directory (independent of WebUI core)
    ├── outputs/                   # Generated images
    └── log/                       # Logs
```

**Note:** When you clone this repository, these directories already exist (with `.gitignore` files). You don't need to run `mkdir -p` separately.

With the mount settings, these directories are bind-mounted to the container as follows:

| Host (Workspace) | Path in Container | Purpose |
| --- | --- | --- |
| `stable-diffusion-webui/models/Stable-diffusion` | `/opt/stable-diffusion-webui/models/Stable-diffusion` | Model checkpoints |
| `stable-diffusion-webui/models/Lora` | `/opt/stable-diffusion-webui/models/Lora` | LoRA models |
| `stable-diffusion-webui/extensions` | `/opt/stable-diffusion-webui/extensions` | Extensions directory |
| `stable-diffusion-webui/outputs` | `/opt/stable-diffusion-webui/outputs` | Generated images |
| `stable-diffusion-webui/log` | `/opt/stable-diffusion-webui/log` | Log files |

**Customizing Mount Settings:**

If your models are stored in a different location on the host or you want to mount other directories, edit the mount settings in `devcontainer.json`.

**Example:** If models are in `~/shared/sd/models/` on the host:

```json
"mounts": [
    "source=${localEnv:HOME}/shared/sd/models,target=/opt/stable-diffusion-webui/models/Stable-diffusion,type=bind",
    // ... etc
]
```

※ `${localEnv:HOME}` refers to the user's home directory on the host.

The container can still launch without modifying the mount settings, but on first startup, the model directory (`stable-diffusion-webui/models/Stable-diffusion`) will be empty, and the default model (~3GB) will be downloaded by the WebUI.

**Restarting After Changing Mount Settings:**

If you change the mount settings in `devcontainer.json`, you need to rebuild the Dev Container to apply the changes.

After making changes, open the VSCode command palette (Ctrl+Shift+P) and run:

- `Dev Containers: Rebuild Container`

This will recreate the container with the new mount settings.

※ Changes may not be reflected with `Reopen in Container`.

**Important Notes:**

* **Mount settings can be added or changed later**  
  Edit `devcontainer.json` as needed. After changes, `Rebuild Container` is required.

* **Files saved only in the container may be lost**  
  Running `Rebuild Container` recreates the container with the new settings. Data in directories that are not bind-mounted may be lost.

* **Save important data to the host (workspace)**  
  It's recommended to save models, generated images, and configuration files to bind-mounted host directories to preserve them.

#### 2.3 Customize PyTorch Installation Command (Optional)

By default, PyTorch is installed using `pip install torch`, which installs the latest version. On older GPUs, the latest version of PyTorch may not work. In that case, specify the corresponding CUDA version.

**How to Configure:**

Specify `TORCH_INSTALL_COMMAND` in the `args` section of `devcontainer.json`:

```json
"args": {
    "TORCH_INSTALL_COMMAND": "pip install torch --extra-index-url https://download.pytorch.org/whl/cu124"
}
```

**Recommended CUDA Versions by GPU Generation:**

Refer to the table below as a guide:

| GPU Generation | Recommended CUDA Version | Configuration Example |
| --- | --- | --- |
| Pascal Generation (GeForce GTX 10xx Series, etc.) | cu121 / cu124 | `pip install torch --extra-index-url https://download.pytorch.org/whl/cu124` |
| RTX 20/30/40/50 Series | Latest version | `pip install torch`（default） |

### 3. Open in VSCode and Reopen in Container

1. Open the repository folder in VSCode
2. When prompted, click **"Reopen in Container"** or use **Ctrl+Shift+P** to type "Reopen in Container"
3. The container will automatically build and start (this may take a few minutes on first run)

### 4. Run the WebUI

Once the container is ready, follow the content displayed in the terminal (WELCOME.txt) to start the WebUI.

You can run the WebUI startup task with:

```text
Ctrl+Shift+B
```

### 5. Access the WebUI

Once the WebUI is running, you can access it from your browser at:

```text
http://localhost:7860
```

This environment uses VSCode Dev Containers' port forwarding feature. If port 7860 is already in use locally, it may be automatically forwarded to a different port.

You can check the actual URL in the "PORTS" tab of VSCode.

**How to Restart:** Press Ctrl+C in the terminal to stop the WebUI, then press Ctrl+Shift+B to run the task again.

**If You Can't Access in Browser:** Check the "PORTS" tab in VSCode to verify that port 7860 is being forwarded. If auto port forwarding is not working, you may need to manually add port forwarding.

## Customization

### 1. Change stable-diffusion-webui Commit

The stable-diffusion-webui commit is pinned using the `SD_WEBUI_COMMIT` build argument:

```json
"args": {
    "SD_WEBUI_COMMIT": "1937682a20f7f0442311a1ede68f9f0cb480163b"
}
```

You can modify this in [.devcontainer/devcontainer.json](.devcontainer/devcontainer.json) to use a different commit.

**Important:** If you change `SD_WEBUI_COMMIT`, you need to **rebuild the container (Rebuild Container)**. Simply restarting the WebUI will not reflect the changes. Run the "Rebuild Container" command in VSCode.

### 2. Launch Options / Tasks

In this environment, WebUI launch options are managed through VSCode tasks. Instead of modifying webui-user.sh, edit the VSCode task ([`.vscode/tasks.json`](.vscode/tasks.json)) to add or change launch options.

The following launch options are specified:

* `--opt-sdp-attention` : Uses PyTorch's native SDP Attention instead of xformers (see also the [xformers section](#3-about-xformers)).
* `--api`: Enables the WebUI REST API. External applications can use the WebUI API.

The `--listen` option is not used. This is because VSCode Dev Containers' port forwarding feature allows access to the WebUI from the local machine running VSCode.

### 3. About xformers

This environment does not install or use xformers.

Xformers can cause compatibility issues in recent NVIDIA GPU environments. In particular, Blackwell generation GPUs may experience issues such as xformers not running, requiring a build, or CUDA mismatches.

Instead, this environment is configured with PyTorch SDP attention options (see the [Launch Options section](#2-launch-options--tasks)).

### 4. Extensions

Extensions are not pre-installed. The `extensions` directory is bind-mounted by default, so it can be used for installation via the WebUI. You can also manually add extensions by placing them in `stable-diffusion-webui/extensions/`. Please refer to each extension's documentation for more details.

---

## License

The configuration files and Docker-related files in this repository are released under the MIT License.

For dependent software, please follow their respective licenses:

* [AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) - [LICENSE](https://github.com/AUTOMATIC1111/stable-diffusion-webui/blob/master/LICENSE.txt)

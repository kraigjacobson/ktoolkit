# AI Toolkit Docker Setup

This project contains a Docker Compose configuration for running [Ostris AI Toolkit](https://github.com/ostris/ai-toolkit) in a containerized environment with all required dependencies.

## Prerequisites

- **Docker** (20.10 or higher)
- **Docker Compose** (2.0 or higher)
- **NVIDIA GPU** with at least 24GB VRAM (for FLUX.1 training)
- **NVIDIA Container Toolkit** installed and configured

### Installing NVIDIA Container Toolkit

If you haven't installed the NVIDIA Container Toolkit yet:

**Ubuntu/Debian:**

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```

Verify GPU access:

```bash
docker run --rm --gpus all nvidia/cuda:12.8.1-base-ubuntu24.04 nvidia-smi
```

## Quick Start

### 1. Configure Environment Variables

Copy the example environment file and edit it with your credentials:

```bash
cp .env.example .env
nano .env  # or use your preferred editor
```

Required variables:

- `AI_TOOLKIT_AUTH`: Password to protect the web UI
- `HF_TOKEN`: HuggingFace token (required for FLUX.1-dev and other gated models)
  - Get a READ token from: https://huggingface.co/settings/tokens
  - Accept FLUX.1-dev license at: https://huggingface.co/black-forest-labs/FLUX.1-dev

### 2. Start the Container

```bash
docker-compose up -d
```

This will:

- Pull the latest `ostris/aitoolkit` image
- Mount necessary directories for datasets, outputs, and configs
- Start the AI Toolkit UI on port 8675
- Enable GPU access for training

### 3. Access the Web UI

Open your browser and navigate to:

```
http://localhost:8675
```

Login with the password you set in `AI_TOOLKIT_AUTH`.

## Project Structure

```
.
├── docker-compose.yml       # Docker Compose configuration
├── .env                     # Environment variables (create from .env.example)
├── .env.example            # Example environment variables
├── README.md               # This file
└── ai-toolkit/             # AI Toolkit repository (cloned)
    ├── datasets/           # Your training datasets
    ├── output/             # Training outputs and models
    ├── config/             # Training configuration files
    └── aitk_db.db         # UI database
```

## Usage

### View Logs

```bash
docker-compose logs -f
```

### Stop the Container

```bash
docker-compose down
```

### Restart the Container

```bash
docker-compose restart
```

### Update to Latest Version

```bash
docker-compose pull
docker-compose up -d
```

## Training Workflows

### Using the Web UI

1. Navigate to http://localhost:8675
2. Upload your training images to the datasets folder
3. Configure your training parameters through the UI
4. Start training jobs and monitor progress

### Using Config Files (CLI)

1. Create a config file in [`ai-toolkit/config/`](ai-toolkit/config/)
2. Copy an example from [`ai-toolkit/config/examples/`](ai-toolkit/config/examples/)
3. Execute training inside the container:

```bash
docker-compose exec ai-toolkit bash
cd /app/ai-toolkit
python run.py config/your_config.yml
```

## Volumes and Data Persistence

The following directories are mounted to persist your data:

- **datasets/**: Training images and captions
- **output/**: Trained models, checkpoints, and samples
- **config/**: Training configuration files
- **huggingface-cache**: Model cache (named Docker volume)

## Environment Variables

| Variable          | Description           | Required         | Default      |
| ----------------- | --------------------- | ---------------- | ------------ |
| `AI_TOOLKIT_AUTH` | Password for web UI   | Yes              | `password`   |
| `HF_TOKEN`        | HuggingFace API token | For gated models | -            |
| `TZ`              | Timezone              | No               | `UTC`        |
| `NODE_ENV`        | Node environment      | No               | `production` |

## Troubleshooting

### GPU Not Detected

Verify NVIDIA runtime is available:

```bash
docker run --rm --gpus all nvidia/cuda:12.8.1-base-ubuntu24.04 nvidia-smi
```

### Permission Issues

Ensure the directories have proper permissions:

```bash
chmod -R 755 ai-toolkit/datasets ai-toolkit/output ai-toolkit/config
```

### Container Crashes on Start

Check logs:

```bash
docker-compose logs ai-toolkit
```

### Out of Memory

- Ensure you have at least 24GB VRAM for FLUX.1
- Set `low_vram: true` in your config if using GPU for display
- Close other GPU-intensive applications

## Resources

- [AI Toolkit Documentation](https://github.com/ostris/ai-toolkit)
- [AI Toolkit Discord](https://discord.gg/VXmU2f5WEU)
- [FLUX.1 Training Tutorial](https://www.youtube.com/watch?v=HzGW_Kyermg)

## License

This Docker setup is provided as-is. AI Toolkit itself is licensed under its own terms. Check the [ai-toolkit repository](https://github.com/ostris/ai-toolkit) for details.

## Support

For issues related to:

- **Docker setup**: Open an issue in this repository
- **AI Toolkit functionality**: Join the [AI Toolkit Discord](https://discord.gg/VXmU2f5WEU) or check the [official repository](https://github.com/ostris/ai-toolkit)

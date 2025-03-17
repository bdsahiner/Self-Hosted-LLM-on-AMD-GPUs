# Prerequisites

- Docker or Podman
- ROCm Drivers (Installed and configured)

# Deploy Ollama

### Running Ollama with GPU Overrides (Supported GPUs)

For AMD GPUs **officially supported** by ROCm, use the following command to run Ollama in Docker:
 
 $ `docker run -d --device /dev/kfd --device /dev/dri -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama:rocm`

### Running Ollama with GPU Overrides (For Unsupported GPUs)

**!WARNING**:  From this point onward, the instructions are not officially supported. Use them at your own risk.

Even if your GPU is **not officially supported** (e.g., RX 6700 XT, RX 6600), you can still attempt to use Ollama in Docker with GPU acceleration. However, this requires overriding certain settings, and results may vary.

To determine your GPU model, run the following command in a Linux terminal:

 $ `lspci | grep -i vga` 

This will output details about your installed graphics card. For example: 

`VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Navi 22 [Radeon RX 6700/6700 XT/6750 XT / 6800M/6850M XT] (rev c1)`

Once you identify your GPU model, check its corresponding **GFX version** in this table:

|GPU Series*	    | Navi Version	       | GFX Version*	    | HSA Override*                   |
| ----------------- | -------------------- | ------------------ | ------------------------------- |
|RDNA 1 (RX 5000)   | Navi 10 / 12 / 14	   | gfx1010–gfx1012	| HSA_OVERRIDE_GFX_VERSION=10.1.0 |
|RDNA 2 (RX 6000)	| Navi 21 (Big Navi)   |gfx1030	            | HSA_OVERRIDE_GFX_VERSION=10.3.0 |
|                   | Navi 22	           |gfx1031	            | HSA_OVERRIDE_GFX_VERSION=10.3.1 |
|                   | Navi 23	           |gfx1032	            | HSA_OVERRIDE_GFX_VERSION=10.3.2 |
|                   | Navi 24	           |gfx1033	            | HSA_OVERRIDE_GFX_VERSION=10.3.3 |
|                   | APU Variants	       |gfx1035	            | HSA_OVERRIDE_GFX_VERSION=10.3.5 |
|RDNA 3 (RX 7000)	| Navi 31	           |gfx1100	            | HSA_OVERRIDE_GFX_VERSION=11.0.0 |
|                   | Navi 32	           |gfx1101	            | HSA_OVERRIDE_GFX_VERSION=11.0.1 |
|                   | Navi 33	           |gfx1102	            | HSA_OVERRIDE_GFX_VERSION=11.0.2 |
|                   | APU Variants	       |gfx1103	            | HSA_OVERRIDE_GFX_VERSION=11.0.3 |
|Vega (MI50, MI60)	| Vega 20	           |gfx906	            | HSA_OVERRIDE_GFX_VERSION=9.0.6  |
|CDNA (MI100, MI200)| Arcturus (MI100)	   |gfx908	            | HSA_OVERRIDE_GFX_VERSION=9.0.8  |
|                   | Aldebaran (MI200)	   |gfx90A	            | HSA_OVERRIDE_GFX_VERSION=9.0.A  |

Note: Some ROCm applications require overriding the GFX version due to compatibility issues. The format for **HSA_OVERRIDE_GFX_VERSION** is typically **X.Y.Z**, derived from the GPU’s microarchitecture.

Since I am using an AMD RX 6700 XT, which is based on Navi 22 and has a GFX version of gfx1031 (HSA override 10.3.1), I have chosen to override it to gfx1030 (HSA override 10.3.0) for better compatibility. AMD officially supports only high-end RDNA 2 GPUs (gfx1030), as these are primarily used for workstation and compute tasks. Mid-tier GPUs like the RX 6700 XT (gfx1031) are not officially supported, but overriding them to gfx1030 allows for potential compatibility with ROCm.

| GFX Version |	GPU Type	       | Example GPUs	         | ROCm Official Support    |
|------------ | ------------------ | ----------------------- | ------------------------ |
| gfx1030	  | High-end RDNA 2	   |  RX 6800, RX 6900 XT	 | **Officially Supported** |
| gfx1031	  | Mid-tier RDNA 2	   |  RX 6700 XT, 6750 XT	 | Not Officially Supported |
| gfx1032	  | Lower-mid RDNA 2   |  RX 6600 XT, 6600M	     | Not Officially Supported |
| gfx1033	  | Entry-level RDNA 2 |  RX 6500 XT	         | Not Officially Supported |

Based on this information, I am using the following command to run my Docker container with Ollama on my AMD RX 6700 XT:

 $ `docker run -d -e HSA_OVERRIDE_GFX_VERSION=10.3.0 -e HCC_AMDGPU_TARGET=gfx1030 --restart always --device /dev/kfd --device /dev/dri -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama:rocm`

# Deploy OpenWebUI

The recommended way to deploy OpenWebUI using Docker is with the following command:

 $ `docker run -d --network=host -v open-webui:/app/backend/data -e OLLAMA_BASE_URL=http://127.0.0.1:11434 --name open-webui --restart always ghcr.io/open-webui/open-webui:main`


# Post Installation

1) Be sure the containers are deployed and their status are healthy.

 $ `docker ps -a`

2) Make sure that port 11434 (Ollama) and default host network ports (OpenWebUI) are not being used by other applications.

3) Ollama does not come pre-installed with models, so you must pull a model before using it.

4) Once both containers are running, open OpenWebUI in your browser.
    `http://127.0.0.1:8080`

5) If you want Ollama and OpenWebUI to start automatically after a system reboot (Optional), ensure their Docker containers are set to restart:

 $ `docker update --restart always ollama`
 $ `docker update --restart always open-webui`


# Uninstall Ollama & OpenWebUI

 $ `docker rm -f open-webui ollama`

# Disclaimer

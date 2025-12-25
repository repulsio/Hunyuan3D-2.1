# Run `Hunyuan3D-2.1` in Cloud

This is working on the latest `Hunyuan3D-2.1` commit ([``](https://github.com/Tencent-Hunyuan/Hunyuan3D-2.1/commit/)).

<br/>

## HyperStack

You can run a `RTX-A6000` VM on **HyperStack** for **$0.50/hour** with the following specs:

- 1 GPU
- 28 CPUs
- 58 GB RAM
- 100 GB Disk

<br/>

1. Choose the `Ubuntu Server 22.04 LTS R550 CUDA 12.4` OS Image.
2. Enable **SSH Access** to your VM.
3. Assign a **Public IP Address** to your VM.

<br/>

> [!NOTE]
> Make sure you choose `Ubuntu Server 22.04 LTS R550 CUDA 12.4` instead of the default `Ubuntu Server 22.04 LTS R535 CUDA 12.2` OS Image!

<br/>

> [!CAUTION]
> On Hyperstack, incoming traffic to VMs are blocked by default, so you need to create a Firewall with a rule that allows incoming TCP traffic to port `7860` (or all ports) and apply it to the VM.

<br/>

## SSH into VM

```shell
ssh ubuntu@<PUBLIC_IP_ADDRESS_OF_VM>
```

<br/>

## Install Conda and Python

**References**:

- [Installing Conda on Ubuntu](https://medium.com/@mustafa_kamal/a-step-by-step-guide-to-installing-conda-in-ubuntu-and-creating-an-environment-d4e49a73fc46)
- [Anaconda versions](https://repo.anaconda.com/archive/)

<br/>

```shell
curl -O https://repo.anaconda.com/archive/Anaconda3-2024.10-1-Linux-x86_64.sh
bash Anaconda3-2024.10-1-Linux-x86_64.sh -b -p $HOME/anaconda3
source $HOME/anaconda3/bin/activate
```

> [!NOTE]
> Installing `Anaconda3-2024.10-1` also installs `Python 3.12.7`. I checked that this is the last Anaconda version that comes with `Python 3.12`.

## `Hunyuan3D-2.1`

The commands below are directly from `Hunyuan3D-2.1`'s [README](https://github.com/Tencent-Hunyuan/Hunyuan3D-2.1/blob/main/README.md):

```shell
git clone -b main https://github.com/repulsio/Hunyuan3D-2.1.git --recursive
cd Hunyuan3D-2.1/
git checkout repulsio/hyperstack
```

<br/>

### Conda Virtual Environment

**References**:

- [PyTorch installation](https://pytorch.org/get-started/previous-versions/)

<br/>

```shell
conda create -n hy3d2.1 python=3.10 -y
conda activate hy3d2.1

pip install torch==2.6.0 torchvision==0.21.0 --index-url https://download.pytorch.org/whl/cu124

pip install torch==2.5.1 torchvision==0.20.1 --index-url https://download.pytorch.org/whl/cu124
pip install -r requirements.txt

pip install bpy==4.0 --extra-index-url https://download.blender.org/pypi/

cd hy3dpaint/custom_rasterizer
pip install -e . --no-build-isolation
cd ../..
cd hy3dpaint/DifferentiableRenderer
bash compile_mesh_painter.sh
cd ../..

```

> [!WARNING]
> Because of the error:
> > AttributeError: module 'PIL._webp' has no attribute 'HAVE_WEBPANIM'
> 
> `HAVE_WEBPANIM` was removed in the [commit](https://github.com/python-pillow/Pillow/commit/a3468996c0b7b6df2b685ff21c4f515f5105ff8c) prior to release `11.0.0`, so we have to use the previous version `10.4.0`.

```shell
pip install Pillow==10.4.0
```

> [!IMPORTANT]
> Here is the [link to the diff](https://github.com/Tencent-Hunyuan/Hunyuan3D-2.1/compare/main...repulsio:repulsio/hyperstack) between
>
> - the official `Tencent-Hunyuan/Hunyuan3D-2.1 - main` and
> - my fork `repulsio/Hunyuan3D-2.1 - repulsio/hyperstack`

> [!NOTE]
> The above command creates a Conda virtual environment named `trellis2` and activates it.
> 
> All of the below commands expect that the virtual environment `trellis2` is activated.
> 
> If you need to reactivate it, run `conda activate trellis2`.

> [!WARNING]
> Unfortunately, `Hunyuan3D-2.1` has 2 gated models as dependencies:
> 1. [`facebook/dinov3-vitl16-pretrain-lvd1689m`](https://huggingface.co/facebook/dinov3-vitl16-pretrain-lvd1689m)
> 2. [`briaai/RMBG-2.0`](https://huggingface.co/briaai/RMBG-2.0)
> 
> Because of this, you need to create a HuggingFace account and request access to both of these models. Then, you need to create a HuggingFace **WRITE Access Token** for your account.

```shell
hf auth login

# Enter your WRITE Access Token
# Type and enter `Y`
```

<br/>

## Run Web Demo

```shell
python3 gradio_app.py \
  --model_path tencent/Hunyuan3D-2.1 \
  --subfolder hunyuan3d-dit-v2-1 \
  --texgen_model_path tencent/Hunyuan3D-2.1 \
  --low_vram_mode
```

Open `http://<PUBLIC_IP_ADDRESS_OF_VM>:8080` in your web browser.

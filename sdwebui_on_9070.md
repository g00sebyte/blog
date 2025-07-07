# Running Stable Diffusion via the WebUI on a Radeon 9070 / 9070xt

This guide applies specifically to **debian linux distros.** Given you're reading this I imagine you bought the 9070xt for gaming as well, so to make sure it's properly configured on your system let's set that up first. If you've already been gaming on your system and getting expected perforamnce you can skip the following.

**Pre-requisites:**
- Kernel version 6.12 or greater
- Mesa 25 or higher
- [amdgpu linux firmware](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/commit/t)

ref: https://forums.linuxmint.com/viewtopic.php?t=442510

## Stable Diffusion WebUI Setup

If you've already setup your 9070 / 9070xt and are achieving expected performance, or if you have another AMD card (RDNA2 or newer) and it's properly configured then these steps should work for you too.

**1. Ensure system is up to date:**
```
sudo apt update
sudo apt upgrade
```

**2. Install ROCM**
```
wget https://repo.radeon.com/amdgpu-install/6.4.1/ubuntu/noble/amdgpu-install_6.4.60401-1_all.deb
sudo apt install ./amdgpu-install_6.4.60401-1_all.deb
sudo apt update
sudo apt install python3-setuptools python3-wheel
sudo usermod -a -G render,video $LOGNAME # Add the current user to the render and video groups
sudo apt install rocm
```
You might also want rocm-smi:
```
sudo apt install rocm-smi
```
This allows you to monitor your GPU via:
```
watch -n05 rocm-smi
```
I typically run this in another terminal while I run stable diffusion.

**3. Install Git & Clone SD-WebUI**
```
sudo apt install git
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
cd stable-diffusion-webui/
```

**4. Edit the file "webui-user.sh"**

You'll want to add the below arguments:
`export COMMANDLINE_ARGS="--upcast-sampling --medvram"`

Note: --medvram isn't necessary but subjectively seems to help with pytorch's bad habit of over-allocating memory

ii) Uncomment and set the python command as below:

python_cmd="python3.10"

Note: the use of python3.10 specifically comes from SD WebUI itself, refer to: https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Install-and-Run-on-AMD-GPUs#automatic-installation

iii) Add the following to the end of the file:

```
export TORCH_COMMAND='pip install torch torchvision --extra-index-url https://download.pytorch.org/whl/rocm6.3'
export PYTORCH_HIP_ALLOC_CONF='expandable_segments:True'
export MIGRAPHX_TRACE_EVAL=1
```

**5. Run `./webui.sh`**

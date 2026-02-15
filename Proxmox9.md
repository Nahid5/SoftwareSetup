# Setting up GPU Passthrough
1. Files to edit

/etc/default/grub
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt pcie_acs_override=downstream,multifunction nofb nomodeset video=vesafb:off,efifb:off"
```

```
echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf

```

/etc/modprobe/pve-blacklist.conf
```
blacklist nvidia
blacklist nvidiafb
blacklist nouveau
blacklist nvidia_drm
blacklist radeon
blacklist amdgpu
blacklist snd_hda_intel
blacklist snd_hda_codec_hdmi
blacklist i915
blacklist xe
```

/etc/modprobe.d/blacklist.conf
```
blacklist nvidia
blacklist nvidiafb
blacklist nouveau
blacklist nvidia_drm
blacklist radeon
blacklist amdgpu
blacklist snd_hda_intel
blacklist snd_hda_codec_hdmi
blacklist i915
blacklist xe
```

cat /etc/modprobe.d/kvm.conf
```
options kvm ignore_msrs=1
```

To get ids of hardware to blacklist use this:
```
lspci -nn | grep -E "VGA|Audio"
lspci -n -s ID
EX:
lspci -n -s 03:00
```

Use the ID here: `/etc/modprobe.d/vfio.conf`
```
options vfio-pci ids=XX:XX,XX:XX,XX:XX,XX:XX disable_vga=1
softdep radeon pre: vfio-pci
softdep amdgpu pre: vfio-pci
softdep nouveau pre: vfio-pci
softdep nvidia pre: vfio-pci
softdep nvidiafb pre: vfio-pci
softdep nvidia_drm pre: vfio-pci
softdep drm pre: vfio-pci
softdep snd_hda_intel pre: vfio-pci
softdep snd_hda_codec_hdmi pre: vfio-pci
softdep i915 pre: vfio-pci
```

2. Configure VM
After the configs are set on the PVE server, then add the hardware from:
- VM -> Hardware -> Add PCI Device
- RAW Device -> Select VGA Device, All Functions, PCI-Express selected (From Advanced settings)

3. Install PCI driver on VM:
- https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md

4. Prevent Crashing
- VM -> Hardware -> Display, set to none. Do this after setting up the VM for remote connectivity some other way. 

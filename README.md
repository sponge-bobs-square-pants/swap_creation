# Linux Swap File Setup Guide

## Check Current Status

```bash
# Check current memory and swap usage
free -h

# Check existing swap devices
swapon --show

# List swap in /proc
cat /proc/swaps
```

## Check Filesystem Type

```bash
# Check filesystem type (important for btrfs)
df -T /
```

## Create Swap File

### For Regular Filesystems (ext4, xfs, etc.)

```bash
# Create swap file (replace 2G with desired size: 1G, 4G, etc.)
sudo fallocate -l 2G /swapfile

# Set permissions
sudo chmod 600 /swapfile

# Set up as swap
sudo mkswap /swapfile

# Enable swap
sudo swapon /swapfile
```

### For BTRFS Filesystem (Required Method)

```bash
# Create empty file first
sudo touch /swapfile

# Set NOCOW attribute BEFORE writing data (critical for btrfs)
sudo chattr +C /swapfile

# Write data (replace count=2097152 for different sizes)
# 1GB: count=1048576, 2GB: count=2097152, 4GB: count=4194304
sudo dd if=/dev/zero of=/swapfile bs=1024 count=2097152

# Set permissions
sudo chmod 600 /swapfile

# Verify NOCOW attribute is set
lsattr /swapfile
# Should show: ---------------C------

# Set up as swap
sudo mkswap /swapfile

# Enable swap
sudo swapon /swapfile
```

## Verify Swap is Working

```bash
# Check memory and swap usage
free -h

# Show all swap devices
swapon --show

# Detailed swap information
cat /proc/swaps
```

## Make Swap Permanent

```bash
# Add to fstab for persistence across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Verify fstab entry
tail -1 /etc/fstab
```

## Swap Size Recommendations

| RAM Size | Recommended Swap |
|----------|------------------|
| 1GB      | 2-4GB           |
| 2GB      | 2-4GB           |
| 4GB      | 2-4GB           |
| 8GB+     | 2-4GB           |

## Optional: Optimize Swap Settings

```bash
# Set swappiness (how aggressively system uses swap)
# Lower = use swap less, Higher = use swap more
sudo sysctl vm.swappiness=10

# Make swappiness setting permanent
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
```

## Managing Swap

```bash
# Disable swap temporarily
sudo swapoff /swapfile

# Re-enable swap
sudo swapon /swapfile

# Remove swap permanently
sudo swapoff /swapfile
sudo sed -i '/swapfile/d' /etc/fstab
sudo rm /swapfile
```

## Troubleshooting

### "Invalid argument" error on btrfs
- **Cause**: Copy-on-write (COW) must be disabled for swap files on btrfs
- **Solution**: Use the BTRFS method above with `chattr +C` before writing data

### Check for errors
```bash
# View recent kernel messages
dmesg | tail -10

# Check available disk space
df -h /
```

## DD Command Size Reference

```bash
# 1GB swap
sudo dd if=/dev/zero of=/swapfile bs=1024 count=1048576

# 2GB swap  
sudo dd if=/dev/zero of=/swapfile bs=1024 count=2097152

# 4GB swap
sudo dd if=/dev/zero of=/swapfile bs=1024 count=4194304

# 8GB swap
sudo dd if=/dev/zero of=/swapfile bs=1024 count=8388608
```

# Xubuntu GUI Setup

## Install Xubuntu Desktop
```bash
sudo apt install xubuntu-desktop -y
```

Choose:
```text
lightdm
```

## Reboot
```bash
sudo reboot
```

## Restart GUI
```bash
sudo systemctl restart lightdm
```

## Set GUI Mode
```bash
sudo systemctl set-default graphical.target
```

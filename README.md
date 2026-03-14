# COOLCOMMANDS
kewl linux stuff
>[!TIP]
>For DD'ing iso's: 
>```bash
> dd if=image.iso of=/dev/sdX bs=4M status=progress oflag=direct conv=fdatasync
>```
>```bash
>cmp image.iso /dev/sdX
>```
><br>
Get rid of Rocky10 bloatware:
>[!TIP]
>```bash
>sudo dnf remove gdm gnome-shell gnome-session gnome-control-center gnome-settings-daemon gnome-software gnome-initial-setup gnome-tour nautilus papers ptyxis loupe baobab firefox cups cups-client cups-browsed cups-filters cups-filters-driverless cups-pk-helper foomatic foomatic-db foomatic-db-ppds gutenprint gutenprint-cups hplip-common hplip-libs sane-backends sane-backends-libs sane-airscan libsane-airscan ipp-usb pipewire pipewire-alsa pipewire-gstreamer pipewire-jack-audio-connection-kit pipewire-pulseaudio pipewire-utils wireplumber pulseaudio-libs bluez bluez-obexd gnome-bluetooth ModemManager mobile-broadband-provider-info NetworkManager-bluetooth NetworkManager-wwan wpa_supplicant iwflatpak flatpak-libs flatpak-session-helper gnome-software PackageKit PackageKit-command-not-found PackageKit-gstreamer-plugin PackageKit-gtk3-module avahi avahi-tools pcp-zeroconf fprintd fprintd-pam pcsc-lite pcsc-lite-ccid gnome-keyring gnome-keyring-pam pinentry-gnome3
> ```
<br>
Get rid of uneeded fw, use with caution!
>[!TIP]
>```bash
> sudo dnf remove -y amd-gpu-firmware atheros-firmware brcmfmac-firmware iwlwifi-dvm-firmware iwlwifi-mvm-firmware mt7xxx-firmware nxpwireless-firmware tiwilink-firmware realtek-firmware netronome-firmware nvidia-gpu-firmware cirrus-audio-firmware
>```

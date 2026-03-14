# COOLCOMMANDS
kewl linux stuff

---

## DD an ISO to a USB drive

> [!TIP]
> For **dd'ing ISO images**:
>
> ```bash
> dd if=image.iso of=/dev/sdX bs=4M status=progress oflag=direct conv=fdatasync
> ```
>
> Verify the write:
>
> ```bash
> cmp image.iso /dev/sdX
> ```

---

## Remove Rocky Linux 10 desktop bloat

> [!TIP]
> Remove unnecessary desktop packages:
>
> ```bash
> sudo dnf remove \
> gdm gnome-shell gnome-session gnome-control-center \
> gnome-settings-daemon gnome-software gnome-initial-setup \
> gnome-tour nautilus papers ptyxis loupe baobab firefox \
> cups cups-client cups-browsed cups-filters cups-filters-driverless \
> cups-pk-helper foomatic foomatic-db foomatic-db-ppds \
> gutenprint gutenprint-cups hplip-common hplip-libs \
> sane-backends sane-backends-libs sane-airscan libsane-airscan ipp-usb \
> pipewire pipewire-alsa pipewire-gstreamer \
> pipewire-jack-audio-connection-kit pipewire-pulseaudio pipewire-utils \
> wireplumber pulseaudio-libs \
> bluez bluez-obexd gnome-bluetooth ModemManager \
> mobile-broadband-provider-info NetworkManager-bluetooth \
> NetworkManager-wwan wpa_supplicant iw \
> flatpak flatpak-libs flatpak-session-helper \
> PackageKit PackageKit-command-not-found \
> PackageKit-gstreamer-plugin PackageKit-gtk3-module \
> avahi avahi-tools pcp-zeroconf \
> fprintd fprintd-pam pcsc-lite pcsc-lite-ccid \
> gnome-keyring gnome-keyring-pam pinentry-gnome3
> ```

---

## Remove unused firmware (optional)

> [!CAUTION]
> Removing firmware packages can break hardware support.  
> Only do this if you have confirmed the hardware does not need them.
>
> ```bash
> sudo dnf remove -y \
> amd-gpu-firmware \
> atheros-firmware \
> brcmfmac-firmware \
> iwlwifi-dvm-firmware \
> iwlwifi-mvm-firmware \
> mt7xxx-firmware \
> nxpwireless-firmware \
> tiwilink-firmware \
> realtek-firmware \
> netronome-firmware \
> nvidia-gpu-firmware \
> cirrus-audio-firmware
> ```

## ROCKY10U1 unneeded services
>[!TIP]
> ```bash
>systemctl disable --now colord
>systemctl disable --now upower
>systemctl disable --now udisks2
>systemctl disable --now alsa-state
>systemctl disable --now pmcd
>systemctl disable --now pmie
>systemctl disable --now pmlogger
>systemctl disable --now pmie_farm
>systemctl disable --now pmlogger_farm
>systemctl disable --now atd
>systemctl disable --now fwupd
>systemctl disable --now libstoragemgmt
> ```

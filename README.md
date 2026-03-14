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

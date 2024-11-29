# Migració d'un servidor tradicional a un servidor amb RAID

En aquest laboratori aprendrem a migrar un servidor tradicional a un servidor amb RAID.

## Notes

> 📝 **Nota 1**
>
> Es possible que observeu un missatge com el següent quan realitzeu el laboratori:
> mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
> Podeu ignorar aquest missatge ja que no afecta el funcionament del laboratori. O si voleu podeu executar la comanda `systemctl daemon-reload` per assegurar-vos que el sistema ha recarregat la configuració.

## Preparació de l'entorn

Assumirem una màquina virtual amb AlmaLinux on tenim un disc dur principal de 20GB particionat de la següent manera:

- `/boot` de 600MB amb el sistema de fitxers `xfs`.
- `/boot/efi` de 1024MB amb el sistema de fitxers `vfat`.
- `swap` de 2GB.
- `/` de 16GB amb el sistema de fitxers `xfs`.

![Estat inicial `lsblk`](figures/migration/estat-inicial.png)

Instal·leu els paquet següents que necessitarem per aquest laboratori:

1. `mdadm`: Utilitzada per gestionar dispositius RAID.

    ```bash
    dnf install mdadm -y
    ```

2. `rsync`: Utilitzada per copiar el contingut de les particions.

    ```bash
    dnf install rsync -y
    ```

## Disseny de la nova arquitectura

Utiltizarem 2 discs secundaris de 20GB cadascun per crear la següent arquitectura:

- **RAID 1** amb `/boot` i `/boot/efi`.
- **RAID 1** amb `swap`.
- **RAID 1** amb `/`.

## Preparació dels discs

1. Creació de la taula de particions i de les particions als discs secundaris.

    - `/boot/efi`:

        ```bash
        echo -e "g\nn\np\n\n\n+600M\nt\n29\nw" | fdisk /dev/nvme0n2
        echo -e "g\nn\np\n\n\n+600M\nt\n29\nw" | fdisk /dev/nvme0n3
        ```

    - `boot`:

        ```bash
        echo -e "n\np\n\n\n+1024M\nt\n2\n29\nw" | fdisk /dev/nvme0n2
        echo -e "n\np\n\n\n+1024M\nt\n2\n29\nw" | fdisk /dev/nvme0n3
        ```

    - `swap`:

        ```bash
        echo -e "n\np\n\n\n+2G\nt\n3\n29\nw" | fdisk /dev/nvme0n2
        echo -e "n\np\n\n\n+2G\nt\n3\n29\nw" | fdisk /dev/nvme0n3
        ```

    - `/`:

        ```bash
        echo -e "n\np\n\n\n+16G\nt\n4\n29\nw" | fdisk /dev/nvme0n2
        echo -e "n\np\n\n\n+16G\nt\n4\n29\nw" | fdisk /dev/nvme0n3
        ```

2. Comproveu que les particions s'han creat correctament amb `lsblk`.

    ![Estat de les particions `lsblk`](figures/migration/particions.png)

## RAID 1 amb `/boot/efi`

1. Creació del RAID 1 amb `/boot/efi`.

    ```bash
    mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 \
    --metadata=1.0 /dev/nvme0n2p1 /dev/nvme0n3p1
    ```

2. Creació del sistema de fitxers `fat32`.

    ```bash
    mkfs.vfat /dev/md0
    ```

3. Crear un punt de muntatge temporal i muntar el sistema de fitxers.

    ```bash
    mkdir /tmp/raid1
    mount /dev/md0 /tmp/raid1
    ```

4. Copiar el contingut de `/boot/efi` al RAID 1.

    ```bash
    rsync -avx /boot/efi/ /tmp/raid1
    ```

5. Desmuntar la RAID:

    ```bash
    umount /tmp/raid1
    ```

6. Seleccionar el UUID del RAID 1:

    ```bash
    EFI_RAID_UUID=$(blkid -s UUID -o value /dev/md0)
    ```

7. Actualitzar `/etc/fstab` amb el nou UUID.

    ```bash
    CURRENT_BOOT_UUID=$(blkid -s UUID -o value /dev/nvme0n1p1)
    sed -i "s|UUID=$CURRENT_BOOT_UUID|UUID=$EFI_RAID_UUID |" /etc/fstab
    ```

## RAID 1 amb `boot`

Repetiu els passos anteriors per crear el RAID 1 anomenta (**/dev/md1**) amb `/boot`. Utiltizant les particions `/dev/nvme0n2p2` i `/dev/nvme0n3p2`.

1. Creació del RAID 1 amb `/boot`.

    ```bash
    mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 \
    --metadata=1.0 /dev/nvme0n2p2 /dev/nvme0n3p2
    ```

2. Creació del sistema de fitxers `xfs`.

    ```bash
    mkfs.xfs /dev/md1
    ```

3. Crear un punt de muntatge temporal i muntar el sistema de fitxers.

    ```bash
    mkdir /tmp/raid2
    mount /dev/md1 /tmp/raid2
    ```

4. Copiar el contingut de `/boot` al RAID 1.

    ```bash
    rsync -avx /boot/ /tmp/raid2
    ```

5. Desmuntar la RAID:

    ```bash
    umount /tmp/raid2
    ```

6. Seleccionar el UUID del RAID 1:

    ```bash
    BOOT_RAID_UUID=$(blkid -s UUID -o value /dev/md1)
    ```

7. Actualitzar `/etc/fstab` amb el nou UUID.

    ```bash
    CURRENT_BOOT_UUID=$(blkid -s UUID -o value /dev/nvme0n1p2)
    sed -i "s|UUID=$CURRENT_BOOT_UUID|UUID=$BOOT_RAID_UUID |" /etc/fstab
    ```

## RAID 1 amb `swap`

1. Creació del RAID 1 amb `swap`.

    ```bash
    mdadm --create --verbose /dev/md2 --level=1 --raid-devices=2 \
    /dev/nvme0n2p3 /dev/nvme0n3p3
    ```

2. Creació del sistema de fitxers `swap`.

    ```bash
    mkswap /dev/md2
    ```

3. Activar el sistema de fitxers `swap`.

    ```bash
    swapon /dev/md2
    ```

4. Actualitzar `/etc/fstab` amb el nou UUID.

    ```bash
    CURRENT_SWAP_UUID=$(blkid -s UUID -o value /dev/nvme0n1p3)
    SWAP_RAID_UUID=$(blkid -s UUID -o value /dev/md2)
    sed -i "s|UUID=$CURRENT_SWAP_UUID|UUID=$SWAP_RAID_UUID |" /etc/fstab
    ```

5. Compproveu que la partició swap s'ha activat correctament.

    ```bash
    swapon --show
    ```

## RAID 1 amb `/`

1. Creació del RAID 1 amb `/`.

    ```bash
    mdadm --create --verbose /dev/md3 --level=1 --raid-devices=2 \
    /dev/nvme0n2p4 /dev/nvme0n3p4
    ```

2. Creació del sistema de fitxers `xfs`.

    ```bash
    mkfs.xfs /dev/md3
    ```

3. Crear un punt de muntatge temporal i muntar el sistema de fitxers.

    ```bash
    mkdir /tmp/raid3
    mount /dev/md3 /tmp/raid3
    ```

4. Noms de dispositius RAID persistents.

    ```bash
    mkdir -p /etc/mdadm
    mdadm --detail --scan > /etc/mdadm/mdadm.conf
    ```

5. Seleccionar el UUID del RAID 1:

    ```bash
    ROOT_RAID_UUID=$(blkid -s UUID -o value /dev/md3)
    ```

6. Actualitzar `/etc/fstab` amb el nou UUID.

    ```bash
    CURRENT_ROOT_UUID=$(blkid -s UUID -o value /dev/nvme0n1p4)
    sed -i "s|UUID=$CURRENT_ROOT_UUID|UUID=$ROOT_RAID_UUID |" /etc/fstab
    ```

7. Copiar el contingut de `/` al RAID 1.

    ```bash
    cp -ax / /tmp/raid3
    ```

## Configuració del GRUB

1. Actualitzar la configuració del GRUB.

    ```bash
    vi /etc/default/grub
    ```

    Teniu dos opcions per configurar el GRUB, com els noms dels dispositius no canviaran, ja que els hem fet persistent, podeu utilitzar el nom del dispositiu `/dev/mdX` o el UUID del RAID.

    ```bash
    GRUB_CMDLINE_LINUX="root=/dev/md3 rd.auto ..."
    GRUB_ENABLE_BLSCFG=false
    GRUB_CMDLINE_LINUX_DEFAULT=""
    ```

    o bé utilitzant el UUID del RAID.

    ```bash
    GRUB_CMDLINE_LINUX="root=UUID=<UUID> rd.auto ..."
    GRUB_ENABLE_BLSCFG=false
    GRUB_CMDLINE_LINUX_DEFAULT=""
    # on <UUID> és el UUID del RAID 1 amb /.
    # rd.auto: Per carregar automàticament els mòduls RAID.
    # GRUB_ENABLE_BLSCFG=false: Per desactivar el suport de BLS.
    # GRUB_CMDLINE_LINUX_DEFAULT="": Per desactivar les opcions per defecte.
    ```

2. Copiar el GRUB al nou disc.

    ```bash
    cp /etc/default/grub /tmp/raid3/etc/default/grub
    ```

3. Instal·lar el GRUB a cada disc secundari.

    ```bash
    efibootmgr --create --disk /dev/nvme0n2 --part 1 --label "almalinux-mirror-01" --loader "\EFI\almalinux\grubaa64.efi"
    efibootmgr --create --disk /dev/nvme0n3 --part 1 --label "almalinux-mirror-02" --loader "\EFI\almalinux\grubaa64.efi"
    ```

4. Actualitzar la configuració del GRUB.

    ```bash
    grub2-mkconfig -o /boot/efi/EFI/almalinux/grub.cfg
    ```

5. Actualitzarem la `initramfs` per incloure els mòduls RAID.

    ```bash
    dracut -f
    ```

6. Reiniciar el sistema.

    ```bash
    reboot
    ```

> **Nota**: Aquesta configuració no és del tot correcta, i veureu que el raid de la partició del sistema `/` no es munta i es segueix utilitzant la partició original. Tot i això, si en el grub indiquem `root=/dev/md3` i `rd.auto`, el sistema arrenca correctament. Per tant, deures investigar com fer que el sistema munti el raid de la partició `/` correctament.
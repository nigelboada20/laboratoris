# Tolerància a fallades

## Preparació de l'entorn

En aquest laboratori podeu utilitzar la mateixa màquina virtual que en el laboratori anterior. Ens centrarem en la tolerància a fallades dels sistemes RAID0 i RAID1.

## Simulant una fallada en un disc del RAID1

1. Crearem dades en el sistema de fitxers del RAID1.

    ```bash
    fallocate -l 100M /mnt/md1/testfile
    ```

2. Simularem una fallada en un dels discos del RAID1. Utilitzant l'argument `--set-faulty` de `mdadm` podem simular una fallada en un dels discos.

    ```bash
    mdadm --manage /dev/md1 --set-faulty /dev/nvme0n4
    ```

3. Comproveu l'estat del RAID1 amb la comanda `mdadm --detail /dev/md1`.

    > 📝 **Nota**
    >
    > Alternativament, podeu utilitzar la comanda `cat /proc/mdstat` per comprovar l'estat dels vostres dispositius RAID.

4. Comproveu integritat de les dades:

    ```bash
    ls -la /mnt/md1
    ```

    Si tot ha anat bé, les dades haurien de ser accessibles.

## Simulant una fallada en un disc del RAID0

1. Crearem dades en el sistema de fitxers del RAID0.

    ```bash
    fallocate -l 100M /mnt/md0/testfile
    ```

2. Utilitzarem la comanda `hexdump` per escriure dades aleatòries al disc i corrompre'l.

    ```bash
    hexdump -C /dev/nvme0n2 | head -n 10
    ```

3. Desmuntarem el sistema de fitxers i el muntarem de nou.

    ```bash
    umount /mnt/md0
    mount /dev/md0 /mnt/md0
    ```

4. El sistema us hauria de mostrar un error semblant a aquest:

    ```text
    mount: /mnt/md0: can't read superblock on /dev/md0.
       dmesg(1) may have more information after failed mount system call.
    ```

    En aquest cas, el sistema no pot muntar el sistema de fitxers ja que el disc ha estat corromput.

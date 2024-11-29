# Comparant el rendiment de diferents nivells de RAIDs

En aquest laboratori, compararem el rendiment de diferents nivells de RAID utilitzant l'eina de benchmark `fio`. En concret, compararem el rendiment del **RAID 0** i del **RAID 1** en operacions d'escriptura i de lectura. Aquesta anàlisi ens permetrà comprendre les diferències de rendiment entre els dos tipus de RAID i determinar quin tipus de RAID és més adequat per a les nostres necessitats.

## Configuració de l'entorn

En aquest laboratori utiltizarem una màquina virtual amb Debian 12 i 4 discos secundaris de 5GB cadascun. Aquests discos secundaris es configuraran en dos nivells de RAID diferents: **RAID0** i **RAID1**.

> 📝 **Nota**
>
> Si no voleu utilitzar 4 discs podeu utiltizar 2 disc amb diferents particions per a crear les RAID. El més important és que la RAID0 i la RAID1 tinguin el mateix nombre de discs, capacitat i tipus per poder fer una comparació justa.

## Creació de les raids

1. Creació de raid0:

    ```bash
    mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/nvme0n2 /dev/nvme0n3
    ```

2. Creació del sistema de fitxers:

    ```bash
    mkfs.ext4 /dev/md0
    ```

3. Muntatge del sistema de fitxers:

    ```bash
    mkdir /mnt/md0
    mount /dev/md0 /mnt/md0
    ```

4. Creació de raid1:

    ```bash
    mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/nvme0n4 /dev/nvme0n5
    ```

5. Creació del sistema de fitxers:

    ```bash
    mkfs.ext4 /dev/md1
    ```

6. Muntatge del sistema de fitxers:

    ```bash
    mkdir /mnt/md1
    mount /dev/md1 /mnt/md1
    ```

## Instal·lant l'eina de benchmark `fio`

La eina `fio` ens permetrà realitzar diferents experriments per tal de comparar diferents operacions d'entrada/sortida amb diferents configuracions de blocs.

```bash
apt install fio -y
```

## Proves de rendiment

1. Test de rendiment del RAID0 en oepracions d'escriptura:

    ```bash
    fio --name=write-raid1 --ioengine=libaio --iodepth=32 --rw=write \
    --bs=4k --size=500M --numjobs=4 --runtime=120 --time_based \
    --ramp_time=15 --group_reporting --filename=/dev/raid1 \
    --output-format=json --output=/tmp/write-raid1.json
    ```

2. Test de rendiment del RAID0 en operacions de lectura:

    ```bash
    fio --name=read-raid0 --ioengine=libaio --iodepth=32 --rw=read \
    --bs=4k --size=500M --numjobs=4 --runtime=120 --time_based \
    --ramp_time=15 --group_reporting --filename=/dev/raid0 \
    --output-format=json --output=/tmp/read-raid0.json
    ```

3. Test de rendiment del RAID1 en operacions d'escriptura:

    ```bash
    fio --name=write-raid1 --ioengine=libaio --iodepth=32 --rw=write \
    --bs=4k --size=500M --numjobs=4 --runtime=120 --time_based \
    --ramp_time=15 --group_reporting --filename=/dev/raid1 \
    --output-format=json --output=/tmp/write-raid1.json
    ```

4. Test de rendiment del RAID1 en operacions de lectura:

    ```bash
    fio --name=read-raid1 --ioengine=libaio --iodepth=32 --rw=read \
    --bs=4k --size=500M --numjobs=4 --runtime=120 --time_based \
    --ramp_time=15 --group_reporting --filename=/dev/raid1 \
    --output-format=json --output=/tmp/read-raid1.json
    ```

## Anàlisi de resultats

Per analitzar els resultats obtinguts, ens centrarem en els següents paràmetres:

- **Bandwidth**: La velocitat de transferència de dades en bytes per segon.
- **IOPS**: El nombre d’operacions d’entrada/sortida per segon.
- **Latència**: El temps que triga el sistema a respondre a una petició
d’entrada/sortida.
- **Total IO**: El nombre total d’operacions d’entrada/sortida.

| Op. | RAID | Bandwidth (GB/s)          | IOPS          | Latència(µs)           | Total IO (GB)          |
|-----|------|---------------------------|---------------|------------------------|------------------------|
| W   |  0   | 10.58                     | 2.65e+06      | 48.22                  | 317.6                  |
| W   |  1   | 7.93                      | 1.98e+06      | 64.40                  | 238.0                  |
| R   |  0   | 18.76                     | 4.69e+06      | 27.16                  | 562.98                 |
| R   |  1   | 18.82                     | 4.71e+06      | 27.08                  | 564.6                  |

La taula mostra un resum dels resultats obtinguts de les proves de rendiment del **RAID 0** i del **RAID 1**. Aquests resultats corroboren les diferències de rendiment entre els dos tipus de RAID. En primer lloc, si ens centrem en el **Bandwidth**, podem observar un rendiment similar en les operacions de lectura amb unes velocitats de *18.76GB/s i 18.82GB/s* respectivament. En canvi, el **RAID 0** té un rendiment superior en les operacions d'escriptura amb una velocitat de 10.59GB/s, mentre que el RAID 1 té una velocitat de *7.93GB/s*. Això es deu al fet que en el **RAID 0** les dades es distribueixen entre els dos discs, mentre que en el **RAID 1** les dades s'han de copiar en tots dos discs. En segon lloc, si analitzem les variables de **Latency** i **IOPS** observem un rendiment similar en les operacions de lectura, amb una latència de *27.16µs i 27.08µs* respectivament. En canvi, en les operacions d'escriptura, el **RAID 0** té una latència de *48.22µs i 64.40µs* en el **RAID 1**. Finalment, en termes de Total IO, el **RAID 0** té un rendiment superior amb *317.69GB* en les operacions d'escriptura, mentre que el RAID 1 té un rendiment de *238.01GB*.

> 🚀 Conclusió:
>
> Aquesta anàlisi demostra que el **RAID 0** ofereix un rendiment superior en termes de velocitat d'escriptura, mentre que el **RAID 1** proporciona una major seguretat de les dades a costa d'una velocitat d'escriptura més lenta. Per tant, la selecció del tipus de RAID depèn de les necessitats específiques de l'usuari, prioritzant la velocitat o la seguretat de les dades.

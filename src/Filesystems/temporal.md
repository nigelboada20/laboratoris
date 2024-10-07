# Sistemes de fitxers temporals

En aquest laboratori, compararem el rendiment dels sistemes de fitxers ext4, xfs i tmpfs en un entorn de còmput científic. Els sistemes de fitxers temporals, com tmpfs, poden millorar significativament el rendiment d’operacions d’entrada/sortida intensives.

## Context

Assumeix l'administració d'un servidor de còmput científic que necessita realitzar moltes operacions d'entrada/sortida. En aquest escenari, el rendiment del sistema de fitxers pot ser crític per a l'eficiència del sistema. En aquest laboratori volem corrobarar la següent hipòtesi: els sistemes de fitxers temporals, com tmpfs, poden millorar significativament el rendiment d'operacions d'entrada/sortida intensives per aquest tipus de càrregues de treball.

## Objectius

- Comprendre el rendiment dels sistemes de fitxers ext4, xfs i tmpfs.
- Comparar el rendiment dels sistemes de fitxers ext4, xfs i tmpfs en un entorn de còmput científic.
- Optimitzar el rendiment del sistema de fitxers utilitzant un sistema de fitxers temporal.

## Preparació de l'entorn

1. Creeu un nova màquina virtual amb Debian 12. Podeu utiltizar una configuració per defecte amb 2GB de RAM i 20GB d'espai en el disc principal.
2. Creeu un disc secundari de 20GB per crear els sistemes de fitxers ext4, xfs i tmpfs.
3. Particioneu el disc secundari amb 3 particions: ext4, xfs i tmpfs cada una amb 5GB d'espai.
4. Munteu les particions a les rutes `/mnt/ext4`, `/mnt/xfs` i `/mnt/tmpfs` respectivament.

Per crear el sistema de fitxers temporal tmpfs, podeu utilitzar la comanda següent:

```bash
mount -t tmpfs -o size=5G tmpfs /mnt/tmpfs
```

Per veure el sistema de fitxers muntat, podeu utilitzar la comanda següent:

```bash
df -h
```

![Estat inicial dels sistemes de fitxers](img/temporal/df-h.png)

## Preparació de l'experiment

Per simular el nostre experiment, crearem un script de Python que generi un fitxer gran i realitzi moltes operacions d'escriptura aleatòria. Aquest script pot ser executat en un servidor de còmput per provar el rendiment del sistema de fitxers. Com a parametres d'entrada podem especificar la ruta del fitxer, la mida del fitxer, el nombre d'operacions d'escriptura aleatòria i la mida del bloc d'escriptura.

```python
import os
import random
import time
import argparse

def create_large_file(file_path, size_in_mb):
    with open(file_path, 'wb') as f:
        f.write(os.urandom(size_in_mb * 1024 * 1024))

def random_write(file_path, num_operations, block_size):
    with open(file_path, 'r+b') as f:
        for _ in range(num_operations):
            offset = random.randint(0, os.path.getsize(file_path) - block_size)
            f.seek(offset)
            f.write(os.urandom(block_size))

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Simulació de càlcul intensiu d'entrada/sortida")
    parser.add_argument('--file_path', type=str, default="/mnt/tmpfs/large_file.bin", help="Camí del fitxer a crear")
    parser.add_argument('--log_file', type=str, default="/mnt/tmpfs/experiment_log.txt", help="Camí del fitxer de registre")
    parser.add_argument('--size_in_mb', type=int, default=1024, help="Mida del fitxer en MB")
    parser.add_argument('--num_operations', type=int, default=10000, help="Nombre d'operacions d'escriptura aleatòria")
    parser.add_argument('--block_size', type=int, default=4096, help="Mida del bloc en bytes")

    args = parser.parse_args()

    start_time = time.time()
    create_large_file(args.file_path, args.size_in_mb)
    random_write(args.file_path, args.num_operations, args.block_size)
    end_time = time.time()

    total_time = end_time - start_time
    log_message = f"Experiment completat en {total_time} segons"

    print(log_message)
```

## Execució de l'experiment

Un cop analitzat el codi, podem executar l'experiment per comparar el rendiment dels sistemes de fitxers ext4, xfs i tmpfs.

1. Executem l'experiment amb el sistema de fitxers ext4:

```bash
python3 simulate_io_intensive.py --file_path /mnt/ext4/large_file.bin --size_in_mb 1024 --num_operations 10000 --block_size 4096
```

2. Executem l'experiment amb el sistema de fitxers xfs:

```bash
python3 simulate_io_intensive.py --file_path /mnt/xfs/large_file.bin --size_in_mb 1024 --num_operations 10000 --block_size 4096
```

3. Executem l'experiment amb el sistema de fitxers tmpfs:

```bash
python3 simulate_io_intensive.py --file_path /mnt/tmpfs/large_file.bin --size_in_mb 1024 --num_operations 10000 --block_size 4096
```

Ara realitzarem l'experiment 10 vegades per obtenir una mitjana del temps d'execució per a cada sistema de fitxers.

- **Sistema de fitxers**: ext4

```bash
for i in {1..10}
do
    python3 simulate_io_intensive.py --file_path /mnt/ext4/large_file.bin --size_in_mb 1024 --num_operations 10000 --block_size 4096
done
```

| Temps d'execució (s) | Iteració |
|----------------------|----------|
| 3.04                 | 1        |
| 2.58                 | 2        |
| 2.62                 | 3        |
| 3.28                 | 4        |
| 2.68                 | 5        |
| 2.98                 | 6        |
| 2.66                 | 7        |
| 3.34                 | 8        |
| 2.79                 | 9        |
| 2.83                 | 10       |
| **Mitjana**          | **2.88** |

- **Sistema de fitxers**: xfs

```bash
for i in {1..10}
do
    python3 simulate_io_intensive.py --file_path /mnt/xfs/large_file.bin --size_in_mb 1024 --num_operations 10000 --block_size 4096
done
```

| Temps d'execució (s) | Iteració |
|----------------------|----------|
| 2.34                 | 1        |
| 2.42                 | 2        |
| 2.41                 | 3        |
| 2.41                 | 4        |
| 2.40                 | 5        |
| 2.39                 | 6        |
| 2.41                 | 7        |
| 2.44                 | 8        |
| 2.49                 | 9        |
| 2.46                 | 10       |
| **Mitjana**          | **2.42** |

- **Sistema de fitxers**: tmpfs

```bash
for i in {1..10}
do
    python3 simulate_io_intensive.py --file_path /mnt/tmpfs/large_file.bin --size_in_mb 1024 --num_operations 10000 --block_size 4096
done
```

| Temps d'execució (s) | Iteració |
|----------------------|----------|
| 2.33                 | 1        |
| 2.29                 | 2        |
| 2.28                 | 3        |
| 2.29                 | 4        |
| 2.32                 | 5        |
| 2.32                 | 6        |
| 2.30                 | 7        |
| 2.33                 | 8        |
| 2.31                 | 9        |
| 2.29                 | 10       |
| **Mitjana**          | **2.31** |

Observeu com el sistema de fitxers tmpfs té un rendiment significativament millor que els sistemes de fitxers ext4 i xfs en aquest escenari. Això es deu al fet que tmpfs emmagatzema les dades a la memòria RAM en lloc de l'emmagatzematge en disc, el que permet un accés més ràpid a les dades. Ara bé, cal tenir en compte que les dades emmagatzemades a tmpfs es perden quan el sistema es reinicia.


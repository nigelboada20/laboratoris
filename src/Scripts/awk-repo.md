# Repositori d'exercicis

Aquesta secció conté els exercicis realitzats pels estudiants de l'assignatura d'Administració i Manteniment de Sistemes i Aplicacions (AMSA).

## Exercicis


### Bàsics
Exercici 1: Filtrar Pokémon amb punts de defensa alts
Descripció:
Volem filtrar tots els Pokémon que tenen més de 120 punts de defensa (columna 7) i mostrar només el nom del Pokémon (columna 2) juntament amb els seus punts de defensa.

Instruccions:
Utilitza AWK per processar el fitxer pokemon.csv i mostrar només aquells Pokémon que tinguin un valor superior a 120 a la columna 7. Només s’han d'imprimir les columnes 2 (nom) i 7 (defensa).

Solució:
awk -F, '$7 > 120 {print $2, $7}' pokemon.csv

Exercici 2: Trobar Pokémon que evolucionen
Descripció:
Volem mostrar els noms de tots els Pokémon que poden evolucionar (valor "True" a la columna 13) i que són de la primera generació (columna 12).

Instruccions:
Utilitza AWK per processar el fitxer pokemon.csv i mostrar només aquells Pokémon de la primera generació (columna 12 amb valor 1) i que poden evolucionar (columna 13 amb valor "True"). Mostra el nom del Pokémon (columna 2).

Solució:
awk -F, '$12 == 1 && $13 == "True" {print $2}' pokemon.csv

- Comanda que permeti imprimir totes les línies que comencin per un "M" majúscula. Mostra només el nom del pokémon (columna 2).

    ```bash
    awk -F, '{ if ($2 ~ /^M/) print $2 }' pokemon.csv
    ```

    Aquesta solució utilitza una estructura if dins de l'script awk per verificar si una línia compleix la condició que volem. En aquest cas, la condició és que el nom del Pokémon (que es troba a la segona columna del CSV) comenci amb la lletra "M" majúscula.

- Imprimiu tots els pokemons que siguin del tipus herba i fosc. Imprimiu el nom, tipus 1 i tipus 2. Podeu fer servir l'operador && per crear l'expressió regular.

    ```bash
    awk -F, '{ if ($3 == "Grass" && $4 == "Dark") print $2, $3, $4 }' pokemon.csv
    ```

    Aquesta alternativa utilitza el bloc BEGIN dins de awk. Aquest bloc s'executa abans que el programa comenci a processar les línies del fitxer. L'ús principal és configurar coses que volem que estiguin preparades abans de començar, com ara definir el delimitador de camps (en aquest cas, una coma perquè és un CSV).

- Substituir totes les aparicions d'una paraula en un fitxer `.csv` fer una versió amb `sed` i una altra amb `awk`.

  - **Versió amb `awk`**:

    ```bash
    awk -F, '{gsub(/paraula_antiga/, "paraula_nova"); print}' pokemon.csv
    ```

    **Explicació:**

    - `gsub(/paraula_antiga/, "paraula_nova")` és una funció d'AWK que reemplaça totes les aparicions de "paraula_antiga" per "paraula_nova" en cada línia.
    - AWK processa el fitxer línia per línia i realitza la substitució globalment (similar al comportament de `sed` amb l'opció `g`).
    - Després de fer la substitució, imprimeix la línia modificada.

  - **Versió amb `sed`**:

    ```bash
    sed 's/paraula_antiga/paraula_nova/g' pokemon.csv
    ```

- Comanda per simular `head -n 5 pokemon.csv` amb `awk`.

    ```bash
    awk 'NR <= 5' pokemon.csv
    ```

    Aquesta solució utilitza la variable NR d'AWK, que representa el número de registre (línia) actual. La condició `NR <= 5` assegura que només s'imprimeixin les primeres 5 línies del fitxer.

- Comanda per simular `tail -n 5 pokemon.csv` amb `awk`.

    ```bash
    awk '{line[NR] = $0} END {for (i = NR-4; i <= NR; i++) print line[i]}' pokemon.csv
    ```

    **Explicació**:
      - AWK no té una funció nativa com `tail` per accedir directament a les últimes línies, per la qual cosa aquí utilitzem un array.
      - `line[NR] = $0` emmagatzema cada línia del fitxer en l'array `line`, utilitzant el número de línia (`NR`) com a índex.
      - Al bloc `END`, quan AWK ha acabat de processar totes les línies, imprimim les últimes 5 línies. L'expressió `for (i = NR-4; i <= NR; i++)` recorre les últimes 5 línies de l'array (de `NR-4` a `NR`).

- Cerca el Pokémon més fort i mes feble de tipus Planta.

    ```bash
    awk -F, 'BEGIN { max=0; min=100 } 
    { 
        if ($3 == "Grass") { 
            if ($7 > max) { 
                max=$7; 
                strongest=$2 
            } 
            if ($7 < min) { 
                min=$7; 
                weakest=$2 
            } 
        } 
    } 
    END { 
        print "Weakest:", weakest; 
        print "Strongest:", strongest 
    }' pokemon.csv
    ```

- Transformar noms de Pokémon a majúscules.

    ```bash
    awk -F, 'BEGIN {OFS=","} 
    { 
        $2 = toupper($2); 
        print 
    }' pokemon.csv
    ```

- Comanda que permeti comptar tots els pokemons que tenim a la pokedex i que tingui la sortida següent:

    ```bash
    awk 'BEGIN { print "Counting pokemons..." } NR > 1 { } END { print "There are", NR-1, "pokemons." }' pokemon.csv
    ```

- Implementeu un script que permeti comptar el nombre de pokemons de tipus aigua i roca. La sortida ha de ser semblant a:

    ```bash
    Water: 64
    Rock: 50
    Others: 689
    ```

    ```bash
    awk -F, 'BEGIN{ water=0; rock=0; others=0 } 
    {
        if ($3 == "Water" || $4 == "Water") {
            water++
        } else if ($3 == "Rock" || $4 == "Rock") {
            rock++
        } else {
            others++
        }
    } 
    END{ print "Water:" water "\nRock:" rock "\nOthers:" others }' pokemon.csv
    ```

- Implementeu un script que mostri la pokedex en ordre invers. Però mantenint la primera línia com a capçalera.

    ```bash
    awk -F, '
    NR == 1 { 
        header = $0  # Guardem la capçalera
        next        # Saltem a la següent línia
    } 
    {
        count[NR] = $0  # Guardem cada línia a un array indexat per NR
    }
    END {
        # Imprimim la capçalera
        print header   
        # Imprimim les línies en ordre invers
        for (i = NR; i > 1; i--) {
            print count[i]  
        }
    }' pokemon.csv
    ```

- Implementeu un filtre que mostri els pokemons de tipus foc. Mostra nom, tipus 1 i tipus 2.

    ```bash
    awk -F, '
    $3 == "Fire" || $4 == "Fire" || $3 == "Fighting" || $4 == "Fighting" {
        print $2, $3, $4
    }' pokemon.csv
    ```

- Implementeu un script que mostri el nom dels pokemons que tinguin una "b" o "B" seguida de "ut". Mostra només el nom del pokémon.

    ```bash
    awk -F, '/[bB]ut/ {print $2}' pokemon.csv
    ```

- Implementeu un script que mostri els pokemons que comencin per "K" majúscula. Mostra només el nom del pokémon.

    ```bash
    awk -F, '$2 ~ /^K/ {print $2}' pokemon.csv
    ```

- Mostra els pokemons que siguin del tipus foc o lluita. Imprimeix nom, tipus 1 i tipus 2.

    ```bash
    awk -F, '/Fire|Fighting/ {print $2, $3, $4}' pokemon.csv
    ```

- Imprimir el nom dels pokemons de la primera generació que siguin llegendaris.

    ```bash
    awk -F, '$12 == 1 && $13 == "True" {print $2}' pokemon.csv
    ```

- Coomptar les línies del fitxer pokemon.csv sense utilitzar una variable explícita.

    ```bash
    awk 'END{ print NR }' pokemon.csv
    ```

- Comprouvar que la suma dels atributs coincideix amb el total (Filtrar només Pokémons de tipus 'Foc').

    ```bash
    awk -F, '$3 == "Fire" { print $2"->Total="$5"=="($6+$7+$8+$9+$10+$11)}' pokemon.csv
    ```

- Traduïm la capçalera al català i ometre les línies incorrectes.

    ```bash
    awk 'NR==1 { $1="#"; $2="Nom"; $3="Tipus 1"; $4="Tipus 2"; $5="Total"; $6="HP"; $7="Atac"; $8="Defensa"; $9="Atac Especial"; $10="Defensa Especial"; $11="Velocitat"; $12="Generació"; $13="Llegendari"; print $0 } NF == 13 {print}' pokemon.csv
    ```

- Comptar Pokémons de tipus Foc de la primera generació (sense Mega).

    ```bash
    awk -F, '$3 == "Fire" && $12 == 1 && $2 !~ /Mega/ { n++ } END{ print "There are", n, "fire type pokemons in the first generation without Mega evolutions." }' pokemon.csv
    ```

- Imprimir el nom del Pokémon i si és llegendari o no, filtrant per tipus 'Drac'.

    ```bash
    awk -F, '$3 == "Dragon" { print $2, "és", ($13 == "True" ? "un pokémon llegendari" : "un pokémon comú") }' pokemon.csv
    ```

### 1. Pokémon amb lletra repetida tres vegades consecutives

```awk
awk -F, '$2 ~ /(.)\1\1/ {print $2}' pokemon.csv
```

Aquest script fa el següent:
- Utilitza ',' com a separador de camps (`-F,`).
- Busca en el segon camp (`$2`, que és el nom del Pokémon) un patró on qualsevol caràcter (`.`) es repeteix exactament tres vegades consecutives (`\1\1`).
- Imprimeix el nom del Pokémon si compleix aquesta condició.

### 2. Pokémon amb nom palindròmic

```awk
awk -F, 'function reverse(str) {
    res = ""
    for (i = length(str); i > 0; i--)
        res = res substr(str, i, 1)
    return res
}
$2 == reverse($2) {print $2}' pokemon.csv
```

Aquest script:
- Defineix una funció `reverse` que inverteix una cadena de text.
- Compara el nom del Pokémon (`$2`) amb la seva versió invertida.
- Imprimeix el nom si són iguals, indicant que és un palíndrom.

### 3. Pokémon amb tipus primari que comença i acaba amb la mateixa lletra

```awk
awk -F, 'substr($3, 1, 1) == substr($3, length($3), 1) {print $2, $3}' pokemon.csv
```

Aquest script:
- Compara la primera lletra del tercer camp (`$3`, tipus primari) amb l'última.
- Si són iguals, imprimeix el nom del Pokémon (`$2`) i el seu tipus primari (`$3`).


### Intermedis
Exercici 1: Comptar el nombre de Pokémon de cada tipus
Descripció: Has de comptar quants Pokémon hi ha de tipus "Fire" i "Dragon" a la Pokedex. A més, has de comptar quants Pokémon no pertanyen a cap d’aquests tipus (Others).
Solució:
awk -F, '{
    if ($3 == "Fire" || $4 == "Fire") fire++;
    else if ($3 == "Dragon" || $4 == "Dragon") dragon++;
    else others++;
} 
END { 
    print "Fire:", fire;
    print "Dragon:", dragon;
    print "Others:", others;
}' pokemon.csv


Exercici 2: Afegir una columna amb la mitjana dels atributs de cada Pokémon
Descripció: Implementeu una comanda que afegeixi una columna nova al final de cada fila, que contingui la mitjana aritmètica dels atributs de cada Pokémon. La mitjana ha d'incloure els valors de les columnes 6 (HP), 7 (Attack), 8 (Defense), 9 (Sp. Atk), 10 (Sp. Def), i 11 (Speed). 
Solució:
awk -F, '{
    avg = ($6 + $7 + $8 + $9 + $10 + $11) / 6;
    print $0 "," avg;
}' pokemon.csv

### Avançats
Exercici 1: Mostrar la Pokédex en ordre invers
L'objectiu és invertir l'ordre dels registres, mantenint la primera línia com a capçalera.
awk -F, '
BEGIN {
    OFS = ","
}
NR == 1 {
    capçalera = $0;
    next
}
{
    pokedex[NR-1] = $0
}
END {
    print capçalera  # Imprimir la capçalera
    for (i = length(pokedex); i > 0; i--) {
        print pokedex[i]
    }
}' pokemon.csv

### 1. Mitjana d'atac per tipus primari

```awk
awk -F, 'NR>1 {sum[$3] += $7; count[$3]++} 
END {
    for (type in sum) 
        printf "%-10s: %.2f\n", type, sum[type]/count[type]
}' pokemon.csv | sort -k2 -rn
```

Aquest script:
- Suma l'atac (`$7`) per a cada tipus primari (`$3`) i compta quants Pokémon hi ha de cada tipus.
- Al final, calcula i imprimeix la mitjana d'atac per a cada tipus.
- Ordena els resultats de major a menor mitjana d'atac.

### 2. Combinacions més rares de tipus primari i secundari

```awk
awk -F, 'NR>1 {combo[$3 "-" $4]++} 
END {
    for (c in combo)
        if (combo[c] == 1) print c
}' pokemon.csv
```

Aquest script:
- Crea una clau única per cada combinació de tipus primari (`$3`) i secundari (`$4`).
- Compta quantes vegades apareix cada combinació.
- Imprimeix només les combinacions que apareixen una sola vegada.

### 3. Histograma ASCII de distribució de velocitats

```awk
awk -F, 'NR>1 {speed[int($11/10)]++} 
END {
    for (i=0; i<=20; i++) {
        printf "%3d-%3d: ", i*10, (i+1)*10-1
        for (j=0; j<speed[i]; j+=5) printf "*"
        print ""
    }
}' pokemon.csv
```

Aquest script:
- Agrupa les velocitats (`$11`) en rangs de 10.
- Crea un histograma ASCII on cada '*' representa 5 Pokémon en aquell rang de velocitat.



### Avançats

1. Crear un sistema de recomanació simple que suggereixi Pokémon similars basats en estadístiques:

### 1. Sistema de recomanació simple

```awk
awk -F, '
function dist(a,b,c,d,e,f, x,y,z,w,v,u) {
    return sqrt((a-x)^2 + (b-y)^2 + (c-z)^2 + (d-w)^2 + (e-v)^2 + (f-u)^2)
}
NR==1 {print "Nom del Pokémon de referència:"; getline ref < "/dev/tty"}
$2 == ref {p=$2; a=$6; b=$7; c=$8; d=$9; e=$10; f=$11}
NR>1 && $2 != ref {
    similarity = dist(a,b,c,d,e,f, $6,$7,$8,$9,$10,$11)
    printf "%s %.2f\n", $2, similarity
}' pokemon.csv | sort -k2 -n | head -5
```

Aquest script:
- Demana a l'usuari un Pokémon de referència.
- Calcula la "distància" entre les estadístiques del Pokémon de referència i tots els altres.
- Mostra els 5 Pokémon més similars basats en aquesta distància.

### 2. Generador de noms de Pokémon

```awk
BEGIN {srand()}
NR>1 {
    split($2, chars, "")
    for (i in chars) {
        prefix[length(prefix)+1] = substr($2, 1, i)
        suffix[length(suffix)+1] = substr($2, i+1)
    }
}
END {
    for (i=1; i<=10; i++) {
        p = prefix[int(rand() * length(prefix)) + 1]
        s = suffix[int(rand() * length(suffix)) + 1]
        print p s
    }
}' pokemon.csv
```

Aquest script:
- Divideix cada nom de Pokémon en tots els possibles prefixos i sufixos.
- Genera 10 nous noms combinant aleatòriament aquests prefixos i sufixos.

### 3. Sistema de puntuació d'equilibri

```awk
awk -F, '
function stdev(sum, sumsq, n) {
    return sqrt(sumsq/n - (sum/n)^2)
}
NR>1 {
    sum = $6 + $7 + $8 + $9 + $10 + $11
    sumsq = $6^2 + $7^2 + $8^2 + $9^2 + $10^2 + $11^2
    score = sum / stdev(sum, sumsq, 6)
    printf "%s: %.2f\n", $2, score
}' pokemon.csv | sort -k2 -rn | head -10
```

Aquest script:
- Calcula una puntuació d'equilibri per a cada Pokémon basada en la suma de les seves estadístiques dividida per la desviació estàndard d'aquestes.
- Una puntuació alta indica un Pokémon amb estadístiques altes i equilibrades.
- Mostra els 10 Pokémon més equilibrats segons aquesta mètrica.


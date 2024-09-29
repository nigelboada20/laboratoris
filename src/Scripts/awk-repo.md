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

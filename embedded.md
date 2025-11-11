#Embedded Systems

## ADDER:
```vhdl
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

entity adder is
    generic(N : integer := 8);
    port(
        a : in std_logic_vector(N-1 downto 0);
        b : in std_logic_vector(N-1 downto 0);
        y : out std_logic_vector(N-1 downto 0)
    );
end adder;

architecture behavioral of adder is
begin
    y <= std_logic_vector(signed(a) + signed(b));
end behavioral;
```

## VECGEN con apertura da file
```vhdl
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;
use std.textio.all;
use std.env.all;

entity vecgen is
    generic (N : natural := 8);
    port(
        a : out std_logic_vector(N-1 downto 0);
        b : out std_logic_vector(N-1 downto 0);
        y : in std_logic_vector(N-1 downto 0)
    );
end vecgen;

architecture behaviorH of vecgen is
begin
end behaviorH;


architecture behaviorF of vecgen is
    file inFile  : text open read_mode is "input.txt";
    file outFile : text open write_mode is "output.txt";
    signal clk : std_logic := '0';
begin
    clk <= not clk after 10 ns;
    process(clk)
        variable inLine  : line;
        variable outLine : line;
        variable ra, rb, re : integer;
    begin
        if rising_edge(clk) then
            if not endfile(inFile) then
                readline(inFile, inLine);
                read(inLine, ra);
                read(inLine, rb);
                read(inLine, re);
                a <= std_logic_vector(to_signed(ra, a'length));
                b <= std_logic_vector(to_signed(rb, b'length));
            else
                finish;
            end if;
        elsif clk'event and clk='0' then
            write(outLine, ra);
            write(outLine, string'(" "));
            write(outLine, rb);
            write(outLine, string'(" "));
            write(outLine, to_integer(signed(y)));
            writeline(outFile, outLine);
        end if;
    end process;
end behaviorF;
```

## TESTBENCH
```vhdl
library ieee;
use ieee.std_logic_1164.all;

entity tb is
end tb;

architecture structure of tb is
    constant TBN : natural := 64;

    component adder is
        generic (N : integer := 8);
        port(
            a : in std_logic_vector(N-1 downto 0);
            b : in std_logic_vector(N-1 downto 0);
            y : out std_logic_vector(N-1 downto 0)
        );
    end component;

    signal ia, ib, iy : std_logic_vector((TBN-1) downto 0) := (others => '0');
    
    for vec1 : vecgen use entity work.vecgen(behaviorF);
begin
    add1 : adder
        generic map (N => TBN)
        port map (
            a => ia,
            b => ib,
            y => iy
        );

    vec1 : vecgen
        generic map (N => TBN)
        port map (
            a => ia,
            b => ib,
            y => iy
        );
end structure;
```
## Esempio in SystemVerilog
### Adder:
```verilog
`timescale 1ns/1ns 
//specifica unità di tempo e precisione ` indica che è una direttiva del preprocessore
//magior tick = 1ns, minor tick = 1ns
//definizione di porta racchiusa in ();
module adder_m # (N = 16) (//# è il generic, N è un nome
    input logic signed [N - 1 : 0] a,
    input logic signed [N - 1 : 0] b,
    output logic signed [N - 1 : 0] y,

    always_comb begin //keywords, tra begin ed end il segmento verrà valutato in maniera combinatoria => quando c'è una transizione
        y = a + b;
    end
endmodule
);
```
SystemVerilog è leggermente ambiguo in confronto al vhdl, quando c'è stato il passaggio da Verilog c'è stato un cambiamento dei tipi.

## Test Bench:
```verilog
module testbench_m;
    //nome del modulo
    adder_m # (.N (TBN)) //ciò che dentro al modulo si chiama N assume il valore TBN
    adder( //nome "a caso"
        .a (add1), //segnali
        .b (add2),
        .y (result), //analogamente, a nel modulo ha il valore add1 etc
    );
    
    monitor_m
    monitor (
        initial begin //initial specifica cosa deve essere fatto quando la simulazione inizia
        $dumpfile (VCDFILE);
        $dumpvars(0, testbenc_m);; //quali variabili inizializzate e quando
        \# 1000; //wait 1000ns
        $display (" "); //stampo a terminale, display va anche a capo
        $finish;
    end
    )
endmodule
```

## Generatore:
Va specificato il valore di default quando c'è un errore, input e output

```verilog

    initial clk = 0; //statement di inizializzazione del clock
    //tutti gli statement con initial sono concorrenti tra loro

    always #(CP / 2) clk = -clk; 
    //always viene eseguito in continuazione 
    // # è un operatore di attesa temporale (attende metà periodo di clock)

    assign un_segnale = un_altro_segnale;
    //assign è un'assegnazione continua, ogni volta che un_altro_segnale cambia, anche un_segnale cambia, è come tirare un filo tra i due
```

## Monitor:
$fscanf(file, \%tipo, segnale) per leggere il file
$fopen per aprire i file

Chi garantisce che il tempo che noi abbiamo impostato di attesa per i risultati sia sufficiente affinché la rete stessa li riesca a generare in caso di una tecnologia molto lenta? (Chi garantisce che i 5ns sono sufficienti?) Essendo una simulazione il ritardo della rete è nullo.
Se conosciamo il tempo di propagazione massimo della rete, possiamo fare in modo che il nostro testbench attenda un tempo maggiore di tale ritardo prima di leggere i risultati.

## Sintesi Architetturale e Logica
Il processo di sintesi architetturale converte il codice VHDL (comportamentale) in una rete di porte logiche. 

### Sintesi Architetturale
La sintesi architetturale è in grado di interpretare solo un sottoinsieme del VHDL, detto VHDL sintetizzabile. Alcuni costrutti del VHDL non sono sintetizzabili, ad esempio i processi che utilizzano file di testo per leggere o scrivere dati. Questi processi sono utilizzati solo nei testbench per generare vettori di test o per monitorare i risultati della simulazione e non fanno parte del circuito sintetizzato.

### Sintesi Logica
La sintesi logica converte la rete di porte logiche in una netlist, che è una descrizione del circuito in termini di gate logici e connessioni tra di essi. La sintesi logica ottimizza la rete di porte logiche per ridurre il numero di gate e migliorare le prestazioni del circuito.

Pure i dispositivi integrati hanno a disposizione delle librerie di celle logiche predefinite, che possono essere utilizzate per implementare le funzioni logiche del circuito.

## Logica Combinatoria

La logica combinatoria è un tipo di logica digitale in cui l'uscita dipende solo dagli ingressi attuali, senza memoria o stato. In altre parole, l'uscita è una funzione diretta degli ingressi.
Le operazioni logiche di base sono:
(in vhdl sono scritti come not, and, nor, or, nand, xor, xnor
in systemverilog sono scritti come ~, &, |, ^, ~&, ~|, ~)

VHDL è case `insensitive`, SystemVerilog è case `sensitive`.
```vhdl
y4 <= a nand b; --vhdl
```
In systemverilog si scrive:
```verilog
y4 = ~(a & b);
```

y4 è uno statement, l'assegnazione viene fatta quando uno degli ingressi cambia
Se ce ne fosse più di uno, l'assegnazione viene fatta ogni volta che uno qualsiasi degli ingressi cambia. Gli statement sono concorrenti tra loro. Lo stesso discorso vale per systemverilog e vengono definiti continuos assign. 

Per fare una and tra le celle di un vettore, in VHDL si deve fare la and specificando ogni bit:
y <= a(0) and a(1) and a(2) and ... and a(N-1);

In SystemVerilog si può fare in modo più compatto: con un reduction operator:

```verilog
y = &a; //fa la and di tutti i bit di a
```

Ammettono una corrispondenza biunivoca con una macchina di turing se esistono un assegnamento, il salto condizionato e la iterazione. Gli HDL supportano tutti e tre questi costrutti e sono quindi Turing complete.

### Conditional Assignment
In VHDL:
```vhdl
y <= a when sel = '1' else b;
``` 
in SystemVerilog:
```verilog
y = sel ? a : b;
```
Analogamente si può dare con i vettori.

### Multiplexer
In VHDL:
```vhdl
with sel select
    y <= a when "00" else
         b when "01" else
         c when "10" else
         d;
        --come uno switch case in C
        --essendo sel un std_logic_vector, la else ha tutte le altre combinazioni possibili
        --supponendo che sel non assuma mai valori diversi da quelli specificati, l'ultima else è ridondante
        --se lascio l'else sono semanticamente diverse, in quanto se sel assume un valore non specificato dall'else, y mantiene il valore precedente e devo inserire un latch per questo
        --se lo specifico a priori, so quale valore assegnare a y in ogni condizione
        --conviene sempre specificare l'else per evitare latch indesiderati
```
In SystemVerilog:
```verilog
    assign y = s[1] ? 
        (s[0] ? d3 : d2) : (s[0] ? d1 : d0);
```


### Internal Signals

In systemVerilog i segnali interni sono definiti con la keyword `logic`
mentre in VHDL sono definiti con la keyword `signal`.

### Numeri
In VHDL i numeri sono rappresentati come vettori di bit (std_logic_vector) e devono essere convertiti in tipi numerici (signed o unsigned) per eseguire operazioni aritmetiche. In SystemVerilog, i numeri possono essere rappresentati direttamente come tipi numerici (logic signed o logic unsigned), semplificando le operazioni aritmetiche.

I bit sono tra singoli apici, i vettori tra doppi apici.

Si può cambiare la base del numero con il prefisso (VHDL):
- b per binario
- o per ottale
- d per decimale
- x per esadecimale

In SystemVerilog si può specificare la base del numero con il formato: lunghezza ' base valore
- b per binario
- o per ottale
- d per decimale
- h per esadecimale

Gli zeri vengono aggiunti a sinistra per raggiungere la lunghezza specificata (padding).

### Tristate
Corrisponde ad alta impedenza, in VHDL è rappresentato con la keyword 'Z', in SystemVerilog con la keyword 'z'. IN VHDL si usa la keyword `std_logic` per rappresentare i segnali che possono assumere più valori (0, 1, Z, X, etc.), mentre in SystemVerilog si usa la keyword `tri` per rappresentare i segnali tristate. Tra `tri` e `trireg` c'è la differenza che `trireg` mantiene il valore precedente quando è in stato Z, mentre `tri` no.

### Bit Coalescing
Si intende selezionare o integrare porzioni di bus in sottoporzioni più piccole o più grandi. In VHDL si usa & per concatenare i vettori. In VHDL si usa (m downto n) per selezionare una porzione di un vettore, in SystemVerilog si usa [m:n].

```verilog
assign y = {c[2:1], {3{d[0]}}, c[0], 3'b101};
```

```vhdl
y <= c(2 downto 1) & d(0) & d(0) & d(0) & c(0) & "101";
```

### Sign Extension e Zero Extension

In VHDL:
```vhdl 
-- per sign extension si intende estendere il segno del numero
-- si aggiungono bit uguali al bit di segno (bit più significativo)
y <= X''0000" & a when a(15) = '0' else
     X''FFFF" & a;
``` 
In SystemVerilog:
```verilog
assign y = {{16{a[15]}} , a[15:0]}; //sign extension
```

### Rappresentazione in virgola fissa (?)

Se chiamiamo X un certo numero binario e chiamiamo X segnato il suo complemento a 1, X + X segnato = 1 se in complemento a 2 = -1.

X = -1 - X segnato
-X = X segnato + 1

-X = X in complemento a 2 = X segnato + 1

### Virgola Mobile
La rappresentazione in virgola mobile è utilizzata per rappresentare numeri reali con una precisione limitata. In VHDL e SystemVerilog, la rappresentazione in virgola mobile segue lo standard IEEE 754, che definisce il formato dei numeri in virgola mobile a singola precisione (32 bit) e doppia precisione (64 bit).
In VHDL, i numeri in virgola mobile possono essere rappresentati utilizzando il tipo `float` o `double`, mentre in SystemVerilog si utilizzano i tipi `real` o `shortreal`. Le operazioni aritmetiche sui numeri in virgola mobile sono supportate direttamente dai linguaggi, ma è importante considerare le limitazioni di precisione e gli errori di arrotondamento associati a questa rappresentazione.

Se ho un vettore di 8 posizioni con la virgola virtuale a sx. il più piccolo numero rappresentabile è 0, il più piccolo intero è 2^-8 = 0.00390625
Il più grande numero rappresentabile è dato da tutti uni.

Supponendo di avere un adder tra a e b, 
con a = 1 - 2^-8
e b = 2^-8
il risultato atteso è 1, ma il risultato effettivo sarà 1 ma non è rappresentabile con 8 bit con la virgola a sx, quindi per il risultato sarebbe necessaria una posizione in più. La posizione della virgola cambia lungo il datapath a seconda dell'operazione che si sta effettuando.

Se voglio rappresentare numeri negativi, uso il complemento a 2. Il primo bit sarà -2^0 e gli altri 

## MANCA UN PEZZO

## Sequential Logics

è conveniente strutturare il programma a livello gerarchico per facilitare la lettura e la manutenzione del codice. In VHDL, si possono utilizzare entità e architetture per creare moduli gerarchici, mentre in SystemVerilog si possono utilizzare moduli e interfacce. 

Conviene non forzare la sintesi di tutti gli elementi come ad esempio i sommatori, in quanto i sintetizzatori moderni sono in grado di ottimizzare il circuito in modo efficiente. Forzare la sintesi di tutti gli elementi può portare a un aumento del numero di gate e a una riduzione delle prestazioni del circuito. 

Il comportamento è descritto tramite logica sequeziale. Ci sono degli idiomi utili per la sintesi che corrispondono a uno specifico circuito sequenziale; altri stili di programmazione possono portare a errori nei circuiti sintetizzati. Una buona pratica è usare il nome `clk` per il clock 

### Register

La maggior parte è creato usando dei flip flop edge-triggered tipo D.

```vhdl
process (clk) begin --se c'è una variazione di clock
    if clk'event and clk = '1' then --fronte di salita
    q <= d; --metto l'uscita uguale all'ingresso (ovvero d)
    --statement concorrente
    end if;
```
Tutto il process viene valutato alla variazione degli ingressi (non è un'esecuzione)

Questo codice non dice cosa fare quando il `clk` è 1 o 0, perché è un registro? Non specificando tutte le condizioni dell'if, mantiene il valore tramite un registro perché tiene il valore precedente.

```vhdl
RISING_EDGE (clk) = clk'event and clk = '1'
```

in system verilog:

```verilog
always_ff @(posedge clk)
    q <= d; //definisce il nonblocking assignment
```

`always_ff` è usato come `always` ma esclusivamente per i flip flop per ridurre gli errori
```verilog
(always @(sensitivity vist))

(always_latch (funziona sul livello))

(always_comb (funzione combinatoria, non basata sul clock))
```

### Resettable Register

Per avere lo stessa uscita per gli stessi ingressi bisogna che il flipflop parta da uno stato zero (con reset) se lo stato è molto importante. Es. il program counter è necessario che parta dallo stesso valore ogni volta.

```vhdl
process (clk) begin
    if clk'event and clk = '1' then
        if reset = '1' then --reset positivo alto e sincrono
            q <= (others => '0');
        else
            q <= d;
        end if;
    end if;
end process;
```

Solitamente si usano i reset sincroni perché sono `bound` al `clk`. 

```verilog
always_ff @(posedge clk)
    if (reset) q <= 4'b0;
    else q <= d;
```

Se non lo voglio sincrono il reset:
```vhdl
process (clk, reset) begin
    if reset = '1' then --reset positivo alto
            q <= (others => '0');
    if clk'event and clk = '1' then
        q <= d;
    end if;
end process;
```

```verilog
always_ff @(posedge clk, posedge reset)
    if(reset) <= 4'b0;
    else q <= d;
```

Se mi interessa avere un segnale di enable per poter leggere/scrivere solo da un registro:
```vhdl
process (clk) begin
    if clk'event and clk = '1' then
        if reset = '1' then
            q <= (others => '0');

```
### Registri resettabili e con enable

Ho una condizione extra dell'if, non solo else `q <= d` ma `if(enable) q <= d`

Necessità di descrivere la cascata di flipflop, posso definire un segnale interno che prende il valore del primo flip flop e se le condizioni lo richiedono (es enable, es positive edge del clock) copio il valore del segnale interno nel secondo flip flop.

### Transparent Latch
Non è opportuno usarlo al posto di enable latch in quanto se ho un enable unico, il segnale si propagherebbe su tutti i latch.

Sono costretto a mantenere stabile il valore di ingresso per tutto il periodo di clock e non solo sul fronte. Il transparent latch diventa opaco con il clock basso.

Al posto di basarsi sul fronte di salita, si basa sull'intero clock = 1

In verilog si usa `always_latch` 

## Contatori

```vhdl
process (clk) begin
    if clk'event and clk = '1' then
        if rst = '1' then q_int <=  "0000"; --reset
            else q_int <= q_int + "0001";
        end if;
    end if;
end process;
```

Va definito un segnale interno `q_int` in quanto un output non può essere usato come segnale di ingress. `q_int` è quindi una copia del segnale `q`.

```verilog
always_ff @(posedge clk)
    if (rst) q <= 4'b0;
    else q <= q + 1;   
```

## Shift register
```verilog
always_ff @(posedge clk)
    if (rst) q <= 0;
    else if (load) q <= d;
    else q <= {q[2:0], sin}; //sposto a sinistra e inserisco il segnale sin

    assign sout = q[3]; 
```


## Always/Process
`always` e `process` sono sinonimi, la sensitivity vist indica qual è la lista dei segnali in base ai quali il corpo del process viene valutato
in vhdl l'assegnazione distingue tra segnali e variabili. Le variabili sono "pezzetti di carta" su cui vado a scrivere. In systemverilog non c'è la differenza e nemmeno il concetto di variabile. L'uguale è non blocking, ciò che conta è solo l'esito finale. Se uso `<=` sono blocking, andrebbe di volta in volta a valutare ciò che accade dopo. Se scrivo con uguale vengono valutati tutti

Se vogliamo la valutazione sincrona, uso `<=`, altrimenti uso `=`

L'assegnazione con <= "termina" fino alla prossima riattivazione 

Il process intero è uno statement concorrente, tutte le istruzioni vengono viste come un unico statement concorrente.
## Memoria
Le memorie normalmente sono organizzate a byte. Una cella di memoria sarà "doppia" divisa tra parte alta e parte bassa. Il micro processore che ha un bus dati e uno di indirizzi di dimensione es 16, avrò due connessioni ai due byte di memoria (8 + 8) in cui i primi 8 li porto sul byte più basso e gli altri su quello più alto.

Supponiamo che la cella di memoria sia 2^N con N dimensione del bus di indirizzi, il bus arriverà ad entrambi i banchi di memoria e tramite i bit che si scrivono sui bus, accedo a uno specifico indirizzo di memoria.

Supponendo che gli indirizzi nel banco di memoria siano 2^N-1, Ciascun banco di memoria occuperà la metà. Supponiamo di aggiungere altri due banchi di dimensione sempre 2^N-1. Se scrivo tutti zeri, scrivo nella prima posizione di tutti e 4 i banchi. Ho la memoria piena ma uso la metà. Ho un bit extra nell'indirizzo di memoria e uso quello per indicare quale parte abilitare (0 o 1) tramite un processo di decodifica. Verrà più complesso a seconda del numero di banchi da decodificare (maggiori se la dimensione del singolo è piccola)
# ADDER
## ADDER VHDL CODE

```vhdl
library ieee; --libreria standard per la logica
use ieee.std_logic_1164.all; --pacchetto per i tipi di dato logici
use ieee.numeric_std.all; --pacchetto per le operazioni aritmetiche sui vettori logici

-- Definizione dell'entità adder con un parametro generico N per la larghezza dei vettori
-- entity: indica la definizione di un modulo hardware
-- generic: definisce un parametro generico per l'entità
-- port: definisce le porte di ingresso e uscita dell'entità

entity adder is
    generic(N : integer := 8);
    port(
        -- a e b sono vettori di ingresso di larghezza N
        -- y è un vettore di uscita di larghezza N
        a : in std_logic_vector(N-1 downto 0);
        b : in std_logic_vector(N-1 downto 0);
        y : out std_logic_vector(N-1 downto 0)
    );
end adder;
-- il modulo va terminato con la parola chiave end seguita dal nome dell'entità

-- Definizione dell'architettura comportamentale dell'adder
-- architecture: indica la definizione dell'implementazione di un'entità
-- behavioral: nome dell'architettura
-- <= : operatore di assegnazione (segnali)
-- signed(): funzione che converte un vettore di tipo std_logic_vector in un tipo signed

architecture behavioral of adder is
begin
    y <= std_logic_vector(signed(a) + signed(b));
end behavioral;
```
## VECGEN VHDL CODE

```vhdl
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;
use std.textio.all; -- per la gestione dei file di testo
use std.env.all; -- per la gestione dell'ambiente di esecuzione

-- vanno definite le entity di tutti i moduli utilizzati nei vari file VHDL

-- si inserisce solo la dichiarazione dell'entity

entity vecgen is
    generic (N : natural := 8);
    port(
        a : out std_logic_vector(N-1 downto 0);
        b : out std_logic_vector(N-1 downto 0);
        y : in std_logic_vector(N-1 downto 0)
    );
end vecgen;


-- si possono definire più architetture per la stessa entity (questa è vuota)
architecture behaviorH of vecgen is
begin
end behaviorH;

-- questa architettura legge i vettori di ingresso da un file di testo
architecture behaviorF of vecgen is
    file inFile  : text open read_mode is "input.txt"; --apertura file di testo in lettura
    file outFile : text open write_mode is "output.txt"; --apertura file di testo in scrittura
    signal clk : std_logic := '0'; --segnale di clock
begin -- inizio della descrizione architetturale
    clk <= not clk after 10 ns; -- generazione del clock con periodo di 20 ns
    process(clk) -- processo sensibile al segnale di clock
        variable inLine  : line; -- variabile per la lettura delle righe del file di input
        variable outLine : line; -- variabile per la scrittura delle righe del file di output
        variable ra, rb, re : integer; -- variabili per i valori interi letti dal file
    begin -- inizio del processo
        if rising_edge(clk) then -- fronte di salita del clock
            if not endfile(inFile) then -- controllo fine file
                readline(inFile, inLine); -- lettura di una riga dal file di input
                read(inLine, ra); -- lettura del primo intero dalla riga
                read(inLine, rb); -- lettura del secondo intero dalla riga
                read(inLine, re); -- lettura del risultato atteso (non usato qui)
                a <= std_logic_vector(to_signed(ra, a'length)); -- conversione e assegnazione del primo intero al segnale a
                b <= std_logic_vector(to_signed(rb, b'length)); -- conversione e assegnazione del secondo intero al segnale b
                --b'length restituisce la larghezza del vettore b ('attributo)
            else
                finish;
            end if;
        elsif clk'event and clk='0' then -- fronte di discesa del clock se c'è un evento
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
## TESTBENCH VHDL CODE

```vhdl
library ieee;
use ieee.std_logic_1164.all;

entity tb is --tb non ha porte
end tb;

architecture structure of tb is
    constant TBN : natural := 64; -- Test Bench Number: larghezza dei vettori di test (definito come costante)

    component adder is -- dichiarazione del componente adder
        generic (N : integer := 8); -- parametro generico N
        port(
            a : in std_logic_vector(N-1 downto 0);
            b : in std_logic_vector(N-1 downto 0);
            y : out std_logic_vector(N-1 downto 0)
        );
    end component;

    -- definizione dei segnali per collegare i componenti
    signal ia, ib, iy : std_logic_vector((TBN-1) downto 0) := (others => '0');
    
    -- indica quale architettura usare per il componente vecgen (ne ha due di cui una vuota)
    for vec1 : vecgen use entity work.vecgen(behaviorF);
begin
    -- crea un istanza del componente adder con etichetta add1
    -- imposta il generic N del componente adder a TBN definito nell'architettura del tb
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
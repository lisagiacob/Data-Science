# Preparazione Esame

Deadline: → February 22, 2024 

## SQL and Warehouses

![Screenshot 2024-02-12 alle 13.20.53.png](Preparazione%20Esame%2007188e8a93b7476e8333a3a4caa3ca08/Screenshot_2024-02-12_alle_13.20.53.png)

```sql
SELECT categoria, 
			 SUM(NumArticoli) as Q_TOT, 
			 SUM(TotFatturato),
			 RANK() OVER(ORDER BY SUM(NumArticoli)) as RANK_NumArticoli,
			 RANK() OVER (ORDER BY SUM(TotFatturato)) as RANK_TotFatturato
FROM Fatturato F, Categoria C
WHERE F.codCatArticolo = C.codCat
GROUP BY categoria
ORDER BY RANK_NumArticoli;         //Order by perchè ho 2 rank e devo scegliere quale utilizzare
```

```sql
SELECT Provincia, Regione, 
			 SUM(TotFatturato) AS F_TOT,
			 RANK() OVER (PARTITION BY Regione 
										ORDER BY F_TOT) AS RANK_TotFatturatoRegione
FROM Fatturato F, Cliente C
WHERE F.codCliente = C.codCliente
GROUP BY Provincia, Regione;
```

```sql
SELECT Provincia, Regione, Mese, 
			 SUM(TotFatturato) AS F_TOT,
			 RANK() OVER(PARTITION BY Mese 
									 ORDER BY F_TOT)
FROM Fatturato F, Cliente C, Tempo T
WHERE F.codCliente = C.codCliente     //Fatturato è il centro stella
			AND F.codTempo = T.codTempo
GROUP BY Provincia, Mese;
```

```sql
SELECT Regione, Mese, 
			 SUM(TotFatturato),
			 //incassi cumulativi
			 SUM(SUM(TotFatturato)) over (PARTITION by Regione 
					ORDER BY Mese ROWS UNBOUNDED PRECEDING),
			 SUM(SUM(TotFatturato)) over (PARTITION by Regione, Anno 
					ORDER BY Mese ROWS UNBOUNDED PRECEDING)
FROM Fatturato F, Cliente C, TempoMese T
WHERE F.codCliente = C.codCliente
			AND F.codTempo = T.codTempo
GROUP BY Regione, Mese, Anno;
```

[DSTBD_Quaderno1-ITA.pdf](Preparazione%20Esame%2007188e8a93b7476e8333a3a4caa3ca08/DSTBD_Quaderno1-ITA.pdf)

![Screenshot 2023-11-27 alle 13.48.38.png](Preparazione%20Esame%2007188e8a93b7476e8333a3a4caa3ca08/Screenshot_2023-11-27_alle_13.48.38.png)

```sql
BIGLIETTO(IdBiglietto, IdTempo, IdMuseo, Tipo, ModAcq, Fascia, Entrate, Numero)
TEMPO(IdTempo, Data, Lavorativo, Mese, Bimestre, Trimestre, Semestre, Anno)
MUSEO(IdMuseo, Nome, Cat, Servizi, Città, Provincia, Regione)
```

```sql
SELECT Tipo, Anno, Mese,
			 SUM(Entrate)/COUNT(DISTINCT Data),
			 SUM(SUM(Entrate)) OVER(PARTITION BY Tipo, Anno
															ORDER BY ANNO ROWS UNBOUNDED PRECEDING),
			 110 * SUM(Numero) / SUM(SUM(Numero)) OVER(PARTITION BY Mese)
FROM BIGLIETTO B, TEMPO T
WHERE B.IdData = T.IdTempo
GROUP BY Tipo, Mese, Anno

SELECT Museo, Tipo, Cat,
			 SUM(Entrate) / SUM(Numero),
			 100 * SUM(Entrate) / SUM(SUM(Entrate)) OVER(PARTITION BY Tipo, Cat),
			 RANK() OVER(PARTITION BY Tipo
									 ORDER BY SUM(Numero) DESC)
FROM MUSEO M, TEMPO T, BIGLIETTO B
WHERE M.IdMuseo = B.IdMuseo AND T.IdTempo = B.IdTempo AND T.Anno = "2021"
GROUP BY Museo, Tipo, Categoria
```

![Screenshot 2024-02-12 alle 15.22.16.png](Preparazione%20Esame%2007188e8a93b7476e8333a3a4caa3ca08/Screenshot_2024-02-12_alle_15.22.16.png)

![Screenshot 2024-02-12 alle 15.31.55.png](Preparazione%20Esame%2007188e8a93b7476e8333a3a4caa3ca08/Screenshot_2024-02-12_alle_15.31.55.png)

```sql
SELECT TipologiaTariffa, Anno,
			 SUM(Prezzo),
			 SUM(SUM(Prezzo)),
			 SUM(SUM(Prezzo)) OVER(PARTITION BY TipologiaTariffa),
			 SUM(SUM(Prezzo)) OVER(PARTITION BY Anno)
FROM
WHERE
GROUP BY TipologiaTariffa, Anno

SELECT 
FROM
WHERE
GROUP BY
```

## Materialized View e Trigger

[Viste-Materializzate-e-trigger_Testo.pdf](Preparazione%20Esame%2007188e8a93b7476e8333a3a4caa3ca08/Viste-Materializzate-e-trigger_Testo.pdf)

1. Definiamo il Blocco A comprendendo i requisiti delle interrogazioni
    1. Per ogni coppia (tipologia dei servizi, semestre), visualizzare l’incasso totale e il numero totale di consulenze, effettuate dai consulenti delle sedi site in Lombardia.
        
        ```sql
        GB TipologiaServizi, Semestre
        Sel Regione <- Predicati di selezione (Regione = Lombardia)
        Aggr SUM(Incasso) SUM(#Cosulenze) <- Aggregati
        
        --- INTERROGAZIONE ASSOCIATA
        SELECT TipologiaServizi, Semestre,
        			 SUM(Incasso),
        			 SUM(#Consulenze),
        FROM INCASSO I, SERVIZIO S, TEMPO T, SEDE-CONSULENTI SC
        WHERE join AND Regione = 'Lombardia'
        GROUP BY TipologiaServizi, Semestre
        ```
        
    2. Considerando solo le aziende italiane e tedesche, selezionare per ogni coppia(Regione delle sedi dei consulenti, Servizio), l’incasso totale e il numero di consulenze effettuate, separatamente per ogni anno.
        
        ```sql
        GB Regione, Servizio, Anno
        Sel Nazionalità (='Italia' || 'Germania')
        Aggr SUM(IncassoTotale), SUM(#Consulenze) 
        
        --- INTERROGAZIONE ASSOCIATA
        SELECT Regione, Servizio, Anno,
        			 SUM(Incasso),
        			 SUM(#Consulenze),
        FROM INCASSO I, SERVIZIO S, TEMPO T, AZIENDA A, SEDE-CONSULENTI SC
        WHERE join AND (Nazionalità = 'Italiana' OR Nazionalità = 'Tedesca')
        GROUP BY Regione, Servizio, Anno;
        ```
        
    3. Considerando solo gli incassi del 2017, 2018, e 2019, per ogni coppia (Tipologia dei servizi, nazionalità dell'azienda), visualizzare l’incasso semestrale e l’incasso semestrale medio per consulenza
        
        ```sql
        GB TipologiaServizio, Nazionalità, 6M
        Sel Anno
        Aggr SUM(IncassoTotale), SUM(#Consulenze)
        
        --- INTERROGAZIONE ASSOCIATA
        SELECT TipologiaServizio, Nazionalità, 6M,
        			 SUM(Incasso),
        			 SUM(Incasso)/SUM(#Consulenze),
        FROM INCASSO I, SERVIZIO S, TEMPO T, AZIENDA A
        WHERE join AND Anno >= 2017 AND Anno <= 2019
        GROUP BY TipologiaServizio, Nazionalità, 6M;
        ```
        
    
    Nelle MW è meglio mettere gli elementi costitutivi, piuttosto che un rapporto pre-calcolato.
    
    ```sql
    CREATE MATERIALIZED VIEW View Incassi
    BUILD IMMEDIATE 
    REFRESH FAST ON COMMIT
    AS
    	Blocco A
    
    Blocco A:
    SELECT --- Tutti i predicati di selezione
    			 TipologiaServizio, Semestre,
    			 Regione, Servizio, Anno,
    			 Nazionalità,
    			 --- Tutti gli aggregati
    			 SUM(IncassoTotale), SUM(#Consulenze)
    
    FROM INCASSO I, SERVIZIO S, TEMPO T, AZIENDA A, SEDE-CONSULENTI SC
    WHERE join --- basta perchè i predicati di selezione non sono in comune tra le query
    					 --- Attenzione se ho un predicato di selezione nelle query, questo deve essere nella SELECT della MW
    GROUP BY TipologiaServizio, Semestre, Regione, Servizio, Anno, Nazionalità;
    				 --- Se nella SELECT ci sono dei predicati per la where, devo aggiungerli nella GB
    ```
    
    Note:
    
    - Gli attributi presenti nei predicati di selezione devono essere disponibili nella vista
    - Normalmente non si indicano condizioni di selezione nella vista ← per consentire l’uso generico
    - Le funzioni aggregate che compaiono nella vista devono essere distributive, in modo da consentire il calcolo di aggregati di livello superiore
2. Si individui la combinazione minimale di attributi che costituisce un identificatore per la vista materializzata ViewIncassi.
La granularità della vista è data dalla group by → guardiamo la group by e vediamo se ci sono delle dipendenze e scegliamo il più granulare
(ex. Semestre, Anno → Semestre) 
TipologiaServizio, Semestre, Regione, Servizio, Anno, Nazionalità → Semestre, Regione, Servizio, Nazionalità
Dove regione e Nazionalità non sono una dipendenza in quanto sono dimensioni separate
Identificatori minimali: Semestre, Regione, Servizio, Nazionalità
3. Si ipotizzi che che la vista materializzata ViewIncassi sia gestita senza utilizzare l’istruzione CREATE MATERIALIZED VIEW. La tabella derivata che realizza la vista materializzata è definita dalla seguente istruzione SQL

4. Si ipotizzi che la gestione dell’aggiornamento della vista materializzata sia svolta mediante trigger. S[i](http://trigger.si/) scriva il trigger per propagare le modifiche alla vista materializzata ViewIncassi in caso di inserimento di un nuovo record nella tabella dei fatti INCASSO.
Trigger AFTER opera dopo l’esecuzione dell’evento innescante
Trigger ROW è eseguito una volta per ogni riga modificata dall’evento innescante
La granularità della vista è molto diversa da quella incasso, bisogna quindi risolvere i problemi dati dal cambiamento delle granularità.
Operazioni da svolgere:
1. Lettura valori necessari degli attributi dalle dimensioni(ex. semestre da Tempo - perchè io orig. ho solo l’identificatore del tempo)
2. Verifica dell’esistenza della tupla nella vista
3. Se la tupla esiste → UPDATE else INSERT
    
    ```sql
    CREATE OR REPLACE TRIGGER RefreshVM_ViewIncassi
    AFTER INSERT ON INCASSO                         --- Opera dopo l’esecuzione dell’evento innescante
    FOR EACH ROW                                    --- Trigger eseguito per ogni riga
    DECLARE                                         --- Dichiaro le variabili che mi servono
    	VarServizio, VarTipoS VARCHAR(20);
    	VarSemestre, VarAnno VARCHAR(20);
    	VarRegione VARCHAR(20);
    	VarNazionalità VARCHAR(20);
    	N INTEGER;
    --- Inizio corpo del Trigger
    BEGIN
    	--- Lettura dei valori necessari che non sono disponibili in INCASSO
    	SELECT Servizio, TipologiaServizio
    		INTO VarServizio, VarTipoS
    	FROM SERVIZIO
    	WHERE IdServizio = :NEW.IdServizo;
    	
    	SELECT Semestre, Anno
    		INTO VarSemestre, VarAnno
    	FROM TEMPO
    	WHERE IdTempo = :NEW.IdTempo;
    	
    	SELECT Regione 
    		INTO VarRegione
    	FROM SEDE-CONSULENTI
    	WHERE IdSede = :NEW.IdSede;
    	
    	SELECT Nazionalità 
    		INTO VarNazionalità
    	FROM AZIENDA
    	WHERE IdCategoriaAzienda = :NEW.IdCategoriaAzienda;
    	
    	--- Verifico l’esistenza della tupla con i valori letti dall'identificatore minimale
    	---(Servizio, Semestre, Regione, Nazionalità)
    	SELECT COUNT(*) INTO N
    	FROM ViewIncassi
    	WHERE Servizio = VarServizio
    		AND Semestre = VarSemestre
    		AND Regione = VarRegione
    		AND Nazionalità = VarNazionalità;
    	
    	IF (N > 0) THEN 
    		UPDATE ViewIncassi
    		--- Aggiorno i valori degli aggregati
    		SET IncassoTotale  = IncassoTotale + :NEW.Incasso
    				NumConsTot = NumConsTot + :NEW.#Consulenze
    		WHERE Servizio = VarServizio
    			AND Semestre = VarSemestre
    			AND Regione = VarRegione
    			And Nazionalià = VarNazionalità;
    	ELSE
    		INSERT INTO ViewIncassi(...nomi colonne...)                           --- Tupla nuova, quindi prima comparizione -> NEW
    		VALUES(VarServizio, VarTipoS, VarSemestre, VarAnno, VarRegione, VarNazionalià, :NEW.Incasso, :NEW.#Consulenze)
    	END IF;
    END;
    ```
    

## Cardinalità e indici:

Regole euristiche:

1. Grandezza di una tabella:
    1. Una tabella è piccola se $card(T)≤ 10^3 \ tuple$ → no indice
    2. Se una tabella è medio/grande $card(T) > 10^3 \ tuple$ bisogna valutarne l’indice
2. Quando un predicato è selettivo? (←quando quindi mi può essere utile l’indice)
    1. La selettività è elevata quando la frazione di tuple che ‘esce’ è  $\le 1/10$, e valuto quindi l’indice
    2. Se la selettività è media/bassa, tipicamente non valuto l’utilizzo dell’indice

Procedimento:

1. Analisi delle tabelle → guardo le statistiche:
    1. cardinalità
    2. estremi di variazione del dominio: min/max e se la distribuzione è uniforma per stimare la selettività, 
    3. Numero di valori distinti → stima cardinalità $\sigma_p \ T$
2. Ispezione istruzione SQL
3. Definizione Query Tree in algebra estesa
4. Valutazione della cardinalità dei risultati intermedi e finale → separare i diversi predicati di selezione
5. Definizione del piano di esecuzione senza strutture fisiche associate
6. Valutazione di possibili strutture accessorie.
Per ogni indice possibile ragiono su selettività, group by, copertura, primario o secondario

## Anticipo GB:

![Screenshot 2024-02-16 alle 11.59.19.png](Preparazione%20Esame%2007188e8a93b7476e8333a3a4caa3ca08/Screenshot_2024-02-16_alle_11.59.19.png)

Abbiamo un SemiJoin: il ramo di destra serve per selezionare ma non esce nel risultato → posso anticipare solo sul ramo di sinistra, escludo k,i,j
Il semiJoin è su CodV, posso anticipare la GB prima del SemiJoin perchè la GB è a sua volta su CodV
h non è la posizione migliore per aumentare l’efficienza dell’interrogazione.
Il join su f e g è un join su CodV, quindi diventa più efficiente se faccio prima la GB → ho come alternative f e g
Nel ramo g non ho DataInizio (non fa parte della tabella Veicolo) e CodV è Chiave primaria di veicolo → una GB su una chiave primaria è priva di significato → g non ha senso
Nel ramo f invece ho a disposizione DataInizio dalla tabella Noleggio-Veicolo, raggruppa i CodV invece di tenere tanti noleggi per lo stesso veicolo.
Possiamo fare meglio di f? No, perchè il join precedente è su CodU, se accorpassimo su CodV, facendo scomparire CodU, non potremo farlo

**RECAP:**

Indici: Li creo con tabelle con cardinalità $> 10^2$, predicato di selezione $\le \frac1{10}$. Se il predicato selettivo è una equivalenza allora è un indice hash, altrimenti se è un range è B$^+$Tree

Cardinalità join = Prodotto dei due rami / cardinalità della tabella su cui faccio il join ← ossia la tabella con card più bassa

Anticipo GB:  Non si anticipa sulla query interna

## Warm Restart

B(Tn): Begin Transaction n
In(Ok): Insert Object k in Transaction n
Dn(Ok): Delete Object k from Transaction n
A(Tn): Abort Transaction n
C(TN): Commit Transaction n

1. Tutte le transaction che fanno Abort o Commit prima di CK vanno bene
2. Tutte le transaction che fanno Commit dopo CK vanno in REDO
3. Tutte le transaction che fanno Abort o niente(ne A ne C) dopo CK vanno in REDO

UNDO: DA DESTRA A SINISTRA

Prendo tutte le operazioni fatte, dall’ultima (quella più a destra) fino alla prima (quella più a sinistra) e faccio il contrario:
Ex. $I_3(O_2), \ D_3(O_4)$ → $I_3(O_4), \ D_3(O_2)$ 

REDO:

Prendo tutte le operazioni fatte e le ripeto nello stesso ordine
Ex: $I_3(O_2), \ D_3(O_4)$ → $I_3(O_2), \ D_3(O_4)$

## PRECISIONE E RECALL

PRECISIONE: 

Numero di volte che ho predetto correttamente l’appartenenza di un punto alla classe fratto il numero di volte in cui ho predetto l’appartenenza alla class
→ numero di i per cui y_pred[i] = y_true[i] = classe / numero di i per cui y_pred[i] = classe

RICHIAMO: 

Numero di volte che ho predetto correttamente l’appartenenza di un punto alla classe fratto, numero di volte che un punto appartiene effettivamente alla classe
→ numero di i per cui y_pred[i] = y_true[i] = classe / numero di i per cui y_true[i] = classe

## LOCKING

ISL IXL SL SIXL XL

![Screenshot 2024-02-18 alle 13.59.02.png](Preparazione%20Esame%2007188e8a93b7476e8333a3a4caa3ca08/Screenshot_2024-02-18_alle_13.59.02.png)

[Esame 16/06/2023](Preparazione%20Esame%2007188e8a93b7476e8333a3a4caa3ca08/Esame%2016%2006%202023%20cdd79c5fbdd84c518b2798fb7dec8543.csv)

[Esame 21/02/2023](Preparazione%20Esame%2007188e8a93b7476e8333a3a4caa3ca08/Esame%2021%2002%202023%20252c8e9b9ef64e2ca6b5a6b275d33a01.csv)

[Esame 23/01/2023](Preparazione%20Esame%2007188e8a93b7476e8333a3a4caa3ca08/Esame%2023%2001%202023%201ef4fd7c9d98401092cdd979cb325835.csv)

[Esame 31/08/2022](Preparazione%20Esame%2007188e8a93b7476e8333a3a4caa3ca08/Esame%2031%2008%202022%20b6677538406042c7add0b2abf9edd193.csv)

[Esame 01/07/2022](Preparazione%20Esame%2007188e8a93b7476e8333a3a4caa3ca08/Esame%2001%2007%202022%20a184880572f34e4bae6695faffade339.csv)

[Esame 09/02/2022](Preparazione%20Esame%2007188e8a93b7476e8333a3a4caa3ca08/Esame%2009%2002%202022%20fc1c299d443a4970ab471b2f1f1aa21d.csv)

[Esame 15/06/2021](Preparazione%20Esame%2007188e8a93b7476e8333a3a4caa3ca08/Esame%2015%2006%202021%207dd5104234c3467db7bb2bac762c7603.csv)
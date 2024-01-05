# mysql_storedprocedure_and_cursor
Example of using a stored procedure with cursor on mysql

Supponiamo di avere una tabella prodotti con i campi IDProdotto, NomeProdotto, Prezzo e Categoria. Vogliamo calcolare il prezzo medio dei prodotti di una categoria fornita in input.

Cliccare sul database -> Routine -> Add Routine -> settare il parametro di input categoria_selezionata

BEGIN
    DECLARE done BOOLEAN DEFAULT FALSE;
    DECLARE id_prodotto INT;
    DECLARE nome_prodotto VARCHAR(255);
    DECLARE prezzo_prodotto DECIMAL(10, 2);
    DECLARE somma_prezzi DECIMAL(10, 2) DEFAULT 0;
    DECLARE contatore INT DEFAULT 0;

    -- Dichiarazione del cursore
    DECLARE cursor_prodotti CURSOR FOR
        SELECT IDProdotto, NomeProdotto, Prezzo
        FROM Prodotti
        WHERE Categoria = categoria_selezionata;

    -- Indica cosa fare quando il cursore è finito
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    -- Apertura del cursore
    OPEN cursor_prodotti;

    -- Fetch delle righe e calcolo della somma dei prezzi
    FETCH_LOOP: LOOP
        FETCH cursor_prodotti INTO id_prodotto, nome_prodotto, prezzo_prodotto;
        IF done THEN
            LEAVE FETCH_LOOP;
        END IF;

        -- Calcolo della somma dei prezzi
        SET somma_prezzi = somma_prezzi + prezzo_prodotto;

        -- Incremento del contatore
        SET contatore = contatore + 1;
    END LOOP FETCH_LOOP;

    -- Calcolo della media dei prezzi
    IF contatore > 0 THEN
        SELECT categoria_selezionata AS Categoria, somma_prezzi / contatore AS MediaPrezzi;
    ELSE
        SELECT 'Nessun prodotto trovato nella categoria ' AS Categoria, NULL AS MediaPrezzi;
    END IF;

    -- Chiusura del cursore
    CLOSE cursor_prodotti;
  END

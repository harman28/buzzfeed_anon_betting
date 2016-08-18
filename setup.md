Data from [that Buzzfeed article](https://github.com/BuzzFeedNews/2016-01-tennis-betting-analysis). Get that or,

    wget https://raw.githubusercontent.com/BuzzFeedNews/2016-01-tennis-betting-analysis/master/data/anonymous_betting_data.csv

To setup the PostgreSQL db:

    createdb misc_tennis
    psql -c "CREATE TABLE anon_betting_buzzfeed (
        match_book_uid character varying(64),
        odds_winner_open double precision,
        odds_loser_open double precision,
        odds_winner_close double precision,
        odds_loser_close double precision,
        year double precision,
        implied_prob_winner_open double precision,
        implied_prob_loser_open double precision,
        implied_prob_winner_close double precision,
        implied_prob_loser_close double precision,
        moved_towards_winner boolean,
        book character varying(1),
        loser character varying(64),
        winner character varying(64),
        is_cancelled_or_walkover boolean,
        match_uid character varying(64)
    );" misc_tennis
    psql -c "\COPY anon_betting_buzzfeed FROM anonymous_betting_data.csv DELIMITER ',' CSV HEADER;"

Yes, the year is in double precision. Lol. Alternately, sanitize all the 2009.0s to 2009s in the csv, and stick to integer.
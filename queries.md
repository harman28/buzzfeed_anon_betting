## Steps (1) and (2)
Taken care of by our setup.

## Step (3)
We need an `median_implied_prob` table to store median probabilities calculated for each match by each bookie. Join this with the main matches table to find the valid match books (without anomalous opening odds), and distinct matches for good measure. Also leave out walkovers and cancellations.

    WITH 
    median_implied_prob AS (
        SELECT match_uid,QUANTILE(implied_prob_winner_open, 0.5) AS median_implied_prob_winner_open,QUANTILE(implied_prob_loser_open,0.5) AS median_implied_prob_loser_open 
        FROM anon_betting_buzzfeed GROUP BY match_uid
        ),
    valid_match_books AS (
        SELECT A.match_book_uid, A.match_uid
        FROM anon_betting_buzzfeed A INNER JOIN median_implied_prob B ON A.match_uid=B.match_uid 
        WHERE ABS(A.implied_prob_winner_open-B.median_implied_prob_winner_open)<0.1 
            AND ABS(A.implied_prob_loser_open-B.median_implied_prob_loser_open)<0.1
            AND NOT is_cancelled_or_walkover)
    SELECT COUNT(distinct match_uid) FROM valid_match_books;

Result:

     count 
    -------
     25918
    (1 row)

Close enough. Buzzfeed got 25,993 matches, although I don't have a good explanation for the difference if we're using exactly the same data. I tried changing < to <=, made no difference.

## Step (4)

High-movement matches are defined as those that see the odds shift by more than 10 percent between open and close.

    high_movement_matches AS (
            SELECT DISTINCT  A.match_uid 
            FROM anon_betting_buzzfeed A INNER JOIN valid_match_books B ON A.match_book_uid=B.match_book_uid
            WHERE ABS(implied_prob_winner_close-implied_prob_winner_open)>0.1
                AND ABS(implied_prob_loser_close-implied_prob_loser_open)>0.1
    
        )
    SELECT COUNT(*) FROM high_movement_matches;

Result:

     count 
    -------
     2709
    (1 row)

Again, slightly off. This is 10.4% of the valid matches and 10.2% of all matches. Buzzfeed reported 11% of all matches were 'high-movement'. No idea why this is happening.

## Step (5)

To make the next bit simpler, a table that just stores each match and its loser. Hack.

    losers AS (
        SELECT max(loser) AS loser, match_uid FROM anon_betting_buzzfeed GROUP BY match_uid 
        )

Join that to our high movement matches to get suspicious losses, and find suspicious players, defined as players with more than 10 suspicious losses.

    suspicious_losses AS (
        SELECT A.loser, A.match_uid
        FROM losers A INNER JOIN high_movement_matches B ON A.match_uid=B.match_uid
        ),
    suspicious_players AS (
        SELECT loser
        FROM suspicious_losses 
        GROUP BY loser HAVING COUNT(match_uid)>10
        )
    SELECT COUNT(*) FROM suspicious_players;

Result:

     count 
    -------
      84
    (1 row)

I knew the differences would eventually come to a head. Buzzfeed reports finding just 39 players who met the criteria. I seem to have found 84. Damn.
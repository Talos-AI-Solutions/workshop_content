-- Players table
CREATE TABLE players (
    player_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tournaments table
CREATE TABLE tournaments (
    tournament_id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    buy_in DECIMAL(10, 2),
    tournament_date DATE NOT NULL,
    total_players INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tournament entries (links players to tournaments with their placement)
CREATE TABLE tournament_entries (
    entry_id SERIAL PRIMARY KEY,
    player_id INTEGER NOT NULL REFERENCES players(player_id) ON DELETE CASCADE,
    tournament_id INTEGER NOT NULL REFERENCES tournaments(tournament_id) ON DELETE CASCADE,
    placement INTEGER NOT NULL,
    prize_amount DECIMAL(10, 2),
    UNIQUE(player_id, tournament_id),
    CONSTRAINT valid_placement CHECK (placement > 0)
);

-- Indexes for common queries
CREATE INDEX idx_tournament_entries_player ON tournament_entries(player_id);
CREATE INDEX idx_tournament_entries_tournament ON tournament_entries(tournament_id);
CREATE INDEX idx_tournaments_date ON tournaments(tournament_date);
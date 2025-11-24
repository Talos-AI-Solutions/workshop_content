openapi: 3.0.3
info:
  title: Poker Tournament API
  description: API for managing poker players, tournaments, and tournament results
  version: 1.0.0
  contact:
    name: API Support
    email: support@pokerdb.com

servers:
  - url: https://api.pokerdb.com/v1
    description: Production server
  - url: http://localhost:8000/v1
    description: Development server

paths:
  /players:
    get:
      summary: List all players
      tags:
        - Players
      parameters:
        - name: limit
          in: query
          schema:
            type: integer
            default: 50
            maximum: 100
        - name: offset
          in: query
          schema:
            type: integer
            default: 0
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/Player'
                  total:
                    type: integer
                  limit:
                    type: integer
                  offset:
                    type: integer

    post:
      summary: Create a new player
      tags:
        - Players
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PlayerCreate'
      responses:
        '201':
          description: Player created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Player'
        '400':
          $ref: '#/components/responses/BadRequest'
        '409':
          description: Player with this email already exists

  /players/{playerId}:
    get:
      summary: Get a player by ID
      tags:
        - Players
      parameters:
        - $ref: '#/components/parameters/PlayerId'
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Player'
        '404':
          $ref: '#/components/responses/NotFound'

    put:
      summary: Update a player
      tags:
        - Players
      parameters:
        - $ref: '#/components/parameters/PlayerId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PlayerUpdate'
      responses:
        '200':
          description: Player updated successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Player'
        '400':
          $ref: '#/components/responses/BadRequest'
        '404':
          $ref: '#/components/responses/NotFound'

    delete:
      summary: Delete a player
      tags:
        - Players
      parameters:
        - $ref: '#/components/parameters/PlayerId'
      responses:
        '204':
          description: Player deleted successfully
        '404':
          $ref: '#/components/responses/NotFound'

  /players/{playerId}/tournaments:
    get:
      summary: Get all tournaments for a player
      tags:
        - Players
      parameters:
        - $ref: '#/components/parameters/PlayerId'
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/TournamentEntry'
        '404':
          $ref: '#/components/responses/NotFound'

  /tournaments:
    get:
      summary: List all tournaments
      tags:
        - Tournaments
      parameters:
        - name: limit
          in: query
          schema:
            type: integer
            default: 50
            maximum: 100
        - name: offset
          in: query
          schema:
            type: integer
            default: 0
        - name: from_date
          in: query
          schema:
            type: string
            format: date
        - name: to_date
          in: query
          schema:
            type: string
            format: date
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/Tournament'
                  total:
                    type: integer
                  limit:
                    type: integer
                  offset:
                    type: integer

    post:
      summary: Create a new tournament
      tags:
        - Tournaments
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/TournamentCreate'
      responses:
        '201':
          description: Tournament created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Tournament'
        '400':
          $ref: '#/components/responses/BadRequest'

  /tournaments/{tournamentId}:
    get:
      summary: Get a tournament by ID
      tags:
        - Tournaments
      parameters:
        - $ref: '#/components/parameters/TournamentId'
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Tournament'
        '404':
          $ref: '#/components/responses/NotFound'

    put:
      summary: Update a tournament
      tags:
        - Tournaments
      parameters:
        - $ref: '#/components/parameters/TournamentId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/TournamentUpdate'
      responses:
        '200':
          description: Tournament updated successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Tournament'
        '400':
          $ref: '#/components/responses/BadRequest'
        '404':
          $ref: '#/components/responses/NotFound'

    delete:
      summary: Delete a tournament
      tags:
        - Tournaments
      parameters:
        - $ref: '#/components/parameters/TournamentId'
      responses:
        '204':
          description: Tournament deleted successfully
        '404':
          $ref: '#/components/responses/NotFound'

  /tournaments/{tournamentId}/results:
    get:
      summary: Get all results for a tournament
      tags:
        - Tournaments
      parameters:
        - $ref: '#/components/parameters/TournamentId'
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/TournamentEntry'
        '404':
          $ref: '#/components/responses/NotFound'

  /tournament-entries:
    post:
      summary: Add a player to a tournament with their result
      tags:
        - Tournament Entries
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/TournamentEntryCreate'
      responses:
        '201':
          description: Tournament entry created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/TournamentEntry'
        '400':
          $ref: '#/components/responses/BadRequest'
        '409':
          description: Player already entered in this tournament

  /tournament-entries/{entryId}:
    put:
      summary: Update a tournament entry
      tags:
        - Tournament Entries
      parameters:
        - name: entryId
          in: path
          required: true
          schema:
            type: integer
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/TournamentEntryUpdate'
      responses:
        '200':
          description: Entry updated successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/TournamentEntry'
        '400':
          $ref: '#/components/responses/BadRequest'
        '404':
          $ref: '#/components/responses/NotFound'

    delete:
      summary: Remove a tournament entry
      tags:
        - Tournament Entries
      parameters:
        - name: entryId
          in: path
          required: true
          schema:
            type: integer
      responses:
        '204':
          description: Entry deleted successfully
        '404':
          $ref: '#/components/responses/NotFound'

components:
  parameters:
    PlayerId:
      name: playerId
      in: path
      required: true
      schema:
        type: integer
      description: Unique identifier for a player

    TournamentId:
      name: tournamentId
      in: path
      required: true
      schema:
        type: integer
      description: Unique identifier for a tournament

  schemas:
    Player:
      type: object
      properties:
        player_id:
          type: integer
          example: 1
        name:
          type: string
          example: "John Doe"
        email:
          type: string
          format: email
          example: "john.doe@example.com"
        created_at:
          type: string
          format: date-time
          example: "2024-01-15T10:30:00Z"

    PlayerCreate:
      type: object
      required:
        - name
        - email
      properties:
        name:
          type: string
          maxLength: 100
          example: "John Doe"
        email:
          type: string
          format: email
          maxLength: 255
          example: "john.doe@example.com"

    PlayerUpdate:
      type: object
      properties:
        name:
          type: string
          maxLength: 100
          example: "John Doe"
        email:
          type: string
          format: email
          maxLength: 255
          example: "john.doe@example.com"

    Tournament:
      type: object
      properties:
        tournament_id:
          type: integer
          example: 1
        name:
          type: string
          example: "World Series of Poker 2024"
        buy_in:
          type: number
          format: decimal
          example: 10000.00
        tournament_date:
          type: string
          format: date
          example: "2024-07-15"
        total_players:
          type: integer
          example: 150
        created_at:
          type: string
          format: date-time
          example: "2024-01-15T10:30:00Z"

    TournamentCreate:
      type: object
      required:
        - name
        - tournament_date
      properties:
        name:
          type: string
          maxLength: 200
          example: "World Series of Poker 2024"
        buy_in:
          type: number
          format: decimal
          minimum: 0
          example: 10000.00
        tournament_date:
          type: string
          format: date
          example: "2024-07-15"
        total_players:
          type: integer
          minimum: 0
          example: 150

    TournamentUpdate:
      type: object
      properties:
        name:
          type: string
          maxLength: 200
          example: "World Series of Poker 2024"
        buy_in:
          type: number
          format: decimal
          minimum: 0
          example: 10000.00
        tournament_date:
          type: string
          format: date
          example: "2024-07-15"
        total_players:
          type: integer
          minimum: 0
          example: 150

    TournamentEntry:
      type: object
      properties:
        entry_id:
          type: integer
          example: 1
        player_id:
          type: integer
          example: 42
        player_name:
          type: string
          example: "John Doe"
        tournament_id:
          type: integer
          example: 15
        tournament_name:
          type: string
          example: "World Series of Poker 2024"
        placement:
          type: integer
          example: 3
        prize_amount:
          type: number
          format: decimal
          example: 50000.00

    TournamentEntryCreate:
      type: object
      required:
        - player_id
        - tournament_id
        - placement
      properties:
        player_id:
          type: integer
          example: 42
        tournament_id:
          type: integer
          example: 15
        placement:
          type: integer
          minimum: 1
          example: 3
        prize_amount:
          type: number
          format: decimal
          minimum: 0
          example: 50000.00

    TournamentEntryUpdate:
      type: object
      properties:
        placement:
          type: integer
          minimum: 1
          example: 3
        prize_amount:
          type: number
          format: decimal
          minimum: 0
          example: 50000.00

    Error:
      type: object
      properties:
        error:
          type: string
          example: "Resource not found"
        message:
          type: string
          example: "Player with ID 123 does not exist"
        code:
          type: string
          example: "NOT_FOUND"

  responses:
    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

    BadRequest:
      description: Invalid request
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
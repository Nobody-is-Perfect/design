# Game Description

## Setup

1. The players log in into the lobby, they enter a name. The server responds by returning a token (which functions as the secure `user_id` token). This token will be saved as a cookie. For V1 the players can only reset the user by removing the token.

```
POST /players HTTP/1.1 
Content-Type: application/json

{
  "name": "My PlayerName"
}


HTTP/1.1 201 Created
Content-Type: application/json
Location: /players/c47dbdc6-7778-41fc-a456-5ba74e365f64

{
  "id": "c47dbdc6-7778-41fc-a456-5ba74e365f64"
}
```

2. One player creates a new game. 

```
POST /games HTTP/1.1 
Content-Type: application/json

{
  "name": "My GameName"
}


HTTP/1.1 201 Created
Content-Type: application/json
Location: /games/74f3f450-86c9-4525-bd15-d94bd02bd775

{
  "id": "74f3f450-86c9-4525-bd15-d94bd02bd775"
}
```

The player immediately joins this new game.
```
POST /games/74f3f450-86c9-4525-bd15-d94bd02bd775/join HTTP/1.1 
Content-Type: application/json

{
  "playerId": "c47dbdc6-7778-41fc-a456-5ba74e365f64"
}


HTTP/1.1 202 Accepted
Content-Type: application/json
Location: /games/74f3f450-86c9-4525-bd15-d94bd02bd775
```

Fetching data about the game will reveal the current players and its status:
```
GET /games/74f3f450-86c9-4525-bd15-d94bd02bd775 HTTP/1.1 
Content-Type: application/json

HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "74f3f450-86c9-4525-bd15-d94bd02bd775",
  "name": "My GameName",
  "status": "waiting-for-players",
  "players": ["c47dbdc6-7778-41fc-a456-5ba74e365f64"]
}
```

3. Other players can list the available games, after they've created a new player
```
GET /games HTTP/1.1 
Content-Type: application/json


HTTP/1.1 200 OK
Content-Type: application/json

[{
  "id": "74f3f450-86c9-4525-bd15-d94bd02bd775",
  "name": "My GameName"
}]
```

Subsequently, they are free to join any of the non-started games:
```
POST /games/74f3f450-86c9-4525-bd15-d94bd02bd775/join HTTP/1.1 
Content-Type: application/json

{
  "playerId": "83f8e222-e150-4b85-9026-d4cb65a9ab97"
}


HTTP/1.1 202 Accepted
Content-Type: application/json
Location: /games/74f3f450-86c9-4525-bd15-d94bd02bd775
```

Now, they'll see the current game status:
```
GET /games/74f3f450-86c9-4525-bd15-d94bd02bd775 HTTP/1.1 
Content-Type: application/json

HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "74f3f450-86c9-4525-bd15-d94bd02bd775",
  "name": "My GameName",
  "status": "waiting-for-players",
  "players": ["c47dbdc6-7778-41fc-a456-5ba74e365f64", "83f8e222-e150-4b85-9026-d4cb65a9ab97"]
}
```


Once, all members have joined, any of the members can start the game:
```
POST /games/74f3f450-86c9-4525-bd15-d94bd02bd775/start HTTP/1.1 
Content-Type: application/json


HTTP/1.1 202 Accepted
Content-Type: application/json


{
  "id": "74f3f450-86c9-4525-bd15-d94bd02bd775",
  "name": "My GameName",
  "status": "started",
  "players": ["c47dbdc6-7778-41fc-a456-5ba74e365f64", "83f8e222-e150-4b85-9026-d4cb65a9ab97"]
}
```

After the game has been joined by any player, the web client is supposed to 
subscribe to the following websocket topic in order to be notified about game events which
define the flow of the game:

```javascript

var webSocket = new WebSocket("ws://" + location.hostname + ":" + location.port + "/games/" + gameId);
```

## Flow of a round

1. A player `P` is randomly selected to choose the current topic. (`/game/{game_id}/topics`)
2. This player `P` choses the current topic.
3. The server choses a random question that is from that topic.
4. All players provide an answer to complete the sentence given by the current question.
5. The server provides all players with the options.
6. All players make a choice
7. The server presents who voted for what answer, and highlights the correct answer
8. The server presents the current score
9. The next round starts

## Score computation

Each player gets:.

- 2 points if a different player voted for their lie / answer
- 5 points if the player identified the correct answer

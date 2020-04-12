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
  "id": "c47dbdc6-7778-41fc-a456-5ba74e365f64",
  "name": "My PlayerName"
}
```

2. One player creates a new game. 

```
POST /games HTTP/1.1 
Content-Type: application/json

{
  "name": "My GameName",
  "numberOfRounds": 3
}


HTTP/1.1 201 Created
Content-Type: application/json
Location: /games/74f3f450-86c9-4525-bd15-d94bd02bd775

{
  "id": "74f3f450-86c9-4525-bd15-d94bd02bd775",
  "name": "My GameName",
  "numberOfRounds": 3,
  "status": "waiting-for-players",
}
```

The player immediately joins this new game.
```
POST /games/74f3f450-86c9-4525-bd15-d94bd02bd775/join HTTP/1.1 
Content-Type: application/json

{
  "player": "c47dbdc6-7778-41fc-a456-5ba74e365f64"
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
  "numberOfRounds": 3,
  "status": "waiting-for-players",
  "players": [
    {
      "id": "c47dbdc6-7778-41fc-a456-5ba74e365f64",
      "name": "My PlayerName"
    }
  ]
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
  "name": "My GameName",
  "numberOfRounds": 3,
  "status": "waiting-for-players",
  "players": [
    {
      "id": "c47dbdc6-7778-41fc-a456-5ba74e365f64",
      "name": "My PlayerName"
    }
  ]
}]
```

Subsequently, they are free to join any of the non-started games:
```
POST /games/74f3f450-86c9-4525-bd15-d94bd02bd775/join HTTP/1.1 
Content-Type: application/json

{
  "player": "83f8e222-e150-4b85-9026-d4cb65a9ab97"
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
  "numberOfRounds": 3,
  "status": "waiting-for-players",
  "players": [
    {
      "id": "c47dbdc6-7778-41fc-a456-5ba74e365f64",
      "name": "My PlayerName"
    },
    {
      "id": "83f8e222-e150-4b85-9026-d4cb65a9ab97",
      "name": "My Other Player"
    }
  ]
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
  "numberOfRounds": 3,
  "status": "started",
  "players": [
    {
      "id": "c47dbdc6-7778-41fc-a456-5ba74e365f64",
      "name": "My PlayerName"
    },
    {
      "id": "83f8e222-e150-4b85-9026-d4cb65a9ab97",
      "name": "My Other Player"
    }
  ]
}
```

After the game has been joined by any player, the web client is supposed to 
subscribe to the following websocket topic in order to be notified about game events which
define the flow of the game:

```javascript

var webSocket = new WebSocket("ws://" + location.hostname + ":" + location.port + "/games/" + gameId);
```



## Flow of a round (Events sent from and to the backend to the webclients over the websocket)

1. The round has started. A player `P` is randomly selected to choose the topic of the upcoming question by the backend. 

```json
{
  "event": "round.started",
  "payload": {
    "playerToPerformSelection": "c47dbdc6-7778-41fc-a456-5ba74e365f64",
    "availableTopics": [
      "topic1",
      "topic2",
      "topic3"
    ]
  }
}
```

2. The player `P` choses the current topic by publishing a message to the websocket. All other players do not need to perform any action here:

```json
{
  "event": "round.topic.selection_performed",
  "payload": {
    "chosenTopic": "topic1"
  }
}
```

3. The server choses a random question that is from the selected topic and notifies each subscribed client:
```json
{
  "event": "round.question.completion_requested",
  "payload": {
    "chosenTopic": "topic1",
    "chosenQuestion": "What is the answer to life, the universe and everything? {placeholder}"
  }
}
```

4. All players provide an answer to complete the sentence given by the current question. All players can complete the question with an arbitrary solution not being equal to the correct answer. The correct answer is provided by the backend automatically:
ient:
```json
{
  "event": "round.question.completion_performed",
  "payload": {
    "chosenTopic": "topic1",
    "chosenQuestion": "What is the answer to life, the universe and everything? {placeholder}",
    "chosenAnswer": "42"
  }
}
```
5. The server provides all players with the options.
```json
{
  "event": "round.answer.selection_requested",
  "payload": {
    "chosenTopic": "topic1",
    "chosenQuestion": "What is the answer to life, the universe and everything? {placeholder}",
    "availableAnswers": ["42", "..."]
  }
}
```

6. All players make a choice:
```json
{
  "event": "round.answer.selection_performed",
  "payload": {
    "playerId": "c47dbdc6-7778-41fc-a456-5ba74e365f64",
    "chosenTopic": "topic1",
    "chosenQuestion": "What is the answer to life, the universe and everything? {placeholder}",
    "chosenAnswer": "42"
  }
}
```
7. The server presents who voted for what answer, highlights the correct answer, and provides the latest score:
```json
{
  "event": "round.ended",
  "payload": {
    "chosenTopic": "topic1",
    "chosenQuestion": "What is the answer to life, the universe and everything? {placeholder}",
    "correctAnswer": "42",
    "scores": [
      {
        "player": "c47dbdc6-7778-41fc-a456-5ba74e365f64",
        "score": 10
      },
      {
        "player": "c47dbdc6-7778-41fc-a456-5ba74e365f64",
        "score": 2
      }
    ]
  }
}
```

The same procedure repeats from the beginning as long as there are rounds left to play.

## Game End
Once all rounds have been played, the following event is sent to all clients and the game is removed from the server:

```json
{
  "event": "game.ended",
  "payload": {
    "id": "74f3f450-86c9-4525-bd15-d94bd02bd775",
    "name": "My GameName",
    "numberOfRounds": 3,
    "status": "ended",
    "players": [
      {
        "id": "c47dbdc6-7778-41fc-a456-5ba74e365f64",
        "name": "My PlayerName"
      },
      {
        "id": "83f8e222-e150-4b85-9026-d4cb65a9ab97",
        "name": "My Other Player"
      }
    ],
    "scores": [
      {
        "player": "c47dbdc6-7778-41fc-a456-5ba74e365f64",
        "score": 10
      },
      {
        "player": "c47dbdc6-7778-41fc-a456-5ba74e365f64",
        "score": 2
      }
    ]
  }
}
```


## Score computation

Each player gets:.

- 2 points if a different player voted for their lie / answer
- 5 points if the player identified the correct answer

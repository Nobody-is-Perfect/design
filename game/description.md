# Game Description

## Setup

1. The players log in into the lobby, they enter a name. The server responds by returning a token (which functions as the secure `user_id` token). This token will be saved as a cookie. For V1 the players can only reset the user by removing the token.
2. One player creates a new game. The player immediately joins this new game.
3. Other players are now free to join the game.

## Flow of a round

1. A player `P` is randomly selected to chose the current topic. (`/game/{game_id}/topics`)
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

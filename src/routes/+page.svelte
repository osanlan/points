<script lang="ts">
  import { Card } from "$lib/components/ui/card";
  import { Button } from "$lib/components/ui/button";
  import * as AlertDialog from "$lib/components/ui/alert-dialog/index.js";
  import ScoreIncrementer from "$lib/components/ScoreIncrementer.svelte";
  
  import type { Player } from "$lib/models/players";

  let players: Player[] = $state(
    localStorage.getItem("players")
      ? JSON.parse(localStorage.getItem("players") || "[]")
      : [],
  );

  let breakingTurn: boolean = $state(true);
  let showBreakDialog: boolean = $state(false);
  let playerIndex = $state(0);
  // svelte-ignore state_referenced_locally
  let currentPlayer: Player = $state(players[0]);

  let streak: number = $state(0);

  let pointsPerIncrease: number = $state(1);

  function addPlayer() {
    const newPlayer: Player = {
      id: playerIndex,
      name: `Player ${playerIndex + 1}`,
      score: 0,
      consecutiveFouls: 0,
    };
    playerIndex += 1;

    players = [...players, newPlayer];
    localStorage.setItem("players", JSON.stringify(players));
  }

  function getNextPlayer(players: Player[], player: Player): Player {
    const currentIndex = players.findIndex((p) => p.id === player.id);
    const nextIndex = (currentIndex + 1) % players.length;
    return players[nextIndex];
  }

  const firstPlayerActions = {
    nextPlayer() {
      playerActions.nextPlayer();
      breakingTurn = false;
    },
    incrementScore(points: number) {
      playerActions.incrementScore(points);
      breakingTurn = false;
    },
    decrementScore() {
      playerActions.decrementScore();
      breakingTurn = false;
    },
    breakingFoul(player: Player) {
      player.score -= 2;
    },
  };

  const playerActions = {
    nextPlayer() {
      currentPlayer.consecutiveFouls = 0;
      currentPlayer = getNextPlayer(players, currentPlayer);
      localStorage.setItem("players", JSON.stringify(players));
    },

    incrementScore(points: number) {
      currentPlayer.score += points;
      if (currentPlayer.consecutiveFouls) {
        currentPlayer.consecutiveFouls = 0; // Reset consecutive fouls on scoring
      }
      streak += points;
      pointsPerIncrease = 1;
      updatePlayer(currentPlayer);
    },

    decrementScore() {
      currentPlayer.score -= 1;
      currentPlayer.consecutiveFouls = currentPlayer.consecutiveFouls + 1;
      streak = 0;
      if (currentPlayer.consecutiveFouls >= 3) {
        alert(
          `${currentPlayer.name} has made 3 consecutive fouls! 15 points deducted. Re-rack the balls!`,
        );
        currentPlayer.score -= 15;
        currentPlayer.consecutiveFouls = 0;
      }
      updatePlayer(currentPlayer);
      currentPlayer = getNextPlayer(players, currentPlayer);
    },
  };

  function removePlayer(player: Player) {
    players = players.filter((p) => p !== player);
  }

  function updatePlayer(player: Player) {
    players = players.map((p) => (p.id === player.id ? player : p));
    localStorage.setItem("players", JSON.stringify(players));
  }

  function resetGame() {
    players = [];
    localStorage.removeItem("players");
    playerIndex = 0;
    addPlayer();
    currentPlayer = players[0];
    streak = 0;
    breakingTurn = true;
    showBreakDialog = false;
  }

  let currentPlayerActions = firstPlayerActions;
</script>

<Card class="flex flex-col items-center">
  <ul class="list-none flex flex-col items-center">
    {#each players as player}
    <li>
      <Card class="w-full flex flex-row items-center py-1">
        <span class="flex-1 py-1">
          {player.name} ({player.id}) - Score: {player.score}
          {#if player.consecutiveFouls > 0}
          - Fouls: {@html "&#10060;".repeat(player.consecutiveFouls)}
          {/if}
        </span>
      </Card>
    </li>
    {/each}
  </ul>
</Card>

{#if players.length > 0}
<Card class="flex flex-col items-center">
  <span class="text-2xl font-bold mb-4">Current player: {currentPlayer.name}</span>
    
  <ScoreIncrementer {currentPlayerActions} {pointsPerIncrease} />
    
  <Button
    onclick={() => currentPlayerActions.nextPlayer()}
    class="bg-blue-500 active:bg-blue-700 hover:bg-blue-700 text-white transition-colors w-[80%] h-[4rem]"
    >
    Next (+0)
  </Button>
  
  <Button
    onclick={() => currentPlayerActions.decrementScore()}
    class="bg-yellow-500 active:bg-yellow-700 hover:bg-yellow-700 text-white transition-colors w-[80%] h-[4rem]"
  >
    Foul (-1)
  </Button>

{#if breakingTurn}
<AlertDialog.Root bind:open={showBreakDialog}>
<!-- AlertDialog for accepting break -->
  <AlertDialog.Trigger class="w-full h-full flex items-center justify-center">
    <Button
    class="bg-orange-500 active:bg-orange-700 hover:bg-orange-700 text-white transition-colors w-[80%] h-[4rem]"
    >
      Breaking foul (-2)
    </Button>
  </AlertDialog.Trigger>
  <AlertDialog.Content interactOutsideBehavior="close">
    <AlertDialog.Header>
      <AlertDialog.Title>Breaking Foul!</AlertDialog.Title>
      <AlertDialog.Description>
        {currentPlayer.name} has made a breaking foul!
        {getNextPlayer(players, currentPlayer).name} can accept the balls
        in position or require {currentPlayer.name} to play another opening
        break shot.
      </AlertDialog.Description>
    </AlertDialog.Header>
    <AlertDialog.Footer>
      <AlertDialog.Cancel>Cancel</AlertDialog.Cancel>
      <AlertDialog.Action onclick={() => {
        currentPlayer.score -= 2;
        showBreakDialog = false;
      }}>
        Re-Break
      </AlertDialog.Action>
      <AlertDialog.Action onclick={() => {
        currentPlayer.score -= 2;
        currentPlayerActions.nextPlayer()
      }}>
        Accept
      </AlertDialog.Action>
    </AlertDialog.Footer>
  </AlertDialog.Content>
</AlertDialog.Root>
{/if}
</Card>
{/if}
              
              <!-- <AlertDialog.Root>
                <AlertDialog.Trigger>
                  <Button
                  class="bg-gray-500 hover:bg-gray-800 text-black text-xl px-1"
                  >
                    Remove Player
                </Button>
              </AlertDialog.Trigger>
              <AlertDialog.Content>
              <AlertDialog.Header>
                <AlertDialog.Title>Remove Player</AlertDialog.Title>
                <AlertDialog.Description>
                  Are you sure you want to remove {currentPlayer.name}? All scores will
                  be lost.
                </AlertDialog.Description>
              </AlertDialog.Header>
              <AlertDialog.Footer>
                <AlertDialog.Cancel>Cancel</AlertDialog.Cancel>
                <AlertDialog.Action onclick={() => removePlayer(currentPlayer)}>
                  Remove
                </AlertDialog.Action>
              </AlertDialog.Footer>
            </AlertDialog.Content>
          </AlertDialog.Root> -->



  <ul class="list-none flex flex-col items-center mt-4">
    <li>
      <Button class="min-w-[180px] min-h-[80px] text-2xl" onclick={() => addPlayer()}>Add Player</Button>
    </li>
    
    <li> 
      <Button class="p-1 m-5" onclick={() => resetGame()} variant="destructive">
        Reset Game
      </Button>
    </li>
  </ul>
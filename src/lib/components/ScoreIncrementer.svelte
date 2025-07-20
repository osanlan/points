<script lang="ts">
  import * as ContextMenu from "$lib/components/ui/context-menu/index.js";
  import { Button } from "$lib/components/ui/button";

  let { currentPlayerActions, pointsPerIncrease } =
    $props<any>();

  function increasePointsPerIncrease() {
    pointsPerIncrease += 1;
  }

  function decreasePointsPerIncrease() {
    if (pointsPerIncrease > 1) {
      pointsPerIncrease -= 1;
    }
  }
</script>

<ContextMenu.Root>
  <ContextMenu.Trigger class="w-full h-full flex items-center justify-center">
    <Button
      onclick={() => currentPlayerActions.incrementScore(pointsPerIncrease)}
      class="bg-green-500 active:bg-green-700 hover:bg-green-700 text-white transition-colors w-[80%] h-[4rem]"
    >
      Score (+{pointsPerIncrease})
    </Button>
  </ContextMenu.Trigger>
  <ContextMenu.Content class="min-w-1">
    <ContextMenu.Item class="min-w-1">
      <Button
        onclick={() => increasePointsPerIncrease()}
        class="bg-green-500 active:bg-green-700 hover:bg-green-700 text-white transition-colors"
      >
        {@html "&#9650"} (+1)
      </Button>
    </ContextMenu.Item>
    <ContextMenu.Item class="min-w-1">
      <Button
        onclick={() => decreasePointsPerIncrease()}
        class="bg-red-500 active:bg-red-700 hover:bg-red-700 text-white transition-colors"
      >
        {@html "&#9660"} (-1)
      </Button>
    </ContextMenu.Item>
  </ContextMenu.Content>
</ContextMenu.Root>

<script lang="ts">
  import PersonBadge from "src/app/shared/PersonBadge.svelte"
  import {getPersonWithFallback} from "src/agent/db"

  export let pubkeys
</script>

<div class="flex flex-col gap-2 notification-section">
  <slot />
  <div class="flex flex-col gap-1">
    {#each pubkeys as pubkey}
      <PersonBadge person={getPersonWithFallback(pubkey)} />
    {/each}
  </div>
</div>

<script>
  import cx from "classnames"
  import {Link} from "svelte-routing"
  import {killEvent} from "src/util/html"
  import {displayPerson} from "src/util/nostr"
  import {routes} from "src/app/state"
  import PersonCircle from "src/app/shared/PersonCircle.svelte"

  export let person
  export let inert = false
</script>

{#if inert}
  <span class={cx($$props.class, "relative z-10 flex items-center gap-2")}>
    <PersonCircle {person} />
    <span class="text-lg font-bold">{displayPerson(person)}</span>
  </span>
{:else}
  <Link
    to={routes.person(person.pubkey)}
    class={cx($$props.class, "relative z-10 flex items-center gap-2 person-link-container")}
    on:click={killEvent}>
    <PersonCircle {person} />
    <span class="text-lg font-bold">{displayPerson(person)}</span>
  </Link>
{/if}

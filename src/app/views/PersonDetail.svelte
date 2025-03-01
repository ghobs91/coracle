<script lang="ts">
  import {last, identity} from "ramda"
  import {navigate} from "svelte-routing"
  import {log} from "src/util/logger"
  import {parseHex} from "src/util/html"
  import {displayPerson, toHex} from "src/util/nostr"
  import {theme, getThemeColor} from "src/partials/state"
  import Tabs from "src/partials/Tabs.svelte"
  import Content from "src/partials/Content.svelte"
  import Spinner from "src/partials/Spinner.svelte"
  import PersonActions from "src/app/shared/PersonActions.svelte"
  import PersonNotes from "src/app/shared/PersonNotes.svelte"
  import PersonLikes from "src/app/shared/PersonLikes.svelte"
  import PersonRelays from "src/app/shared/PersonRelays.svelte"
  import pool from "src/agent/pool"
  import {sampleRelays, getPubkeyWriteRelays} from "src/agent/relays"
  import {getPersonWithFallback, watch} from "src/agent/db"
  import {routes} from "src/app/state"
  import PersonCircle from "src/app/shared/PersonCircle.svelte"
  import PersonAbout from "src/app/shared/PersonAbout.svelte"
  import PersonStats from "src/app/shared/PersonStats.svelte"

  export let npub
  export let activeTab
  export let relays = []
  export let fromNostrBand;
  export let indexedRelays;

  const tabs = ["notes", "likes", pool.forceUrls.length === 0 && "relays"].filter(identity)
  const pubkey = toHex(npub)
  const person = watch("people", () => getPersonWithFallback(pubkey))
  const nostrBandApi = `https://api.nostr.band/nostr?method=search&count=1&q=${pubkey}`

  let loading = true
  let rgb, rgba

  $: ownRelays = getPubkeyWriteRelays(pubkey)
  // $: indexedRelays = getRelaysFromNostrBand()
  $: relays = sampleRelays(relays.concat(ownRelays))


  
  const nostrbandProfileInfo = async (): Promise <any[]> => {
    const res = await fetch(nostrBandApi, {
              method: "GET"
            })
    const json = await res.json()
    return json;
  }

  const getRelaysFromNostrBand = async (): Promise<any> => {
      nostrbandProfileInfo().then((response) => {
        const relayKeys = response["people"][0]["relays"];
        const relayListFromKeys = [];
        relayKeys.forEach(key => {
          relayListFromKeys.push({url: response["relays"][key]})
        });
        console.log(`final relayListFromKeys: ${relayListFromKeys}`);
        fromNostrBand = true;
        indexedRelays = relayListFromKeys;
        return indexedRelays;
      });
  }


  $: {
    const color = parseHex(getThemeColor($theme, "gray-8"))

    rgba = `rgba(${color.join(", ")}, 0.4)`
    rgb = `rgba(${color.join(", ")})`
  }

  log("Person", npub, $person)

  document.title = displayPerson($person)

  const setActiveTab = tab => navigate(routes.person(pubkey, tab))
  console.log('running PersonDetail');
</script>

<div
  class="absolute left-0 h-64 w-full"
  style="z-index: -1;
         background-size: cover;
         background-image:
          linear-gradient(to bottom, {rgba}, {rgb}),
          url('{$person.kind0?.banner}')" />

<Content>
  <div class="flex gap-4 text-gray-1">
    <PersonCircle person={$person} size={16} class="sm:h-32 sm:w-32" />
    <div class="flex flex-grow flex-col gap-4">
      <div class="flex items-start justify-between gap-4">
        <div class="flex flex-grow flex-col gap-2">
          <div class="flex items-center gap-2">
            <h1 class="text-2xl">{displayPerson($person)}</h1>
          </div>
          {#if $person.verified_as}
            <div class="flex gap-1 text-sm">
              <i class="fa fa-user-check text-accent" />
              <span class="text-gray-1">{last($person.verified_as.split("@"))}</span>
            </div>
          {/if}
        </div>
        <PersonActions person={$person} />
      </div>
      <PersonAbout person={$person} />
      <PersonStats person={$person} />
    </div>
  </div>

  <Tabs {tabs} {activeTab} {setActiveTab} />

  {#if activeTab === "notes"}
    <PersonNotes {pubkey} {relays} />
  {:else if activeTab === "likes"}
    <PersonLikes {pubkey} {relays} />
  {:else if activeTab === "relays"}
    {#await getRelaysFromNostrBand()}
    <!-- <Spinner /> -->
      {:then}
        {#if indexedRelays}
            <PersonRelays indexedRelays={indexedRelays} relays={null} fromNostrBand={true} />
        {:else}
            <PersonRelays indexedRelays={null} relays={ownRelays} fromNostrBand={false} />
        {/if}
      <!-- {:else if loading}
        <Spinner />
      {:else}
        <Content size="lg" class="text-center">Unable to show network for this person.</Content>
      {/if} -->
    {/await}
  {/if}
</Content>

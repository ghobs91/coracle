<script>
    import {pluck, find} from "ramda"
    import {Tags, displayRelay} from "src/util/nostr"
    import {modal, toast} from "src/partials/state"
    import Heading from "src/partials/Heading.svelte"
    import PersonBadge from "src/app/shared/PersonBadge.svelte"
    import Content from "src/partials/Content.svelte"
    import Button from "src/partials/Button.svelte"
    import Input from "src/partials/Input.svelte"
    import MultiSelect from "src/partials/MultiSelect.svelte"
    import {searchTopics, searchPeople, searchRelays, getPersonWithFallback} from "src/agent/db"
    import user from "src/agent/user"
    import {warn} from "src/util/logger"
    import {navigate} from "svelte-routing"
  
    export let topic;
    let list;
    list = find(e => e.id !== list?.id && Tags.from(e).getMeta("d") === "agora_muted_words", user.getLists());
  
    const tags = Tags.wrap(list?.tags || [])
    console.log('tags: ', tags);
    console.log('list: ', list);
  
    let values = {
      name: tags.getMeta("d") || "",
      params: tags.type(["t", "p"]).all(),
      relays: tags.type("r").all(),
    }
  
    console.log('values: ', values)
  
    const search = q => {
        return $searchTopics(q)
          .slice(0, 5)
          .map(({name}) => ["t", name])
    }
  
    const _searchRelays = q => pluck("url", $searchRelays(q)).map(url => ["r", url])
  
    const submit = async () => {
      if (!values.name) {
        values.name = "agora_muted_words";
      }
      const existingList = find(e => e.id !== list?.id && Tags.from(e).getMeta("d") === values.name, user.getLists());
      if (
        find(e => e.id !== list?.id && Tags.from(e).getMeta("d") === values.name, user.getLists())
      ) {
        list.id = existingList.id;
      }
  
      if (values.params.length < 1) {
        return toast.show("error", "Add at least 1 word to mute!")
      }
  
      const {name, params, relays} = values
  
      user.putList(list?.id, name, params, relays)
      toast.show("info", "Your list has been saved!")
      navigate("/notes")
      // modal.pop()
    }
  </script>
  
  <form on:submit|preventDefault={submit}>
    <Content>
      <Heading class="text-center">Mute Topics You Don't Want To See</Heading>
      <div class="flex w-full flex-col gap-8">
        <div class="flex flex-col gap-1 list-name-field">
          <strong>Name</strong>
          <Input bind:value={values.name} />
          <p class="text-sm text-gray-4">
            Lists are identified by their name, so this has to be unique.
          </p>
        </div>
        <div class="flex flex-col gap-1">
          <strong>Your Muted Words</strong>
          <MultiSelect {search} bind:value={values.params}>
            <div slot="item" let:item>
              {#if item[0] === "p"}
                <div class="-my-1">
                  <PersonBadge inert person={getPersonWithFallback(item[1])} />
                </div>
              {:else}
                <strong>{item[1]}</strong>
              {/if}
            </div>
          </MultiSelect>
        </div>
        <!-- <div class="flex flex-col gap-1">
          <strong>Relays</strong>
          <MultiSelect search={_searchRelays} bind:value={values.relays}>
            <div slot="item" let:item>
              {displayRelay({url: item[1]})}
            </div>
          </MultiSelect>
          <p class="text-sm text-gray-4">
            Select which relays to limit this list to. If you leave this blank, your default relays
            will be used.
          </p>
        </div> -->
        <Button type="submit" class="text-center">Save</Button>
      </div>
    </Content>
  </form>
  
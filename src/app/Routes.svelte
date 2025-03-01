<script lang="ts">
  import {Route} from "svelte-routing"
  import {onReady} from "src/agent/db"
  import {base64DecodeOrPlainWebSocketURL} from "src/util/misc"
  import EnsureData from "src/app/EnsureData.svelte"
  import Notifications from "src/app/views/Notifications.svelte"
  import Bech32Entity from "src/app/views/Bech32Entity.svelte"
  import ChatDetail from "src/app/views/ChatDetail.svelte"
  import ChatList from "src/app/views/ChatList.svelte"
  import Feeds from "src/app/views/Feeds.svelte"
  import UserKeys from "src/app/views/UserKeys.svelte"
  import Login from "src/app/views/Login.svelte"
  import Logout from "src/app/views/Logout.svelte"
  import MessagesDetail from "src/app/views/MessagesDetail.svelte"
  import MessagesList from "src/app/views/MessagesList.svelte"
  import NotFound from "src/app/views/NotFound.svelte"
  import PersonDetail from "src/app/views/PersonDetail.svelte"
  import Search from "src/app/views/Search.svelte"
  import RelayDetail from "src/app/views/RelayDetail.svelte"
  import RelayList from "src/app/views/RelayList.svelte"
  import UserProfile from "src/app/views/UserProfile.svelte"
  import UserSettings from "src/app/views/UserSettings.svelte"
  import { inject } from '@vercel/analytics'
  import Discover from "./views/Discover.svelte"
  import PopularFeed from "./shared/PopularFeed.svelte"
  import TopUsers from "./views/TopUsers.svelte"
  import TopicFeed from "src/app/views/TopicFeed.svelte"
  import FediverseTopicFeed from "./views/FediverseTopicFeed.svelte"
    import PopularBluesky from "./shared/PopularBluesky.svelte"
    import FediverseTopicsFeed from "./views/FediverseTopicsFeed.svelte"

  let ready = false
  let signup
  inject({ mode: 'production' });

  onReady(() => {
    ready = true
  })
</script>

{#if ready}
  <div class="routes-container pt-16 text-gray-2 lg:ml-56">
    <Route path="/notifications" component={Notifications} />
    <Route path="/notifications/:activeTab" component={Notifications} />
    <Route path="/discover" let:params>
      <!-- <PopularBluesky/> -->
      <PopularFeed/>
    </Route>
    <!-- <Route path="/topusers" let:params>
      <TopUsers/>
    </Route> -->
    <Route path="/search">
      <EnsureData enforcePeople={false}>
        <Search />
      </EnsureData>
    </Route>
    <Route path="/notes" let:params>
      <EnsureData>
        <Feeds />
      </EnsureData>
    </Route>
    <Route path="/people/:npub/:activeTab" let:params>
      {#key params.npub}
        <PersonDetail npub={params.npub} activeTab={params.activeTab} />
      {/key}
    </Route>
    <Route path="/chat" component={ChatList} />
    <Route path="/chat/:entity" let:params>
      {#key params.entity}
        <ChatDetail entity={params.entity} />
      {/key}
    </Route>
    <Route path="/messages">
      <MessagesList activeTab="messages" />
    </Route>
    <Route path="/requests">
      <MessagesList activeTab="requests" />
    </Route>
    <Route path="/messages/:entity" let:params>
      {#key params.entity}
        <MessagesDetail entity={params.entity} />
      {/key}
    </Route>
    <Route path="/keys" component={UserKeys} />
    <Route path="/relays" component={RelayList} />
    <Route path="/relays/:b64OrUrl" let:params>
      {#key params.b64url}
        <RelayDetail url={base64DecodeOrPlainWebSocketURL(params.b64OrUrl)} />
      {/key}
    </Route>
    <Route path="/topic/:topic" let:params>
      <!-- <TopicFeed topic={params.topic} /> -->
      <FediverseTopicFeed topicFromFilter={params.topic}/>
    </Route>
    <Route path="/profile" component={UserProfile} />
    <Route path="/settings" component={UserSettings} />
    <Route path="/login" component={Login} />
    <Route path="/logout" component={Logout} />
    <Route path="/:entity" let:params>
      {#key params.entity}
        <Bech32Entity entity={params.entity} />
      {/key}
    </Route>
    <Route path="*" component={NotFound} />
  </div>
{/if}

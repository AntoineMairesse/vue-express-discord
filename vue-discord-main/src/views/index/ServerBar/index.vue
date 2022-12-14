<template>
  <div class="hide-scrollbar overflow-scroll">
    <div class="flex flex-col gap-y-4 items-center py-5">

      <!-- MODALS -->
      <ServerCreation ref="serverCreation"/>
      <ServerUpdate ref="serverUpdate"/>
      <ServerDeletion ref="serverDeletion"/>
      <ServerInvite ref="serverInvite"/>
      <ServerLeave ref="serverLeave"/>

      <ContextMenu
          v-model:item="contextMenuItem"
          :items="contextMenuItems"
          :condition="true"
          ref="contextMenu"
          @inviteToServer="(item) => $refs.serverInvite.toggle(item)"
          @editServer="(item) => $refs.serverUpdate.toggle(item)"
          @deleteServer="(item) => $refs.serverDeletion.toggle(item)"
          @leaveServer="(item) => $refs.serverLeave.toggle(item)"
      />

      <item icon="ri-group-line" i18nKey="private_messages"/>

      <n-tooltip v-for="server in servers" trigger="hover" placement="right">
        <template #trigger>
          <div
              class="w-12 h-12 bg-cover bg-center cursor-pointer rounded-full	"
              :style="{'background-image': serverPhotoURL(server)}"
              :class="{'border-2 border-green': selectedServer?._id === server._id}"
              @contextmenu.prevent="$refs.contextMenu.handleContextMenu($event, server)"
              @click="setSelectedServer(server)"
          />
        </template>
        {{ server.name }}
      </n-tooltip>

      <item icon="ri-add-line" i18nKey="server.add" @itemClick="$refs.serverCreation.toggle()"/>
      <item icon="ri-compass-3-fill" i18nKey="server.explore"/>

    </div>
  </div>
</template>

<script>
import {mapActions, mapGetters, mapState} from 'vuex';
import {NTooltip} from 'naive-ui';
import ServerCreation from '../dialogs/server/ServerCreation.vue';
import ServerUpdate from '../dialogs/server/ServerUpdate.vue';
import ServerDeletion from '../dialogs/server/ServerDeletion.vue';
import ServerInvite from '../dialogs/server/ServerInvite.vue';
import ServerLeave from '../dialogs/server/ServerLeave.vue';
import serverPhotoURL from "../../../mixins/serverPhotoURL";
import ContextMenu from "../../../components/ContextMenu.vue";
import Item from "./item.vue";

export default {
  name: 'ServerBar',
  mixins: [serverPhotoURL],
  mounted() {
    this.getServers();
  },
  components: {
    Item,
    ContextMenu,
    ServerLeave,
    ServerInvite,
    ServerDeletion,
    ServerUpdate,
    ServerCreation,
    NTooltip,
  },
  data() {
    return {
      contextMenuItem: null
    };
  },
  computed: {
    ...mapGetters('server', ['permissionsForServer']),
    ...mapState('auth', ['user']),
    ...mapState('server', ['servers', 'selectedServer']),
    permissions(){
      return this.permissionsForServer(this.contextMenuItem?._id)
    },
    contextMenuItems() {
      return [
        {
          key: 'server.context_menu.invite',
          event: 'inviteToServer',
          condition: true
        },
        {
          key: 'server.context_menu.edit',
          event: 'editServer',
          condition: (this.permissions?.editServer || this.contextMenuItem?.creator === this.user?._id)
        },
        {
          key: 'server.context_menu.delete',
          warning: true,
          event: 'deleteServer',
          condition: (this.permissions?.deleteServer || this.contextMenuItem?.creator === this.user?._id)
        },
        {
          key: 'server.leave',
          warning: true,
          event: 'leaveServer',
          condition: (this.contextMenuItem?.creator !== this.user?._id)
        }
      ]
    }
  },
  methods: {
    ...mapActions('server', ['getServers', 'setSelectedServer', 'deleteServer'])
  },
};
</script>

<style scoped>
</style>

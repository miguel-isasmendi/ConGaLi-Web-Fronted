<template>
  <div class='available-games-page'>
    <div id="fake-nav">
      <a @click.prevent='toggleCreateGameModal'>{{ $t("availableGames.createGame") }}</a>
      <create-game-modal v-if='showCreateGameModal' v-on:accepted='createGame' v-on:cancel='toggleCreateGameModal'/>
      <join-game-modal v-if='gameDescriptorSelected' v-on:accepted='joinGame' v-on:cancel='toggleJoinGameModal'/>
    </div>

    <div id="available-games-toolbar">
      <a class='available-games-toolbar-action actionButton' @click.prevent='refreshAvailableGamesList'>{{ $t("label.refreshList") }}</a>
    </div>

    <table class='game-selection-table noselect' draggable='false'>
      <tr class='game-selection-header noselect'>
        <th>{{ $t("availableGames.gameName") }}</th>
        <th>{{ $t("label.creationDate") }}</th>
        <th>{{ $t("availableGames.ownerId") }}</th>
        <th>{{ $t("availableGames.usersConnected") }}</th>
      </tr>
      <template v-for="(gameDescriptor, key, index) in gameList">
        <tr class='game-selection-item' :class="{'unevenItem': key % 2 === 0}" draggable='false'>
          <td>{{gameDescriptor.name}}</td>
          <td>{{gameDescriptor.createdOn}}</td>
          <td>{{gameDescriptor.ownerId}}</td>
          <td>{{Object.keys(gameDescriptor.users).length}}</td>
          <td>
            <a class='game-selection-item-button actionButton' @click.prevent='gameDescriptorSelected = gameDescriptor'>{{ $t("label.join") }}</a>
          </td>
        </tr>
      </template>
    </table>
  </div>
</template>

<script>

import CreateGameModal from './modal/CreateGameModal.vue'
import JoinGameModal from './modal/JoinGameModal.vue'

export default {
  name: 'available-games-page',
  components: {
    CreateGameModal,
    JoinGameModal
  },
  methods: {
    toggleCreateGameModal: function () {
      this.$data.showCreateGameModal = !this.$data.showCreateGameModal
    },
    toggleJoinGameModal: function () {
      this.$data.gameDescriptorSelected = null
    },
    createGame: function (newGameSubComponentData) {
      this.toggleCreateGameModal()
      let newGameData = {
        name: newGameSubComponentData.name,
        refreshInterval: newGameSubComponentData.refreshInterval,
        resolution: newGameSubComponentData.resolution,
        user: {
          id: this.cgStorage.readLocalUserData().id,
          color: newGameSubComponentData.color.hex.substr(1)
        }
      }

      this.$socket.emit('createGame', newGameData)

      console.log('data sent to server: ' + JSON.stringify(newGameData))
    },
    saveOwnGameIntoDataLocalStorage: function (gameData) {
      this.cgStorage.saveOwnGameData(gameData)
    },
    saveAlienGameDataIntoLocalStorage: function (gameData) {
      this.cgStorage.saveAlienGameData(gameData)
    },
    joinGame: function (joinGameSubComponentData) {
      let data = {
        game: this.$data.gameDescriptorSelected,
        user: {
          id: this.cgStorage.readLocalUserData().id,
          color: joinGameSubComponentData.color.hex.substr(1)
        }
      }

      this.toggleJoinGameModal()
      this.$socket.emit('joinGame', data)
    },
    refreshAvailableGamesList: function () {
      this.$socket.emit('getAvailableGames', {user: { id: this.cgStorage.readLocalUserData().id }})
    }
  },
  socket: {
    events: {
      gameCreated (gameData) {
        console.log(`Game creation confirmed with data: ${JSON.stringify(gameData)}`)

        console.log('Saving data into storage...')
        this.saveOwnGameIntoDataLocalStorage(gameData)

        console.log('Redirecting to game page')
        this.$router.push(`/user/${gameData.ownerId}/game/${gameData.id}`)
      },
      joinedToGame (gameData) {
        console.log(`Successful joined game ${JSON.stringify(gameData)}`)

        console.log('Saving data into storage...')
        this.saveAlienGameDataIntoLocalStorage(gameData)

        console.log('Redirecting to game page')
        this.$router.push(`/user/${gameData.ownerId}/game/${gameData.id}`)
      },
      receiveAvailableGames (gameDescriptors) {
        this.$data.gameList = gameDescriptors
      }
    }
  },
  data () {
    return {
      showCreateGameModal: false,
      gameDescriptorSelected: null,
      gameList: []
    }
  },
  mounted () {
    this.refreshAvailableGamesList()
  }
}
</script>

<style scoped>
.game-selection-table {
  width: 100%;
}

.game-selection-table th {
  padding: 0.5em;
  vertical-align: middle;
  background-color: darkGrey;
  cursor: default;
  color: #FFFFFF;
}

.game-selection-item {
  width: 96%;
  text-align: left;
  padding: 0.5em;
  vertical-align: middle;
}

.game-selection-item td {
  padding: 0.5em;
}

.unevenItem {
  background-color: lightGray;
}

.game-selection-item-button {
  float:right;
  margin-top:-8px;
  margin-bottom:-8px;
  margin-right:-8px;
}

.available-games-toolbar {
  width: 96%;
  padding: 2em;
}

.available-games-toolbar-action {
  float: right;
}
</style>

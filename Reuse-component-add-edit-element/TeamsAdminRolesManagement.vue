<template>
  <v-card class="mt-4">
    <card-title
      icon="mdi-account-group-outline"
      :title="$t('Administration.RolesManagement')"
    >
      <v-spacer />
      <v-btn
        small
        :title="$t('Common.Add')"
        color="primary"
        depressed
        fab
        class="mt-4"
        @click="addRole"
      >
        <v-icon>mdi-plus</v-icon>
      </v-btn>
    </card-title>
    <v-card-text>
      <v-row>
        <v-col cols="12" sm="6" md="4" offset-md="4" offset-sm="1"> </v-col>
      </v-row>
      <v-data-table
        :items="list"
        :headers="headers"
        :footer-props="{
          itemsPerPageText: $t('Administration.Roles'),
          itemsPerPageAllText: $t('Administration.allRoles'),
          itemsPerPageOptions: [5, 10, 25, 50, 100],
        }"
        item-key="name"
      >
        <template v-slot:item.actions="{ item }">
          <v-container>
            <v-row>
              <v-col cols="6">
                <v-btn
                  icon
                  color="primary"
                  :title="$t('Common.Update')"
                  @click="editRole(item)"
                >
                  <v-icon small> mdi-pencil </v-icon>
                </v-btn>
              </v-col>
              <v-col cols="6">
                <team-roles-delete-role :item="item" @deleteRole="deleteRole" />
              </v-col>
            </v-row>
          </v-container>
        </template>
        <template v-slot:item.scopes="{ item }">
          <v-chip v-for="(scope, index) in item.scopes" :key="index">{{
            scope
          }}</v-chip>
        </template>

        <template #no-data>
          <div class="text-center">{{ $t("Parts.Empty") }}</div>
        </template>
      </v-data-table>

      <team-roles-add-role
        v-model="roleEditDialog"
        @save="save"
        :scopes="scopes"
        :element.sync="newRole"
      />
    </v-card-text>
  </v-card>
</template>

<script lang="ts">
import { Component, mixins } from 'nuxt-property-decorator'
import BaseMixin from '~/components/mixins/base';
import AuthMixin from '~/components/mixins/auth';
import TeamsMixin from '../mixins/teams';

@Component
export default class TeamsAdminRolesManagement extends mixins(BaseMixin, AuthMixin, TeamsMixin) {


  headers = [
    { text: this.$i18n.t('Administration.Name'), value: 'name', align: 'left', configable: false, visible: true, sortable: true },
    { text: this.$i18n.t('Administration.Scopes'), value: 'scopes', align: 'left', configable: false, visible: true, sortable: true },
    { text: this.$t('Parts.Actions'), value: 'actions', align: 'left', configable: false, visible: true, sortable: false }
  ]

  get list () {
    return this.$store.getters['teams/server/admin/getRoles'] ?? []
  }

  get scopes () {
    return this.$store.getters['teams/server/roles/getScopes'] ?? []
  }

  // Создание новой роли
  roleEditDialog: boolean = false
  newRole: { id?: string, name: string, scopes: string[], team_id: string } = { name: '', scopes: [], team_id: '' }

  addRole () {
    this.roleEditDialog = true
    this.newRole = { name: '', scopes: [], team_id: '' }
  }

  editRole (item: any) {
    this.roleEditDialog = true
    this.newRole = { ...item }
  }

  save (val: { id?: string, name: string, scopes: string[], team_id: string }) {
    this.roleEditDialog = false
    val.team_id = this.adminCurrentTeam?.id ?? ''

    this.$socket.emit('server.admin.roles.post_role', val)
    this.$socket.emit('server.admin.roles.list', { team_id: val.team_id }, { action: 'teams/server/admin/addRoles' })
    this.newRole = { name: '', scopes: [], team_id: '' }
    this.$emit('updateUsersList')
  }

  deleteRole (obj: {}) {
    this.$socket.emit('server.admin.roles.delete_role', obj, { action: 'teams/server/admin/deleteRole' })
  }

}
</script>

<style>
</style>

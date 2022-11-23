<!--
Красивое отображение v-autocomplete в виде списка с аватаром, подзаголовками
-->

<template>
  <span>
    <v-btn
      small
      :title="$t('Common.Add')"
      color="primary"
      depressed
      fab
      @click.stop="addItem()"
      class="mt-4"
    >
      <v-icon>mdi-plus</v-icon>
    </v-btn>

    <v-dialog v-model="isDialog" max-width="500px">
      <v-card class="pa-3">
        <card-title class="text-h6" icon="mdi-account-plus-outline"
          >{{ $t("Administration.AddUserToTeam") }}
        </card-title>
        <v-card-text>
          <v-form ref="form">
            <v-container>
              <v-row>
                <v-col cols="12">
                  <v-autocomplete
                    v-model="select"
                    :loading="loading"
                    :items="users"
                    :search-input.sync="search"
                    cache-items
                    :label="$t('Administration.Email')"
                    :rules="[rules.required]"
                    hide-no-data
                    hide-details
                    outlined
                    item-value="id"
                    item-text="email"
                  >
                    <template v-slot:selection="{ item, attrs, on }">
                      <v-list-item v-bind="attrs" v-on="on">
                        <v-list-item-avatar>
                          <v-avatar color="primary" class="ma-1">
                            <img :src="item.avatar" />
                          </v-avatar>
                        </v-list-item-avatar>
                        <v-list-item-content>
                          <v-list-item-title>{{
                            item.username
                          }}</v-list-item-title>
                          <v-list-item-subtitle>{{
                            item.email
                          }}</v-list-item-subtitle>
                        </v-list-item-content>
                      </v-list-item>
                    </template>
                    <template v-slot:item="{ item }">
                      <v-list-item-avatar>
                        <v-avatar color="primary" class="ma-1">
                          <img :src="item.avatar" />
                        </v-avatar>
                      </v-list-item-avatar>
                      <v-list-item-content>
                        <v-list-item-title>{{
                          item.username
                        }}</v-list-item-title>
                        <v-list-item-subtitle>{{
                          item.email
                        }}</v-list-item-subtitle>
                      </v-list-item-content>
                    </template>
                  </v-autocomplete>
                </v-col>
              </v-row>
              <v-row>
                <v-col cols="12">
                  <v-select
                    :label="$t('Administration.Role')"
                    :items="rolesList"
                    v-model="selectedRole"
                    :rules="[rules.required]"
                    required
                    chips
                    outlined
                    dense
                    height="50"
                    small-chips
                  >
                    <template v-slot:selection="data">
                      {{ data.item.name }}
                    </template>
                    <template v-slot:item="data">
                      {{ data.item.name }}
                    </template>
                  </v-select>
                </v-col>
              </v-row>
            </v-container>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-btn @click="closeAddDialog" depressed outlined color="primary">
            {{ $t("Common.Cancel") }}
          </v-btn>
          <v-spacer></v-spacer>
          <v-btn
            @click="addConfirm"
            depressed
            color="primary"
            :disabled="!valid"
          >
            {{ $t("Common.Save") }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </span>
</template>


<script lang="ts">
import { Component, mixins, Prop, Watch } from 'nuxt-property-decorator'
import BaseMixin from '@/components/mixins/base'
import TeamsMixin from '@/components/mixins/teams'
import { IRole } from '~/store/teams/server/types'
@Component
export default class AddUser extends mixins(BaseMixin, TeamsMixin) {
  @Prop({ type: Array, default: () => [] }) userList!: Array<String>
  @Prop({ type: Array, default: () => [] }) rolesList!: Array<String>
  select = ''
  selectedRole: IRole = { id: '', team_id: '', name: '', scopes: [] }
  isDialog = false
  items: Array<String> = []
  $refs!: {
    form: HTMLFormElement,
  }
  get valid () {
    return (String(this.select).split(' - ')[0] !== '' && this.selectedRole.id !== '')
  }
  async mounted () {
    await this.getUsersFromServer()
  }
  @Watch('userList') userListCh () {
    // Если userList пуст, то заполняем его всеми пользователями
    if (this.userList.length === 0) {
      this.items = Object.values(this.$store.getters['auth/server/localAuthorization/getList'] ?? [])
        .map((itm: any) => this.objectToString(itm))
    } else {
      this.items = this.userList
    }
  }
  addItem () {
    this.isDialog = true
    this.selectedRole.name = ''
  }
  closeAddDialog () {
    this.isDialog = false
  }
  addConfirm () {
    this.isDialog = false
    if (this.$refs.form.validate()) {
      const req = {
        user_id: this.select,
        role_id: this.selectedRole.id,
        team_id: '',
        edit: "false"
      }
      console.log('before emit', req)
      this.$emit('saveUser', req)
    }
  }
  get rules () {
    return {
      required: (value: string) => !!value || this.$t('Common.Required'),
    }
  }
  @Watch('isDialog')
  AddDialog (val: boolean) {
    val || this.closeAddDialog()
  }
  async querySelections () {
    await this.getUsersFromServer()
  }
  async getUsersFromServer () {
    let params = {
      start: 0,
      limit: 3,
      sort_by: 'username',
      sort_desc: false,
      // search: this.search,
    }
    const res = await this.$axios.get('https://' + process.env.VUE_APP_AUTH_HOSTNAME + '/access/users/list', { params: params })
    this.$store.dispatch('auth/server/localAuthorization/getUsers', res.data.result.items)
  }
  selectChange (val: string) {
    this.select = val
  }
}
</script>

<style>
</style>

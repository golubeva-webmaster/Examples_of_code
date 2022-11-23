<template>
  <v-dialog
    :value="isDialog"
    @input="$emit('change', $event)"
    max-width="500px"
  >
    <v-card class="pa-3">
      <card-title
        class="text-h6"
        :icon="
          action === 'Add'
            ? 'mdi-account-plus-outline'
            : 'mdi-account-edit-outline'
        "
        closeable
        @close="closeAddDialog"
        >{{ $t(`Administration.${action}Role`) }}
      </card-title>
      <v-card-text>
        <v-form v-model="valid">
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-text-field
                  v-model="elementSync.name"
                  :label="$t('Administration.Name')"
                  :rules="[rules.required]"
                  outlined
                ></v-text-field>
                <v-autocomplete
                  :label="$t('Administration.Scopes')"
                  :items="scopes"
                  v-model="elementSync.scopes"
                  :rules="[rules.notEmpty]"
                  multiple
                  chips
                  outlined
                  dense
                  height="50"
                  small-chips
                ></v-autocomplete>
              </v-col>
            </v-row>
          </v-container>
        </v-form>
      </v-card-text>
      <v-card-actions>
        <v-btn @click="addConfirm" depressed color="primary" :disabled="!valid">
          {{ $t("Common.Save") }}
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script lang="ts">
import { Component, mixins, Model, Prop, PropSync, Watch } from 'nuxt-property-decorator'
import BaseMixin from '@/components/mixins/base'
import TeamsMixin from '@/components/mixins/teams'
import AuthMixin from '@/components/mixins/auth'
import { IRole } from '~/store/teams/server/types'


@Component
export default class TeamRolesAddRole extends mixins(BaseMixin, TeamsMixin, AuthMixin) {
  @Model('change', { type: Boolean, default: false }) isDialog!: boolean
  @PropSync('element', { type: Object, default: () => { return { name: '', scopes: [], team_id: '' } } }) elementSync!: IRole
  @Prop({ type: Array, default: () => [] }) scopes!: string[]
  @Prop({ type: String, default: 'add' }) action!: String

  valid = false

  closeAddDialog () {
    this.$emit('change', false)
  }

  addConfirm () {
    this.closeAddDialog()
    // this.$emit("update:element", this.elementSync)
    this.$emit('save', this.elementSync)
  }

  get rules () {
    return {
      required: (value: string) => !!value || this.$t('Common.Required'),
      notEmpty: (value: string) => !!value.length || this.$t('Common.NotEmptyScopes'),
    }
  }

}

</script>

<style>
</style>

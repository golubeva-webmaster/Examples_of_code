
В компоненте TeamsAdminRolesManagement.vue дважны используется компонент TeamRolesAddRole.vue
- Для создания нового элемента
- Для редактирования

Дочерний компонент располагается в родительскм однажды

      <team-roles-add-role
        v-model="roleEditDialog"
        @save="save"
        :scopes="scopes"
        :element.sync="newRole"
      />
      
А вызывается дочерний так:
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
      
      
      <v-btn
      icon
      color="primary"
      :title="$t('Common.Update')"
      @click="editRole(item)"
      >
      <v-icon small> mdi-pencil </v-icon>
    </v-btn>
    
Открытие-закрытие диалогового окна дочернего компонента осуществляется внутри родителя:
    this.roleEditDialog = true

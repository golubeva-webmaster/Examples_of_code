
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
      

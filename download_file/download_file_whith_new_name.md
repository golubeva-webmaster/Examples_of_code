Скачивать файл по клику, предварительно его переименовывая
```
<v-list dense>
    <v-list-item-group color="primary">
      <v-list-item v-for="(file, i) in info.files" :key="i">
        <v-list-item-content>
          <v-list-item-title
            :href="file.path"
            :name="file.name"
            @click="downloadFile"
            v-text="file.name"
          ></v-list-item-title>
        </v-list-item-content>
      </v-list-item>
    </v-list-item-group>
  </v-list>
  ```
  
  ```
async downloadFile (event: any) {
    if (true) {
      event.preventDefault()
      let href = ""
      if ('href' in event.target.attributes) href = event.target.attributes.href.value;
      if ('href' in event.target.parentElement.attributes) href = event.target.parentElement.attributes.href.value;
      let result = await this.$axios.$get(href, {
        responseType: 'blob'
      })
      const url = window.URL
        .createObjectURL(new Blob([result]));
      const link = document.createElement('a');
      link.href = url;
      let filename = href.split('/').pop() || ''
      filename = event.target.attributes.name.value
      link.setAttribute('download', filename);
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    } else {
      event.preventDefault()
      let href = ""
      if ('href' in event.target.attributes) href = event.target.attributes.href.value;
      if ('href' in event.target.parentElement.attributes) href = event.target.parentElement.attributes.href.value;
      window.open(href)
    }
  }
  
  ```

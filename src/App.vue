<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark>
    </v-app-bar>
    <v-content style="padding:50px; margin-top:30px;">
      <!-- Upload file here -->
      <h1>
        Upload file to Cloudinary
      </h1>
      <v-file-input multiple label="Add your files" chips @change="onAddFiles" />

      <v-card v-if="files.length > 0">
        <v-card-text>
          <v-alert type="success" v-for="file in files" :key="file.public_id">
            File uploaded: {{file.original_filename}} at {{file.url}}
          </v-alert>
        </v-card-text>
      </v-card>

      <v-alert v-if="isError">
        {{errorText}}
      </v-alert>

    </v-content>
  </v-app>
</template>

<script>
export default {
  name: 'App',

  data: () => ({
    files: [],
    isError: false,
    errorText: null
  }),
  methods: {
    onAddFiles(files) {
      if (files.length > 0) {
        files.forEach((file) => {
          window.console.log(file);
          this.uploadFileToCloudinary(file).then((fileResponse) => {
            this.files.push(fileResponse);
          });
        });
      }
    },
    uploadFileToCloudinary(file) {
      return new Promise(function (resolve, reject) {
        //Ideally these to lines would be in a .env file
        const CLOUDINARY_URL = 'https://api.cloudinary.com/v1_1/YOUR_CLOUD_NAME/upload';
        const CLOUDINARY_UPLOAD_PRESET = 'YOUR_UPLOAD_PRESET';

        let formData = new FormData();
        formData.append('upload_preset', CLOUDINARY_UPLOAD_PRESET);
        formData.append('folder', 'cloudinary-demo');
        formData.append('file', file);

        let request = new XMLHttpRequest();
        request.open('POST', CLOUDINARY_URL, true);
        request.setRequestHeader('X-Requested-With', 'XMLHttpRequest');

        request.onreadystatechange = () => {
          // File uploaded successfully
          if (request.readyState === 4 && request.status === 200) {
            let response = JSON.parse(request.responseText);
            resolve(response);
          }

          // Not successfull, let find our what happened
          if (request.status !== 200) {
            let response = JSON.parse(request.responseText);
            let error = response.error.message;
            this.errorText = 'error uploading files ' + error;
            this.isError = true;
            reject(error);
          }

        };

        request.onerror = (err) => {
          alert('error: ' + err);
          reject(err);
        };

        request.send(formData);
      });
    }
  }
};
</script>

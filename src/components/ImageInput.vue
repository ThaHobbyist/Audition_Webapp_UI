<template>
  <div>
    <div @click="launchFilePicker()">
      <slot name="activator"></slot>
    </div>
    <input
      type="file"
      ref="file"
      :name="uploadFieldName"
      @change="onFileChange($event.target.name, $event.target.files)"
      style="display: none"
    />
    <v-dialog v-model="errorDialog" max-width="300">
      <v-card>
        <v-card-text class="subheading">{{ errorText }}</v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn @click="errorDialog = false" flat>Got it!</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import common from "@/services/common.js";
export default {
  name: "image-input",
  data: () => ({
    errorDialog: null,
    errorText: "",
    uploadFieldName: "file",
    maxSize: 1024,
    ImgLink: "",
  }),
  props: [{ value: Object }],
  methods: {
    launchFilePicker() {
      this.$refs.file.click();
    },
    async onFileChange(fieldName, file) {
      const { maxSize } = this;
      let imageFile = file[0];
      if (file.length > 0) {
        let size = imageFile.size / maxSize / maxSize;
        if (!imageFile.type.match("image.*")) {
          // check whether the upload is an image
          this.errorDialog = true;
          this.errorText = "Please choose an image file";
        } else if (size > 1) {
          // check whether the size is greater than the size limit
          this.errorDialog = true;
          this.errorText =
            "Your file is too big! Please select an image under 1MB";
        } else {
          // Append file into FormData and turn file into image URL
          let formData = new FormData();
          let imageURL = URL.createObjectURL(imageFile);
          formData.append(fieldName, imageFile);
          // Emit the FormData and image URL to the parent component
          this.$emit("input", { formData, imageURL });
          await common.upload(formData).then((res) => {
            // console.log(res.data);
            this.ImgLink = res.data.link;
            // console.log(this.ImgLink);
            this.$emit("getLink", {
              link: this.ImgLink,
            });
          });
        }
      }
    },
  },
};
</script>

<template>
  <div>
    <v-toolbar flat>
      <v-toolbar-title>
        <v-card-actions>
          <h5 class="title_cat">Update Moment</h5>
        </v-card-actions>
      </v-toolbar-title>
    </v-toolbar>
    <v-row>
      <v-col cols="12" md="4" sm="4" offset-md="1">
        <v-carousel height="350px">
          <v-carousel-item v-for="(item, i) in moment.image" :key="i">
            <v-img class="rounded-lg" :src="url + item.img"> </v-img
          ></v-carousel-item>
        </v-carousel>
      </v-col>
      <v-col cols="12" sm="6" md="6">
        <v-row class="pa-12">
          <v-container>
            <v-col cols="12">
              <v-form ref="form" class="login">
                <div v-if="errMessage">
                  <v-alert border="top" color="red lighten-2" dark>
                    <div
                      v-for="(item, index) in errMessage.errors"
                      :key="index"
                    >
                      <p>{{ item }}</p>
                    </div>
                  </v-alert>
                </div>
                <h5>Moment Title</h5>
                <v-text-field
                  v-model="moment.title"
                  :rules="nameRules"
                  placeholder="Input Title"
                  required
                  outlined
                  value="rounded"
                  dense
                  color="#489FB5"
                ></v-text-field>
                <div>
                  <h5>Description</h5>
                  <v-textarea
                    v-model="moment.description"
                    :rules="descRules"
                    placeholder="Input Description"
                    outlined
                    rounded
                    dense
                    color="#489FB5"
                  ></v-textarea>
                </div>

                <h5>Location</h5>
                <v-autocomplete
                  clearable
                  rounded
                  :rules="locRules"
                  item-text="name"
                  :items="location_data"
                  item-value="name"
                  label="Select Location"
                  v-model="moment.location"
                  placeholder="Select Location"
                  prepend-inner-icon="mdi-magnify"
                  solo
                >
                </v-autocomplete>

                <v-row>
                  <v-col>
                    <div>
                      <h5>Pet</h5>

                      <v-radio-group
                        v-model="moment.animal_name"
                        row
                        @change="subSpecies(moment.animal_name)"
                      >
                        <v-radio
                          color="#489FB5"
                          label="Cat"
                          on-icon="mdi-cat"
                          value="Cat"
                        >
                        </v-radio>
                        <v-radio
                          color="#489FB5"
                          label="Dog"
                          on-icon="mdi-dog"
                          value="Dog"
                        ></v-radio>
                      </v-radio-group>
                    </div>
                  </v-col>

                  <v-col>
                    <div>
                      <h5>Gender</h5>
                      <v-radio-group v-model="moment.gender" row>
                        <v-radio
                          label="Male"
                          on-icon="mdi-gender-male"
                          value="Male"
                        ></v-radio>
                        <v-radio
                          label="Female"
                          value="Female"
                          on-icon="mdi-gender-female"
                        ></v-radio>
                      </v-radio-group>
                    </div>
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="12" sm="12" md="12">
                    <h5>Type</h5>
                    <v-autocomplete
                      clearable
                      rounded
                      item-text="name"
                      :items="animal_list"
                      item-value="slug"
                      :rules="typeRules"
                      label="Search Pet Type"
                      v-model="moment.animal_type"
                      placeholder="Select Pet Type"
                      prepend-inner-icon="mdi-magnify"
                      solo
                    >
                      <template slot="selection" slot-scope="{ item }">
                        {{ item.type }} - {{ item.name }}
                      </template>
                      <template slot="item" slot-scope="{ item }">
                        {{ item.type }} - {{ item.name }}
                      </template>
                    </v-autocomplete>
                  </v-col>
                </v-row>
                <v-menu
                  v-model="menu2"
                  :close-on-content-click="false"
                  :nudge-right="40"
                  transition="scale-transition"
                  offset-y
                  min-width="auto"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="moment.date"
                      label="Date"
                      prepend-icon="mdi-calendar"
                      readonly
                      :rules="dateRules"
                      v-bind="attrs"
                      v-on="on"
                    ></v-text-field>
                  </template>
                  <v-date-picker
                    v-model="moment.date"
                    @input="menu2 = false"
                  ></v-date-picker>
                </v-menu>

                <v-row>
                  <v-col cols="4" offset-md="1">
                    <v-btn
                      :to="{
                        name: 'profile',
                        params: { username: moment.user.username },
                      }"
                      rounded
                      class="btn_login"
                      outlined
                      color="#489FB5"
                      block
                      elevation="7"
                    >
                      Back To Profile
                    </v-btn>
                  </v-col>
                  <v-col cols="6">
                    <v-btn
                      rounded
                      class="btn_login"
                      color="#489FB5"
                      dark
                      elevation="7"
                      block
                      @click="upload"
                    >
                      Submit Change
                    </v-btn>
                  </v-col>
                </v-row>
              </v-form>
            </v-col>
          </v-container>
        </v-row>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import { required, minLength } from "vuelidate/lib/validators";

import { mapGetters } from "vuex";
import axios from "axios";
export default {
  nama: "UploadAdoption",
  created() {
    this.$emit("update:layout", "div");
  },
  data(){
    return {
    url: process.env.VUE_APP_IMAGE_URL,
    moment: [],
    menu2: false,
    showform: false,
    isDragging: false,
    dragCount: 0,
    animal_list: [],
    files: [],
    images: [],
    errMessage: "",
    animal_name: "",
    type: undefined,
    gender: "",
    name: "",
    title: "",
    location_data: [],
    body: "",
    health: "",
    nameRules: [(value) => !!value || "Name is required"],
    desc: "",
    
    typeRules: [(value) => !!value || "Animal Type must be selected"],
    dateRules: [(value) => !!value || "Date must be selected"],
    descRules: [(value) => !!value || "Description is required"],
    loc: "",
    locRules: [(value) => !!value || "Location is required"],
    submitStatus: null,
    age: "",
    ageRules: [(value) => !!value || "Age is required"],
    color: "",
    colorRules: [(value) => !!value || "Type is required"],
  }
  },
  validations: {
    name: {
      required,
    },
    images: {
      required,
    },
    desc: {
      required,
    },
  },
  computed: {
    ...mapGetters({
      isLoggedIn: "isLoggedIn",
      user: "user",
    }),
  },
  mounted() {
    let uri_moment =
      process.env.VUE_APP_ROOT_API + "auth/moment/" + this.$route.params.id;
    this.$http
      .get(uri_moment)
      .then((response) => {
        this.moment = response.data;
        this.type = response.data.animal_type;
      })
      .catch((error) => {
        if (error.response.status === 404 || error.response.status === 422)
          this.$router.push("/error");
      });
    let uri_animal = process.env.VUE_APP_ROOT_API + "animal";
    this.$http.get(uri_animal).then((response) => {
      this.animal_list = response.data;
    });
    let uri_loc = process.env.VUE_APP_ROOT_API + "location/cities";
    this.$http.get(uri_loc).then((response) => {
      this.location_data = response.data;
    });
  },
  methods: {
    deleteImage() {
      this.images.splice(0, this.images.length);
    },
    subSpecies(value) {
      if (value != null) {
        let uri_cat = process.env.VUE_APP_ROOT_API + "animal/" + value;
        this.$http.get(uri_cat).then((response) => {
          this.animal_list = response.data;
        });
      } else {
        let uri_cat = process.env.VUE_APP_ROOT_API + "animal/";
        this.$http.get(uri_cat).then((response) => {
          this.animal_list = response.data;
        });
      }
    },
    submit() {
      this.$v.$touch();
      if (this.$v.$invalid) {
        this.submitStatus = "Error";
      } else {
        console.log("sss"), (this.submitStatus = "Pending");
        setTimeout(() => {
          this.submitStatus = "OK";
        }, 500);
      }
    },
    OnDragEnter(e) {
      e.preventDefault();

      this.dragCount++;
      this.isDragging = true;
      return false;
    },
    OnDragLeave(e) {
      e.preventDefault();
      this.dragCount--;
      if (this.dragCount <= 0) this.isDragging = false;
    },
    onInputChange(e) {
      const files = e.target.files;
      for (let i = 0; i < 4; i++) {
        this.addImage(files[i]);
      }
    },
    onDrop(e) {
      e.preventDefault();
      e.stopPropagation();
      this.isDragging = false;
      const files = e.dataTransfer.files;
      for (let i = 0; i < 4; i++) {
        this.addImage(files[i]);
      }
    },
    addImage(file) {
      if (!file.type.match("image.*")) {
        this.$toastr.e(`${file.name} is not an image`);
        return;
      }
      this.files.push(file);
      const img = new Image(),
        reader = new FileReader();
      reader.onload = (e) => this.images.push(e.target.result);
      reader.readAsDataURL(file);
    },
    getFileSize(size) {
      const fSExt = ["Bytes", "KB", "MB", "GB"];
      let i = 0;

      while (size > 900) {
        size /= 1024;
        i++;
      }
      return `${Math.round(size * 100) / 100} ${fSExt[i]}`;
    },
    upload() {
      const formData = new FormData();
      const config = {
        headers: {
          "content-type": "multipart/form-data",
        },
      };

      formData.append("user_id", this.user.id);
      formData.append("animal_name", this.moment.animal_name);
      formData.append("title", this.moment.title);
      formData.append("desc", this.moment.description);
      formData.append("type", this.moment.animal_type);
      formData.append("gender", this.moment.gender);
      formData.append("loc", this.moment.location);
      formData.append("date", this.moment.date);
      /*$.each(this.images, function (key, image) {
        formData.append(`images[${key}]`, image)
        })*/
      this.files.forEach((file) => {
        formData.append("images[]", file, file.name);
      });
      axios
        .post(
          process.env.VUE_APP_ROOT_API +
            "profile/" +
            this.user.id +
            "/moment/" +
            this.$route.params.id,
          formData,
          config
        )
        .then((response) => {
          this.$swal({
            icon: "success",
            confirmButtonColor: "#3085d6",
            text: "Moment has been updated successfully",
            confirmButtonText: "Confirm",
            closeOnCancel: true,
          });
          this.$router.push({name : 'profile', params : {username : this.user.username}});
        })
        .catch((error) => {
          this.errMessage = error.response.data;
        });
    },
  },
};
</script>

<style lang="scss" scoped>
/* Upload Adoption */
.uploader {
  width: 100%;
  background: #2196f3;
  color: #fff;
  padding: 40px 15px;
  text-align: center;
  border-radius: 10px;
  border: 3px dashed #fff;
  font-size: 20px;
  position: relative;
  &.dragging {
    background: #fff;
    color: #2196f3;
    border: 3px dashed #2196f3;
    .file-input label {
      background: #2196f3;
      color: #fff;
    }
  }
  i {
    font-size: 85px;
  }
  .file-input {
    width: 200px;
    margin: auto;
    height: 68px;
    position: relative;
    label,
    input {
      background: #fff;
      color: #2196f3;
      width: 100%;
      position: absolute;
      left: 0;
      top: 0;
      padding: 10px;
      border-radius: 4px;
      margin-top: 7px;
      cursor: pointer;
    }
    input {
      opacity: 0;
      z-index: -2;
    }
  }
  .images-preview {
    display: flex;
    flex-wrap: wrap;
    margin-top: 20px;
    .img-wrapper {
      width: 160px;
      display: flex;
      flex-direction: column;
      margin: 10px;
      height: 150px;
      justify-content: space-between;
      background: #fff;
      box-shadow: 5px 5px 20px #3e3737;
      img {
        max-height: 105px;
      }
    }
    .details {
      font-size: 12px;
      background: #fff;
      color: #000;
      display: flex;
      flex-direction: column;
      align-items: self-start;
      padding: 3px 6px;
      .name {
        overflow: hidden;
        height: 18px;
      }
    }
  }
  .upload-control {
    position: absolute;
    width: 100%;
    background: #fff;
    top: 0;
    left: 0;
    border-top-left-radius: 7px;
    border-top-right-radius: 7px;
    padding: 10px;
    padding-bottom: 4px;
    text-align: right;
    button,
    label {
      background: #2196f3;
      border: 2px solid #03a9f4;
      border-radius: 3px;
      color: #fff;
      font-size: 15px;
      cursor: pointer;
    }
    label {
      padding: 2px 5px;
      margin-right: 10px;
    }
  }
}
</style>
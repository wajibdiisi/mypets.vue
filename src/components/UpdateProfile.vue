<template>
  <div>
    <v-toolbar flat>
      <v-toolbar-title>
        <v-card-actions>
          <h5 class="title_cat">Edit Profile</h5>
        </v-card-actions>
      </v-toolbar-title>
    </v-toolbar>
    <v-row>
      <v-col cols="12" md="4" sm="4" offset-md="1">
        <div
          class="uploader"
          @dragenter="OnDragEnter"
          @dragleave="OnDragLeave"
          @dragover.prevent
          @drop="onDrop"
          :class="{ dragging: isDragging }"
        >
          <div class="upload-control" v-show="images.length">
            <label for="file">Select a file</label>
            <button @click="deleteImage">Delete Image</button>
          </div>

          <div v-show="!images.length">
            <i class="fa fa-cloud-upload"></i>
            <p>Current Photo</p>
            <div v-if="user.picture != null">
              <v-img
                class="img_update_profile"
                max-height="400"
                max-width="400"
                :src="url + user.picture"
              />
            </div>
            <div class="file-input">
              <label for="file">Select a file</label>
              <input type="file" id="file" @change="onInputChange" multiple />
            </div>
          </div>
          <div class="images-preview" v-show="images.length">
            <div
              class="img-wrapper justify-center"
              v-for="(image, index) in images"
              :key="index"
            >
              <img :src="image" :alt="`Image Uplaoder ${index}`" />
              <div class="details">
                <span class="name" v-text="files[index].name"></span>
                <span
                  class="size"
                  v-text="getFileSize(files[index].size)"
                ></span>
              </div>
            </div>
          </div>
        </div>
      </v-col>

      <v-col cols="12" sm="6" md="6">
        <v-row class="pa-12">
          <v-container>
            <v-col cols="12">
              <v-form ref="form" class="login">
                <div  v-if="errMessage">
            <v-alert
      border="top"
      color="red lighten-2"
      dark
    >
    <div v-for="(item,index) in errMessage.errors" :key="index">
      <p>{{item}}</p>
      </div> 
    </v-alert>
          </div> 
                <h5 class="name_edit_profile">Name</h5>
                <v-text-field
                  v-model="user.full_name"
                  :rules="nameRules"
                  placeholder="Input Name"
                  required
                  outlined
                  rounded
                  dense
                  color="#489FB5"
                ></v-text-field>
                <div>
                  <h5>Phone</h5>
                  <v-text-field
                    v-model="user.phone"
                    :rules="phoneRules"
                    placeholder="Input Phone Number"
                    type="number"
                    required
                    outlined
                    rounded
                    dense
                    color="#489FB5"
                  ></v-text-field>
                </div>

                <div>
                  <h5>Email</h5>
                  <v-text-field
                    v-model="user.email"
                    :rules="emailRules"
                    placeholder="Input Email"
                    required
                    outlined
                    rounded
                    dense
                    color="#489FB5"
                  ></v-text-field>
                </div>

                <div>
                  <h5>Location</h5>
                  <v-autocomplete
                    clearable
                    rounded
                    :rules="locRules"
                    item-text="name"
                    :items="location_data"
                    item-value="name"
                    label="Select Location"
                    v-model="user.location"
                    placeholder="Select Location"
                    prepend-inner-icon="mdi-magnify"
                    solo
                  >
                  </v-autocomplete>
                </div>
                <v-checkbox
                  v-model="user.whatsapp"
                  label="Does your phone number attached to Whatsapp Application?"
                ></v-checkbox>
                <v-row>
                  <v-col cols="12" lg="6">
                    <v-btn
                      rounded
                      dark
                      class="btn_login"
                      outlined
                      color="#489FB5"
                      block
                      :to="{
                        name: 'profile',
                        params: { username: user.username },
                      }"
                      elevation="7"
                    >
                      Back To Profile
                    </v-btn>
                  </v-col>
                  <v-col cols="12" lg="6">
                    <v-btn
                      rounded
                      class="btn_login"
                      color="#489FB5"
                      dark
                      elevation="7"
                      block
                      @click="upload"
                    >
                      Update Profile
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
import axios from "axios";
import { mapGetters } from "vuex";
export default {
  nama: "UploadMoment",
  created() {
    this.$emit("update:layout", "div");
  },
  mounted() {

    let uri_loc = process.env.VUE_APP_ROOT_API + "location/cities";
    this.$http.get(uri_loc).then((response) => {
      this.location_data = response.data;
    });
    let uri =
      process.env.VUE_APP_ROOT_API +
      "profile/" +
      this.$route.params.id +
      "/information";
    this.$http
      .get(uri)
      .then((response) => {
        this.user = response.data;
        this.username = response.data.username;
      })
      .catch(function (error) {
        
        this.$router.push("/");
      });
  },
  data() {
    return {
      username: "",
      errMessage: "",
      url: process.env.VUE_APP_IMAGE_URL,
      isDragging: false,
      dragCount: 0,
      files: [],
      user : [],
      images: [],
      location_data: [],

      name: "",
      nameRules: [
        (value) => !!value || "Name required",
        (value) =>
          (value && value.length >= 5) || "Username must have 5+ characters",
      ],
      username: "",
      usernameRules: [
        (value) => !!value || "Username required",
        (value) =>
          (value && value.length >= 5) || "Username must have 5+ characters",
        (value) => /([@])/.test(value) || "Must have one special character [@]",
      ],
      phone: "",
      phoneRules: [
        (value) => !!value || "Must Number & Required",
        (value) =>
          (value && value.length >= 7) || "Phone must have 7+ characters",
      ],
      email: "",
      emailRules: [
        (value) => !!value || "E-mail required",
        (value) => /.+@.+/.test(value) || "E-mail must be valid",
      ],
      loc: "",
      locRules: [(value) => !!value || "Location required"],
    };
  },
  methods: {
    deleteImage() {
      this.images.pop();
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

      this.addImage(files[0]);
    },
    onDrop(e) {
      e.preventDefault();
      e.stopPropagation();
      this.isDragging = false;
      const files = e.dataTransfer.files;

      this.addImage(files[0]);
    },
    addImage(file) {
      if (!file.type.match("image.*")) {
        this.$toastr.e(`${file.name} is not an image`);
        return;
      }
      this.files.push(file);
      const img = new Image(),
        reader = new FileReader();
      this.images.pop();
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
      formData.append("full_name", this.user.full_name);
      formData.append("phone", this.user.phone);
      formData.append("email", this.user.email),
        formData.append("location", this.user.location);
      formData.append("whatsapp", this.user.whatsapp);
      /*$.each(this.images, function (key, image) {
        formData.append(`images[${key}]`, image)
        })*/
      this.files.forEach((file) => {
        formData.append("images[]", file, file.name);
      });
      axios
        .post(
          process.env.VUE_APP_ROOT_API + "profile/" + this.user.id + "/update",
          formData,
          config
        )
        .then((response) => {
          this.$swal({
            icon: "success",
            confirmButtonColor: "#3085d6",
            text: "Profile has been successfully updated",
            confirmButtonText: "Confirm",
          });
          console.log(response.data)
          this.$router.push({name : 'profile', params : {username : this.user.username}});
        })
        .catch((err) => {
          this.errMessage = err.response;
        });
    },
  },
  computed: {
    ...mapGetters({
      isLoggedIn: "isLoggedIn",
      user: "user",
    }),
  },
};
</script>


<style lang="scss" scoped>
/* Upload Adoption */
.img_update_profile {
  clip-path: inset(20px 20px 50px round 15px 0);
}
.name_edit_profile {
  margin-top: 30px;
}
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
    clip-path: inset(20px 20px 50px round 15px 0);
    .img-wrapper {
      width: 400px;
      display: flex;
      flex-direction: column;
      margin: 30px;
      height: 400px;
      background: #fff;
      box-shadow: 5px 5px 20px #3e3737;
      img {
        max-height: 400px;
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
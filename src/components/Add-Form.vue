<template>
  <v-card class="overview ml-10" max-width="100%" outlined color="#111315">
    <Header />
    <v-container fluid>
      <v-row align="center">
        <v-col class="d-flex flex-column mx-10" cols="12" sm="12">
          ROUND TIME:
          <input type="number" style="width: 20%" v-model="Rtime" />
          QUESTION:
          <input
            outlined
            color="#BEFFC1"
            name="input-7-4"
            v-model="Ques"
            style="width: 80%"
          />
          <div class="media">
            UPLOAD FILES:
            <div class="uploadBox">
              <UploadImage @getImageLink="updateImageLink" :avatarImg="avatarImg" />
              <UploadAudio @getAudioLink="updateAudioLink" :avatarAudio="avatarAudio"/>
            </div>
          </div>
          QUSETION TYPE:
          <v-select
            :items="Qtype"
            v-model="Questype"
            dense
            outlined
            value=""
            color="#7B849F"
            class="mb-10 dropdown"
            style="width: 30%; border: 0.2px solid #7b849f; height: 40px"
          ></v-select>
          <div
            class="mcq-options mx-5 pa-5"
            v-if="
              Questype === 'SINGLE CHOICE' || Questype === 'MULTIPLE CHOICE'
            "
          >
            <v-container>
              <v-row>
                <v-col cols="12" sm="6">
                  <v-text-field
                    v-model="choice1"
                    label="Choice 1"
                    solo
                  ></v-text-field>
                </v-col>

                <v-col cols="12" sm="6">
                  <v-text-field
                    v-model="choice2"
                    solo
                    label="Choice 2"
                  ></v-text-field>
                </v-col>

                <v-col cols="12" sm="6">
                  <v-text-field
                    v-model="choice3"
                    solo
                    label="Choice 3"
                  ></v-text-field>
                </v-col>

                <v-col cols="12" sm="6">
                  <v-text-field
                    v-model="choice4"
                    label="Choice 4"
                    solo
                  ></v-text-field>
                </v-col>
              </v-row>
              <div class="d-flex float-right mr-12 pa-2">
                <v-btn
                  class="ma-2 black--text"
                  color="#BEFFC1"
                  @click="saveOptions"
                  v-if="showBtn"
                >
                  Save
                </v-btn>
              </div>
            </v-container>
          </div>

          <v-btn
            class="mx-2"
            fab
            dark
            color="#4288CA"
            :disabled="Ques === '' || Questype === ''"
            @click="addQues"
          >
            <v-icon dark> mdi-plus </v-icon>
          </v-btn>
        </v-col>
      </v-row>
    </v-container>
    <v-snackbar v-model="snackbar"
      >THE ROUND HAS BEEN SAVED SUCCESSFULLY
      <template v-slot:action="{ attrs }">
        <v-btn color="pink" text v-bind="attrs" @click="snackbar = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>
    <div class="d-flex float-right mr-12 pa-2">
      <v-btn
        class="ma-2"
        :loading="loading1"
        :disabled="loading1"
        color="info"
        @click="saveRound"
      >
        Save Round
        <template v-slot:loader>
          <span class="custom-loader">
            <v-icon light>mdi-cached</v-icon>
          </span>
        </template>
      </v-btn>
    </div>
  </v-card>
</template>

<script>
import UploadImage from "./UploadImg.vue";
import UploadAudio from "../components/UploadAudio.vue";
import Header from "../components/Header.vue";
import common from "@/services/common.js";

export default {
  components: {
    UploadImage,
    UploadAudio,
    Header,
  },
  props: ["Questions"],
  data: () => ({
    Rtime: 10,
    Ques: "",
    Questype: "",
    ImgLink: "",
    AudioLink: "",
    score: 0,
    avatarImg: null,
    avatarAudio: null,
    saving: false,
    saved: false,
    showBtn: true,
    showMediaBtn: false,
    choice1: "",
    choice2: "",
    choice3: "",
    choice4: "",
    options: [],
    loader: null,
    loading: false,
    loading1: false,
    snackbar: false,
    currentround: "",
    Qtype: ["SINGLE CHOICE", "MULTIPLE CHOICE", "ATTACH FILE", "TEXTAREA"],
  }),
  methods: {
    saveOptions() {
      this.options = [this.choice1,this.choice2,this.choice3,this.choice4]
      console.log(this.options);
      (this.choice1 = ""),
        (this.choice2 = ""),
        (this.choice3 = ""),
        (this.choice4 = "");
      this.showBtn = false;

    },
    addQues() {
      if (this.Ques !== "") {
        this.Questions.push({
          quesText: this.Ques,
          score: this.score,
          quesType: this.Questype,
          options: this.options,
          ImageLink : this.ImgLink,
          AudioLink : this.AudioLink,
        });
        console.log("==");
        console.log(this.Questions);
        this.Ques = [];
        this.options = [];
        this.Questype = "";
        this.showBtn = true;
        this.avatarImg = null;
        this.avatarAudio = null;
        this.ImgLink = "";
        this.AudioLink = "";
      }
    },
    saveRound() {
      var round = { time: this.Rtime, questions: this.Questions };
      console.log(round);
      common.addround(round).then((res) => {
        console.log("==========");
        console.log(res.data);
        console.log("==========");
        localStorage.removeItem("Questions");
        this.Questions.splice(0, this.Questions.length);
        this.snackbar = true;
      });
    },
    updateImageLink(link){
      console.log(link.link);
      this.ImgLink = link.link;
    },
    updateAudioLink(link){
      console.log(link.link);
      this.AudioLink = link.link;
    }
  },
  watch: {
    Questions: {
      handler() {
        console.log(this.Questions);
        localStorage.setItem("Questions", JSON.stringify(this.Questions));
      },
      deep: true,
    },
    avatar: {
      handler: function () {
        this.saved = false;
      },
      deep: true,
    },
  },

};
</script>

<style scoped>
.custom-loader {
  animation: loader 1s infinite;
  display: flex;
}
@-moz-keyframes loader {
  from {
    transform: rotate(0);
  }
  to {
    transform: rotate(360deg);
  }
}
@-webkit-keyframes loader {
  from {
    transform: rotate(0);
  }
  to {
    transform: rotate(360deg);
  }
}
@-o-keyframes loader {
  from {
    transform: rotate(0);
  }
  to {
    transform: rotate(360deg);
  }
}
@keyframes loader {
  from {
    transform: rotate(0);
  }
  to {
    transform: rotate(360deg);
  }
}
input {
  border: 0.2px solid #7b849f;
  margin-bottom: 30px;
  margin-top: 30px;
  border-radius: 2px;
  color: white;
  padding: 5px;
  padding-left: 10px;
}

.mcq-options {
  border: 0.02rem solid rgba(245, 245, 245, 0.521);
  border-radius: 4px;
  width: 60%;
  margin-bottom: 25px;
  padding-bottom: 0px;
}
.media {
  display: flex;
  flex-direction: column;
}
.uploadBox {
  display: flex;
}
.dropdown {
  margin-top: 10px;
}
.upload {
  display: flex;
  justify-content: center;
  height: 200px;
  width: 200px;
  border-radius: 20px;
  background-color: #1a1d1f;
  border: 1px solid #7b849f;
  margin: 10px;
  margin-bottom: 30px;
  margin-top: 30px;
}
.text {
  opacity: 0.4;
}
.inner {
  border: 2px dashed rgb(7, 7, 7);
  height: 170px;
  width: 170px;
  border-radius: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>

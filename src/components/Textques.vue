<template>
    <div>
        <v-container class="d-flex justify-center">
            <div style="width: 95%">
                <div
                    class="d-flex mb-2"
                    style="background-color: rgba(0, 0, 0, 0); width: 95%"
                    v-bind:class="{
                        'flex-column justify-center mx-auto': mobileView,
                        'ml-4': !mobileView,
                    }"
                >
                    <v-card-text
                        class="mb-6 pa-0"
                        id="text"
                        :class="{
                            'justify-center text-center mx-auto': mobileView,
                            'd-flex align-center justify-start text-left':
                                !mobileView,
                        }"
                        >{{ question.quesText }}</v-card-text
                    >
                    <div
                        class="mx-auto d-flex flex-column"
                        :class="{ 'flex-column justify-center': mobileView }"
                        style="min-width: 20%"
                    >
                        <a :href="question.ImageLink" target="_blank">
                            <v-img
                                class="ma-4 img"
                                v-if="question.ImageLink"
                                :src="question.ImageLink"
                                :class="{
                                    'mx-auto': mobileView,
                                    'mt-0': !mobileView,
                                }"
                            ></v-img>
                        </a>
                        <vuetify-audio
                            class="ma-4"
                            v-if="question.AudioLink"
                            :file="question.AudioLink"
                            color="success"
                            :ended="audioFinish"
                            downloadable
                            :class="{ 'mx-auto': mobileView }"
                        ></vuetify-audio>
                    </div>
                </div>
                <div style="background-color: rgba(0, 0, 0, 0); width: 95%">
                    <v-textarea
                        v-model="answer"
                        filled
                        outlined
                        name="input-7-4"
                        label="Answers"
                        auto-grow
                        color="#00FFBF"
                        class="text"
                    ></v-textarea>
                    <v-textarea
                        v-model="studentanswer"
                        filled
                        readonly
                        v-if="admin === true"
                        name="input-7-4"
                        label="Answers"
                        auto-grow
                        color="#00FFBF"
                        class="text"
                    ></v-textarea>
                </div>
            </div>
        </v-container>
        <div
            class="d-flex justify-end mx-auto"
            style="width: 90%"
            :class="{ 'justify-center': !vertical }"
        >
            <v-btn
                class="ma-2 black--text"
                :loading="loading"
                :disabled="loading"
                color="#4288CA"
                @click="saveAnswer"
            >
                <v-icon class="mr-2">mdi-content-save</v-icon>Save
            </v-btn>
        </div>
        <v-snackbar v-model="snackbar">
            {{ text }}
            <template v-slot:action="{ attrs }">
                <v-btn
                    color="blue lighten-3"
                    text
                    v-bind="attrs"
                    @click="snackbar = false"
                    >Close</v-btn
                >
            </template>
        </v-snackbar>
    </div>
</template>

<script>
import common from "../services/common.js";
export default {
    name: "Textques",
    props: ["question", "admin", "studentanswer", "mobileView", "uuid"],
    data: () => ({
        test: [],
        answer: "",
        ansArray: [],
        snackbar: false,
        text: "Your Answer is saved",
        loading: false,
    }),
    components: {
        VuetifyAudio: () => import("vuetify-audio"),
    },

    created() {
        console.log(localStorage.getItem("answers"));
        if (localStorage.getItem("answers") != null) {
            var answers = JSON.parse(localStorage.getItem("answers"));
            answers.forEach((answer) => {
                if (answer.qid === this.question.quesId) {
                    console.log("---------");
                    console.log(typeof answer);
                    this.answer = answer.answer[0];
                }
            });
        }
    },
    methods: {
        saveAnswer() {
            this.loading = true;

            var current_answer = JSON.parse(
                localStorage.getItem("answers")
            ).find((answer) => answer.qid === this.question.quesId);
            
            if (current_answer) {
                common.updateAnswer(current_answer).then(() => {
                    console.log(current_answer);
                    this.snackbar = true;
                    this.loading = false;
                });
            }else {
              this.loading = false;
            }
        },
    },
    watch: {
        answer: {
            handler() {
                this.ansArray[0] = this.answer;
                console.log(this.answer);
                if (
                    localStorage.getItem("answers") === null &&
                    (this.admin === null || this.admin === undefined)
                ) {
                    var newanswer = {
                        answer: this.ansArray,
                        qid: this.question.quesId,
                        qtype: this.question.quesType,
                        roundNo: this.question.roundmodelRoundNo,
                        ansLink: null,
                        userUuid: this.uuid,
                    };
                    var ans = [];
                    ans.push(newanswer);
                    localStorage.setItem("answers", JSON.stringify(ans));
                } else {
                    var answers = JSON.parse(localStorage.getItem("answers"));
                    var foundanswer = false;
                    answers.forEach((answer) => {
                        if (answer.qid === this.question.quesId) {
                            answer.answer = this.ansArray;
                            foundanswer = true;
                        }
                    });
                    if (foundanswer === false) {
                        var newans = {
                            answer: this.ansArray,
                            qid: this.question.quesId,
                            qtype: this.question.quesType,
                            roundNo: this.question.roundmodelRoundNo,
                            ansLink: null,
                            userUuid: this.uuid,
                        };
                        answers.push(newans);
                    }
                    localStorage.setItem("answers", JSON.stringify(answers));
                }
            },
        },
    },
};
</script>

<style>
#text {
    color: white;
    /* text-align: center !important; */
    font-size: 1rem;
    font-weight: 500;
}
.img {
    display: block;
    max-height: 300px;
    max-width: 300px;
    width: auto;
    height: auto;
}

.opt_cont {
    width: 700px;
}

.option_item {
    display: block;
    margin: 10px;
    width: 180px !important;
    height: 50px !important;
}

.option_item .option_inner {
    height: 100%;
    width: 100%;
    background-color: #2f333f;
    border-radius: 10px;
    padding: 5px auto;
    text-align: center;
    cursor: pointer;
    color: #fff;
    display: block;
    border: 3px solid transparent;
}

.option_item .checkbox {
    z-index: 1;
    opacity: 1;
    display: none;
}

.name {
    width: 69%;
}

.option_item .checkbox:checked ~ .option_inner {
    border-color: #515c64;
}

.option_inner .icon {
    opacity: 0;
}

.option_item .checkbox:checked ~ .option_inner {
    border-color: #515c64;
}

.option_item .checkbox:checked ~ .option_inner .tickmark {
    border-radius: 0;
}

.option_item .option_inner .tickmark {
    background-color: #515c64;
    border-radius: 5px;
    width: 31%;
    height: 100%;
}

.tickmark .icon {
    align-self: center;
}

@media screen and (max-width: 960px) {
    .img {
        max-height: 200px;
        max-width: 200px;
    }

    .opt_cont {
        width: 100%;
        margin-left: 0%;
    }

    #text {
        max-width: 90%;
    }
}
</style>

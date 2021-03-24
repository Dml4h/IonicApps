<style scoped>
/* Center and style the content */
.header {
  font-family: "HovdenStitch";
  padding: 2rem;
  display: flex;
  text-align: center;
}
.Home {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  background-image: url(/img/BackgroundImg.b4af3dae.jpeg);
  background-size: cover;
  background-position-x: center;
  height: 100%;
}

/* Allow the flourishes to visually curve more closely around the text */
.adviceText {
  /* max-width: 686px; */
  position: absolute;
  font-size: 2rem;
  text-align: center;
  color: linen;
  font-family: fantasy;
  /* top: calc(50%); */
  height: 40%  !important;
  bottom: 0;
  /* transform: translateY(80%); */
}

/* Animate new advice being added and old advice being removed */
.fadeIn {
  animation: fadeIn ease 10s;
}
.fadeOut {
  animation: fadeOut ease 10s;
}

@keyframes fadeIn {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
@keyframes fadeOut {
  0% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}
</style>
<template>
  <IonPage>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title class="header">A DAILY DOSE OF ADVICE </ion-title>
      </ion-toolbar>
    </ion-header>

    <IonContent>
      <div class="Home">
        <ion-card>
          <p class="adviceText" :class="animationState">{{ advice }}</p>
        </ion-card>
        <!-- <img src="../../public/assets/AdobeStock_301304275.jpeg" alt="dots" /> -->
        <p class="adviceText" :class="animationState">{{ advice }}</p>
        <!-- <img src="../../public/assets/happyDog.png" alt="happyDog" /> -->
      </div>
    </IonContent>
  </IonPage>
</template>

<script lang="ts">
// use HTTP plugin to bypass CORS and make successfull HTTP requests
import "@capacitor-community/http";
import { Plugins } from "@capacitor/core";
const { Http } = Plugins;

import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
} from "@ionic/vue";

import { defineComponent } from "vue";

export default defineComponent({
  name: "Home",
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
  },
  data: () => ({
    advice: "",
    animationState: "",
    hourToFetchNewAdvice: new Date().getHours(),
    lastSaveDate: null,
    today: new Date(),
  }),
  computed: {
    currentDate(): any {
      return this.today.getDate();
    },

    currentHour(): number {
      return this.today.getHours();
    },

    hourToEraseCurrentAdvice(): any {
      let oneHourPrior = 0;

      if (this.hourToFetchNewAdvice !== null) {
        oneHourPrior = this.hourToFetchNewAdvice - 1;
        if (oneHourPrior < 0) oneHourPrior = 23;
        return oneHourPrior;
      }
      return oneHourPrior;
    },
  },

  // When we'll check if advice data needs to be changed
  async ionViewWillEnter() {
    // If we haven't stored an hourToFetchNewAdvice before, calculate and store that and hourToEraseCurrentAdvice
    if (this.hourToFetchNewAdvice) this.hourToFetchNewAdvice = this.currentHour;

    this.updateAdvice();
  },

  methods: {
    async fetchAdvice(): Promise<any> {
      return await Http.request({
        method: "GET",
        url: "https://api.adviceslip.com/advice",
      }).then(({ data }) => {
        //fade in text
        this.animationState = "fadeIn";
        this.resetAnimationState();

        //save fetched advice
        this.advice = JSON.parse(data).slip.advice.toUpperCase();

        //update variable
        this.lastSaveDate = this.currentDate;
      });
    },

    updateAdvice(): void {
      //24 hours have passed
      if (
        this.currentHour === this.hourToFetchNewAdvice &&
        this.currentDate != this.lastSaveDate
      )
        this.fetchAdvice();
      // If 23 hours have passed, start fading out current advice
      else if (
        this.currentHour === this.hourToEraseCurrentAdvice &&
        this.advice
      ) {
        // Set dynamic class to fade out text
        this.animationState = "fadeOut";
        this.resetAnimationState();

        // Clear advice from state after the fade out animation ends
        setTimeout(() => {
          this.advice = "";
        }, 10000);
      }

      // Check every 10m if it's time to fetch/erase advice
      setTimeout(this.updateAdvice, 600000);
    },

    // Clear animation from advice after one playthrough
    // A safer approach might be to listen for `transistionend`
    resetAnimationState(): void {
      setTimeout(() => {
        this.animationState = "";
      }, 10000);
    },
  },
});
</script>

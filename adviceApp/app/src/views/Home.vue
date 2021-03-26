<style scoped>
.header {
  font-family: "HovdenStitch";
  /* padding: 2rem; */
  display: flex;
  text-align: center;
  font-size: 70%;
}

.displayImage {
  /* display: grid; */
  background-image: url(/img/BackgroundImg.b4af3dae.jpeg);
  background-size: cover;
  background-position: top;
  background-origin: content-box;
  background-repeat: no-repeat;
}
.avatarSvg {
  background: floralwhite;
}
#paddingRow {
  padding: 20%;
  visibility: hidden;
}
#textRow {
  padding-top: 50%;
  padding-bottom: 10%;
  text-align: center;
}

#textContainer {
  line-height: normal;
  text-align: center;
  font-family: cursive;
}
/* center text in the bottom-center of the background image  */

#footerRow {
  text-align: end;
}
</style>

<template>
  <IonPage name="Home">
    <IonHeader>
      <IonToolbar>
        <IonTitle class="header">A DAILY DOSE OF ADVICE </IonTitle>
      </IonToolbar>
    </IonHeader>

    <IonContent className="ion-padding" color="dark">
      <IonCard color="light">
        <IonItem>
          <IonAvatar slot="start" class="avatarSvg">
            <img src="../../public/assets/person.svg" />
          </IonAvatar>
          <IonLabel>
            <h3>{{ day }}</h3>
            <p>{{ formattedDate }}</p>
          </IonLabel>
        </IonItem>
        <IonCardContent class="displayImage">
          <!-- <IonCol > -->
          <IonRow
            class="align-content-center ion-padding-vertical"
            id="paddingRow"
          >
            {{ "advice" }}
          </IonRow>
          <IonRow
            class="align-content-center ion-padding-vertical"
            id="textRow"
          >
            <IonCol size="2" class="ion-padding-vertical">
              <div></div>
            </IonCol>
            <IonCol
              class="ion-text-md-center ion-text-capitalize ion-padding-vertical"
            >
              <ion-text size="auto" id="textContainer">
                {{ advice }}
              </ion-text>
            </IonCol>

            <IonCol size="2" class="ion-padding-vertical">
              <div></div>
            </IonCol>
          </IonRow>
        </IonCardContent>

        <IonFooter>
          <IonRow id="footerRow">
            <IonCol text-center>
              <Clock></Clock>
            </IonCol>
          </IonRow>
        </IonFooter>
      </IonCard>
    </IonContent>
  </IonPage>
</template>

<script lang="ts">
// use HTTP plugin to bypass CORS and make successfull HTTP requests
import "@capacitor-community/http";
import { defineComponent } from "vue";
import { Plugins } from "@capacitor/core";
import Clock from "./Clock.vue";
import moment from "moment";
const { Http } = Plugins;

import {
  IonAvatar,
  IonCard,
  IonCardContent,
  IonCol,
  IonContent,
  IonFooter,
  IonHeader,
  IonLabel,
  IonPage,
  IonRow,
  IonTitle,
  IonToolbar,
  IonItem,
} from "@ionic/vue";

export default defineComponent({
  name: "Home",
  components: {
    Clock,
    IonAvatar,
    IonCard,
    IonCardContent,
    IonCol,
    IonContent,
    IonFooter,
    IonHeader,
    // IonIcon,
    IonLabel,
    IonPage,
    IonRow,
    IonTitle,
    IonToolbar,
    IonItem,
    // IonItemDivider,
    // IonItemGroup,
  },
  data: (): {
    advice: string;
    animationState: string;
    hourToFetchNewAdvice: number;
    lastSaveDate: any;
    today: Date;
    day: string;
    formattedDate: string;
  } => ({
    advice: "",
    animationState: "",
    hourToFetchNewAdvice: new Date().getHours(),
    lastSaveDate: null,
    today: new Date(),
    day: moment().format("dddd"),
    formattedDate: moment().format("MM/DD/YYYY"),
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
  async ionViewWillEnter(): Promise<void> {
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
        this.advice = JSON.parse(data).slip.advice;

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
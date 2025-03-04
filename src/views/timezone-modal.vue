<template>
  <ion-header>
    <ion-toolbar>
      <ion-buttons slot="start">
        <ion-button @click="closeModal"> 
          <ion-icon slot="icon-only" :icon="closeOutline" />
        </ion-button>
      </ion-buttons>
      <ion-title>{{ $t("Select time zone") }}</ion-title>
    </ion-toolbar>
    <ion-toolbar>
      <ion-searchbar @ionFocus="selectSearchBarText($event)" :placeholder="$t('Search time zones')"  v-model="queryString" @ionInput="findTimeZone()"></ion-searchbar>
    </ion-toolbar>
  </ion-header>

  <ion-content class="ion-padding">
    <!-- Empty state -->
    <div class="empty-state" v-if="filteredTimeZones.length === 0">
      <p>{{ $t("No time zone found")}}</p>
    </div>

    <!-- Timezones -->
    <div v-else>
      <ion-list>
        <ion-radio-group value="rd" v-model="timeZoneId">
          <ion-item  v-bind:key="timeZone.id" v-for="timeZone in filteredTimeZones">
            <ion-label>{{ timeZone.label }} ({{ timeZone.id }})</ion-label>
            <ion-radio :value="timeZone.id" slot="start"></ion-radio>
          </ion-item>
        </ion-radio-group>
      </ion-list>
    </div>
    
    <ion-fab vertical="bottom" horizontal="end" slot="fixed">
      <ion-fab-button :disabled="!timeZoneId" @click="saveAlert">
        <ion-icon :icon="save" />
      </ion-fab-button>
    </ion-fab>
  </ion-content>
</template>

<script lang="ts">
import { 
  IonButtons,
  IonButton,
  IonContent,
  IonFab,
  IonFabButton,
  IonHeader,
  IonItem,
  IonIcon,
  IonLabel,
  IonRadioGroup,
  IonRadio,
  IonList,
  IonSearchbar,
  IonTitle,
  IonToolbar,
  modalController,
  alertController } from "@ionic/vue";
import { defineComponent } from "vue";
import { closeOutline, save } from "ionicons/icons";
import { useStore } from "@/store";
import { UserService } from "@/services/UserService";
import { hasError } from '@/utils'
import moment from 'moment';
import "moment-timezone";

export default defineComponent({
  name: "TimeZoneModal",
  data() {
    return {
      queryString: '',
      filteredTimeZones: [],
      timeZones: [],
      timeZoneId: ''
    }
  },
  computed: {
  },
  methods: {
    closeModal() {
      modalController.dismiss({ dismissed: true });
    },
    async saveAlert() {
      const message = this.$t("Are you sure you want to change the time zone to?", { timeZoneId: this.timeZoneId });
      const alert = await alertController.create({
        header: this.$t("Update time zone"),
        message,
        buttons: [
            {
              text: this.$t("Cancel"),
            },
            {
              text: this.$t("Confirm"),
              handler: () => {
                this.setUserTimeZone();
                }
              }
            ],
      });
      return alert.present();
    },
    escapeRegExp(text: string) {
      //TODO Handle it in a better way
      // Currently when the user types special character as it part of Regex expressions it breaks the code
      // so removed the characters for now
      return text.replace(/[-[\]{}()*+?.,\\^$|#\s]/g, '\\$&');
    },
    findTimeZone() { 
      const regularExp = new RegExp(`${this.escapeRegExp(this.queryString)}`, 'i');
      this.filteredTimeZones = this.timeZones.filter((timeZone: any) => {
        return regularExp.test(timeZone.id) || regularExp.test(timeZone.label);
      });
    },
    async getAvailableTimeZones() {
      UserService.getAvailableTimeZones().then((resp: any) => {
        if (resp.status === 200 && !hasError(resp)) {
          // TODO FIx this as fixed on backend
          // Currently backend API returns some of the legacy timezones which are not found in moment list
          // Due to which if we set them we get an error
          // Filtered them out till it is fixed at backend
          this.timeZones = resp.data.filter((timeZone: any) => {
            return moment.tz.zone(timeZone.id);
          });
          this.findTimeZone();
        }
      })
    },
    selectSearchBarText(event: any) {
      event.target.getInputElement().then((element: any) => {
        element.select();
      })
    },
    async setUserTimeZone() {
      return this.store.dispatch("user/setUserTimeZone", {
          "tzId": this.timeZoneId
      }).then(() => {
        this.closeModal()
      })
    }
  },
  beforeMount () {
    this.getAvailableTimeZones();
  },
  setup() {
    const store = useStore();
    return {
      closeOutline,
      save,
      store
    };
  },
  components: { 
    IonButtons,
    IonButton,
    IonContent,
    IonFab,
    IonFabButton,
    IonHeader,
    IonIcon,
    IonItem,
    IonLabel,
    IonList,
    IonRadioGroup,
    IonRadio,
    IonSearchbar,
    IonTitle,
    IonToolbar 
    },
});
</script>

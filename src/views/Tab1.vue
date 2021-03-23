<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="end">
          <ion-button router-link="/tabs/tab1/edit">
            <ion-icon :icon="add"></ion-icon>
          </ion-button>
        </ion-buttons>
        <ion-title>Notes</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Notes</ion-title>
        </ion-toolbar>
      </ion-header>
      <ion-item
        v-for="(note, idx) of notes"
        :key="idx"
        :routerLink="'/tabs/tab1/edit/' + note"
        >{{ note }}</ion-item
      >
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonItem,
  IonButtons,
  IonButton,
  IonIcon,
} from '@ionic/vue';

import { add } from 'ionicons/icons';
import { Filesystem, FilesystemDirectory } from '@capacitor/filesystem';
import { defineComponent, onMounted, ref } from 'vue';

export default defineComponent({
  name: 'Tab1',
  components: {
    IonItem,
    IonHeader,
    IonToolbar,
    IonTitle,
    IonContent,
    IonPage,
    IonButtons,
    IonButton,
    IonIcon,
  },
  ionViewDidEnter() {
    this.initFileSystem();
  },
  setup() {
    const notes = ref<string[]>([]);

    const noop = () => true;

    const makeDir = async () => {
      return await Filesystem.mkdir({
        path: 'notes',
        directory: FilesystemDirectory.Documents,
      });
    };

    const readDir = async () => {
      return await Filesystem.readdir({
        path: 'notes',
        directory: FilesystemDirectory.Documents,
      });
    };
    const initFileSystem = () => {
      makeDir()
        .then(noop, noop)
        .then(readDir)
        .then(({ files }) => {
          notes.value = files.sort(
            (a, b) =>
              parseInt(b.replace(/note-/, '').replace(/.txt/, '')) -
              parseInt(a.replace(/note-/, '').replace(/.txt/, ''))
          );
        });
    };
    onMounted(async () => {
      await Filesystem.requestPermissions();
      initFileSystem();
    });
    return { notes, add, initFileSystem };
  },
});
</script>

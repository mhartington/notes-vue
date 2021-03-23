<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-back-button default-href="/tabs/tab1"></ion-back-button>
        </ion-buttons>
        <ion-buttons slot="end">
          <ion-button @click="saveFile">
            <ion-icon :icon="save"></ion-icon>
          </ion-button>
          <ion-button v-if="route.params.id" @click="deleteFile">
            <ion-icon :icon="trashBin"></ion-icon>
          </ion-button>
        </ion-buttons>
        <ion-title>{{ title }}</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">{{ title }}</ion-title>
        </ion-toolbar>
      </ion-header>
      <note-editor v-model="content"></note-editor>
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
  IonButton,
  IonButtons,
  IonIcon,
  IonBackButton,
} from '@ionic/vue';
import { save, trashBin } from 'ionicons/icons';
import {
  Filesystem,
  FilesystemDirectory,
  FilesystemEncoding,
} from '@capacitor/filesystem';
import { defineComponent, onMounted, ref } from 'vue';
import NoteEditor from '../components/NoteEditor.vue';
import { useRoute, useRouter } from 'vue-router';
export default defineComponent({
  name: 'Editor',
  components: {
    IonHeader,
    IonToolbar,
    IonTitle,
    IonContent,
    IonPage,
    NoteEditor,
    IonButton,
    IonButtons,
    IonIcon,
    IonBackButton,
  },
  setup() {
    const content = ref<string | null>(null);
    const route = useRoute();
    const router = useRouter();
    const title = route.params.id ? 'Edit' : 'New';

    const saveFile = async () => {
      const file = route.params.id ? route.params.id : `note-${Date.now()}.txt`;
      await Filesystem.writeFile({
        path: `notes/${file}`,
        data: content.value || '',
        directory: FilesystemDirectory.Documents,
        encoding: FilesystemEncoding.UTF8,
      });
      router.back();
    };

    const deleteFile = async () => {
      const file = route.params.id;
      if (file) {
        await Filesystem.deleteFile({
          path: `notes/${file}`,
          directory: FilesystemDirectory.Documents,
        });
      }
      router.back();
    };
    onMounted(async () => {
      const file = route.params.id;

      if (file) {
        const { data } = await Filesystem.readFile({
          path: `notes/${file}`,
          directory: FilesystemDirectory.Documents,
          encoding: FilesystemEncoding.UTF8,
        });
        content.value = data;
      }
    });

    return {
      content,
      save,
      saveFile,
      deleteFile,
      trashBin,
      route,
      title,
    };
  },
});
</script>

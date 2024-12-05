<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>HOME</ion-title>
      <!-- Logout Button -->
        <ion-button slot="end" fill="clear" @click="logout" style="--color: gray;">
            <ion-icon slot="end" :icon="exit"></ion-icon>
            <ion-label>Logout</ion-label>
         </ion-button>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <!-- Komponen Refresher -->
      <ion-refresher
        slot="fixed"
        :pull-factor="0.5"
        :pull-min="100"
        :pull-max="200"
        @ionRefresh="handleRefresh($event)"
      >
        <ion-refresher-content></ion-refresher-content>
      </ion-refresher>

      <!-- Kata Sambutan -->
      <div class="welcome-message">
        <h2>Selamat Datang Di Aplikasi Toys Story</h2>
      </div>

      <!-- Tombol Tambahkan Data di Tengah -->
      <div class="button-container">
        <ion-button expand="block" @click="isOpen = true">
          Tambahkan Data Mainan
        </ion-button>
      </div>

      <!-- Modal untuk Input -->
      <InputModal
        v-model:isOpen="isOpen"
        v-model:editingId="editingId"
        :mainan="mainan"
        @submit="handleSubmit"
      />
    </ion-content>
  </ion-page>
</template>


<script setup lang="ts">
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  loadingController,
  IonRefresher,
  IonRefresherContent,
  toastController
} from '@ionic/vue';
import {
  checkmarkCircle,
  closeCircle,
  warningOutline,
  exit
} from 'ionicons/icons';
import InputModal from '@/components/InputModal.vue';
import { onMounted, ref, onUnmounted } from 'vue';
import { firestoreService, type Mainan } from '@/utils/firestore';
import { useAuthStore } from '@/stores/auth';

const authStore = useAuthStore();

const logout = () => {
    useAuthStore.logout();
};

// modifikasi src/views/HomePage.vue deklarasikan variabel yang akan digunakan
const isOpen = ref(false);
const editingId = ref<string | null>(null);
const mainans = ref<Mainan[]>([]);
const mainan = ref<Omit<Mainan, 'id' | 'createdAt' | 'updatedAt' | 'status'>>({
  title: '',
  description: '',
});
const itemRefs = ref<Map<string, HTMLIonItemSlidingElement>>(new Map());
let intervalId: any;
const timeUpdateTrigger = ref(0);


// toast notifikasi
const showToast = async (message: string, color: string = 'success', icon: string = checkmarkCircle) => {
  const toast = await toastController.create({
    message,
    duration: 2000,
    color,
    position: 'top',
    icon
  });
  await toast.present();
};

// handle swipe refresher
const handleRefresh = async (event: any) => {
  try {
    await loadMainans(false);
  } catch (error) {
    console.error('Error refreshing:', error);
  } finally {
    event.target.complete();
  }
};

// handle submit add/edit pada modal
const handleSubmit = async (mainan: Omit<Mainan, 'id' | 'createdAt' | 'updatedAt' | 'status'>) => {
  if (!mainan.title) {
    await showToast('Title is required', 'warning', warningOutline);
    return;
  }
  try {
    if (editingId.value) {
      await firestoreService.updateMainan(editingId.value, mainan as Mainan);
      await showToast('Data updated successfully', 'success', checkmarkCircle);
    } else {
      await firestoreService.addMainan(mainan as Mainan);
      await showToast('Data added successfully', 'success', checkmarkCircle);
    }
    loadMainans();
  } catch (error) {
    await showToast('An error occurred', 'danger', closeCircle);
    console.error(error);
  } finally {
    editingId.value = null;
  }
};

// handle edit click
const handleEdit = async (editMainan: Mainan) => {
  const slidingItem = itemRefs.value.get(editMainan.id!);
  await slidingItem?.close();

  editingId.value = editMainan.id!;
  mainan.value = {
    title: editMainan.title,
    description: editMainan.description,
  }
  isOpen.value = true;
}

// handle delete click/swipe
const handleDelete = async (deleteMainan: Mainan) => {
  try {
    await firestoreService.deleteMainan(deleteMainan.id!);
    await showToast('Data deleted successfully', 'success', checkmarkCircle);
    loadMainans();
  } catch (error) {
    await showToast('Failed to delete Data', 'danger', closeCircle);
    console.error(error);
  }
};

// load data
const loadMainans = async (isLoading = true) => {
  let loading;
  if (isLoading) {
    loading = await loadingController.create({
      message: 'Loading...'
    });
    await loading.present();
  }

  try {
    mainans.value = await firestoreService.getMainans();
  } catch (error) {
    console.error(error);
  } finally {
    if (loading) {
      await loading.dismiss();
    }
  }
};

// dijalankan setiap halaman diload, load data dan set interval update 1 menit
onMounted(() => {
  loadMainans();
  intervalId = setInterval(() => {
    timeUpdateTrigger.value++;
  }, 60000);
});

// reset interval update
onUnmounted(() => {
  clearInterval(intervalId);
});
</script>

<!-- modifikasi src/views/HomePage.vue tambahkan keseluruhan style -->
<style scoped>
ion-card {
  width: 100%;
}

ion-fab {
  margin: 25px;
}

.limited-text {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-box-orient: vertical;
}

ion-card-title.limited-text {
  -webkit-line-clamp: 1;
  line-clamp: 1;
}

ion-card-subtitle.limited-text {
  -webkit-line-clamp: 2;
  line-clamp: 2;
}

.welcome-message {
  text-align: center;
  margin-top: 20px;
  font-size: 1.5em;
}

.button-container {
  display: flex;
  margin-top: -150px;
  justify-content: center;
  align-items: center;
  height: 100vh;
  margin-bottom: -100px
}

ion-item {
  --padding-start: 0;
  --inner-padding-end: 0;
}

</style>
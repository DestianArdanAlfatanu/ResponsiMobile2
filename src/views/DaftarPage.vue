<template>
    <ion-page>
      <ion-header :translucent="true">
        <ion-toolbar>
          <ion-title>DAFTAR MAINAN</ion-title>
          <!-- Logout Button -->
        <ion-button slot="end" fill="clear" @click="logout" style="--color: gray;">
            <ion-icon slot="end" :icon="exit"></ion-icon>
            <ion-label>Logout</ion-label>
         </ion-button>
        </ion-toolbar>
      </ion-header>
  
      <!-- modifikasi src/views/HomePage.vue pada bagian ion-content dalam template -->
  <ion-content :fullscreen="true">
      <!-- komponen paling atas dari ion-content -->
      <ion-refresher slot="fixed" :pull-factor="0.5" :pull-min="100" :pull-max="200"
      @ionRefresh="handleRefresh($event)">
      <ion-refresher-content></ion-refresher-content>
    </ion-refresher>
  
    <!-- bagian refresher -->
  <!-- nomral mainans -->
  <div class="scrollable-container">
    <ion-list>
      <ion-item-sliding v-for="mainan in activeMainans" :key="mainan.id" :ref="(el) => setItemRef(el, mainan.id!)">
        <ion-item-options side="start" @ionSwipe="handleDelete(mainan)">
          <ion-item-option color="danger" expandable @click="handleDelete(mainan)">
            <ion-icon slot="icon-only" :icon="trash" size="large"></ion-icon>
          </ion-item-option>
        </ion-item-options>
  
        <ion-item>
          <ion-card>
            <ion-card-header>
              <ion-card-title class="ion-text-wrap limited-text">{{ mainan.title }}</ion-card-title>
              <ion-card-subtitle class="limited-text">{{ mainan.description }}</ion-card-subtitle>
            </ion-card-header>
  
            <ion-card-content>
              <ion-badge>{{ getRelativeTime(mainan.updatedAt) }}</ion-badge>
            </ion-card-content>
          </ion-card>
        </ion-item>
  
        <ion-item-options side="end" @ionSwipe="handleStatus(mainan)">
          <ion-item-option @click="handleEdit(mainan)">
            <ion-icon slot="icon-only" :icon="create" size="large"></ion-icon>
          </ion-item-option>
          <ion-item-option color="success" expandable @click="handleStatus(mainan)">
            <ion-icon slot="icon-only" :icon="checkmarkCircle" color="light" size="large"></ion-icon>
          </ion-item-option>
        </ion-item-options>
      </ion-item-sliding>
      <ion-item v-if="activeMainans.length === 0" class="ion-text-center">
        <ion-label>Tidak Ada Data Mainan</ion-label>
      </ion-item>
    </ion-list>
  </div>
  
  <!-- completed mainans -->
  <ion-item class="accordion-container">
    <ion-accordion-group>
      <ion-accordion value="first">
        <ion-item slot="header" color="light">
          <ion-label class="ion-text-center">Favorit</ion-label>
        </ion-item>
        <div slot="content" class="scrollable-container">
          <ion-list>
            <ion-item-sliding v-for="mainan in completedMainans" :key="mainan.id" :ref="(el) => setItemRef(el, mainan.id!)">
              <ion-item-options side="start" @ionSwipe="handleDelete(mainan)">
                <ion-item-option color="danger" expandable @click="handleDelete(mainan)">
                  <ion-icon slot="icon-only" :icon="trash" size="large"></ion-icon>
                </ion-item-option>
              </ion-item-options>
  
              <ion-item>
                <ion-card>
                  <ion-card-header>
                    <ion-card-title class="ion-text-wrap limited-text">{{ mainan.title }}</ion-card-title>
                    <ion-card-subtitle class="limited-text">{{ mainan.description }}</ion-card-subtitle>
                  </ion-card-header>
  
                  <ion-card-content>
                    <ion-badge>{{ getRelativeTime(mainan.updatedAt) }}</ion-badge>
                  </ion-card-content>
                </ion-card>
              </ion-item>
  
              <ion-item-options side="end" @ionSwipe="handleStatus(mainan)">
                <ion-item-option @click="handleEdit(mainan)">
                  <ion-icon slot="icon-only" :icon="create" size="large"></ion-icon>
                </ion-item-option>
                <ion-item-option color="warning" expandable @click="handleStatus(mainan)">
                  <ion-icon slot="icon-only" :icon="close" color="light" size="large"></ion-icon>
                </ion-item-option>
              </ion-item-options>
            </ion-item-sliding>
            <ion-item v-if="completedMainans.length === 0" class="ion-text-center">
              <ion-label>Tidak Ada Mainan Favorit</ion-label>
            </ion-item>
          </ion-list>
        </div>
      </ion-accordion>
    </ion-accordion-group>
  </ion-item>
  
  <!-- bagian tombol dan modal -->
    <ion-fab vertical="bottom" horizontal="end" slot="fixed">
    </ion-fab>
    <InputModal v-model:isOpen="isOpen" v-model:editingId="editingId" :mainan="mainan" @submit="handleSubmit" />
  </ion-content>
    </ion-page>
  </template>
  
  <!-- modifikasi src/views/HomePage.vue import semua komponen yang diperlukan -->
  <script setup lang="ts">
  import {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonItem,
    IonItemOption,
    IonItemOptions,
    IonItemSliding,
    IonCard,
    IonCardHeader,
    IonCardTitle,
    IonCardSubtitle,
    IonCardContent,
    IonBadge,
    IonIcon,
    IonFab,
    IonAccordion,
    IonAccordionGroup,
    IonLabel,
    IonList,
    loadingController,
    IonRefresher,
    IonRefresherContent,
    toastController
  } from '@ionic/vue';
  import {
    checkmarkCircle,
    close,
    create,
    trash,
    closeCircle,
    warningOutline, 
    exit
  } from 'ionicons/icons';
  import InputModal from '@/components/InputModal.vue';
  import { onMounted, ref, computed, onUnmounted } from 'vue';
  import { firestoreService, type Mainan } from '@/utils/firestore';
  import { formatDistanceToNow } from 'date-fns';
  import { useAuthStore } from '@/stores/auth';


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
  const activeMainans = computed(() => mainans.value.filter(mainan => !mainan.status));
  const completedMainans = computed(() => mainans.value.filter(mainan => mainan.status));
  const itemRefs = ref<Map<string, HTMLIonItemSlidingElement>>(new Map());
  let intervalId: any;
  const timeUpdateTrigger = ref(0);
  
  // mendapatkan value dari item
  const setItemRef = (el: any, id: string) => {
    if (el) {
      itemRefs.value.set(id, el.$el);
    }
  };
  
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
  
  // mendapatkan perbedaan waktu
  const getRelativeTime = (date: any) => {
    timeUpdateTrigger.value;
    try {
      const jsDate = date?.toDate ? date.toDate() : new Date(date);
      return formatDistanceToNow(jsDate, { addSuffix: true });
    } catch (error) {
      return 'Invalid date';
    }
  };
  
  // handle swipe refresher
  const handleRefresh = async (event?: any) => {
  try {
    await loadMainans(false);
  } catch (error) {
    console.error('Error refreshing:', error);
  } finally {
    if (event) event.target.complete();
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
  
  // handle status click/swipe, mengubah status mainan normal (false)/favorit (true)
  const handleStatus = async (statusMainan: Mainan) => {
    const slidingItem = itemRefs.value.get(statusMainan.id!);
    await slidingItem?.close();
    try {
      await firestoreService.updateStatus(statusMainan.id!, !statusMainan.status);
      await showToast(
        `Data Mainan marked as ${!statusMainan.status ? 'favorit' : 'normal'}`,
        'success',
        checkmarkCircle
      );
      loadMainans();
    } catch (error) {
      await showToast('Failed to update status', 'danger', closeCircle);
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
  onMounted(async () => {
  await handleRefresh(); 
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
  ion-card,
  ion-accordion-group {
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
  
  .scrollable-container {
    max-height: 80vh;
    overflow-y: auto;
    overflow-x: hidden;
  }
  
  .accordion-container {
    --padding-start: 0;
    --inner-padding-end: 0;
  }
  
  ion-item {
    --padding-start: 0;
    --inner-padding-end: 0;
  }
  
  .scrollable-container::-webkit-scrollbar {
    width: 8px;
  }
  
  .scrollable-container::-webkit-scrollbar-track {
    background: #f1f1f1;
  }
  
  .scrollable-container::-webkit-scrollbar-thumb {
    background: #888;
    border-radius: 4px;
  }
  
  .scrollable-container::-webkit-scrollbar-thumb:hover {
    background: #555;
  }
  </style>
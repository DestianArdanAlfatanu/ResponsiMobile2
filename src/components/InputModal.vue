<template>
    <ion-modal :is-open="isOpen" @did-dismiss="cancel">
        <ion-header>
            <ion-toolbar>
                <ion-title>{{ editingId ? 'Edit' : 'Add' }} Data Mainan</ion-title>
                <ion-buttons slot="start">
                    <ion-button @click="cancel"><ion-icon :icon="close"></ion-icon></ion-button>
                </ion-buttons>
            </ion-toolbar>
        </ion-header>
        <ion-content>
            <ion-item>
                <ion-input v-model="localMainan.title" label="Nama Mainan" label-placement="floating"
                    placeholder="Masukkan Nama Mainan"></ion-input>
            </ion-item>
            <ion-item>
                <ion-textarea v-model="localMainan.description" label="Tentang Mainan" label-placement="floating"
                    placeholder="Masukkan Deskripsi Mainan" :autogrow="true" :rows="3"></ion-textarea>
            </ion-item>
            <ion-row>
                <ion-col>
                    <ion-button type="button" @click="input" shape="round" color="primary" expand="block">
                        {{ editingId ? 'Edit' : 'Add' }} Data Mainan
                    </ion-button>
                </ion-col>
            </ion-row>
        </ion-content>
    </ion-modal>
</template>
<script setup lang="ts">
import { ref, watch } from 'vue';
import { IonModal, IonHeader, IonToolbar, IonTitle, IonButtons, IonButton, IonIcon, IonInput, IonRow, IonCol, IonItem, IonContent, IonTextarea } from '@ionic/vue';
import { close } from 'ionicons/icons';
import { Mainan } from '@/utils/firestore';

const props = defineProps<{
    isOpen: boolean,
    editingId: string | null,
    mainan: Omit<Mainan, 'id' | 'createdAt' | 'updatedAt' | 'status'>
}>();

const emit = defineEmits<{
    'update:isOpen': [value: boolean],
    'update:editingId': [value: string | null],
    'submit': [item: Omit<Mainan, 'id' | 'createdAt' | 'updatedAt' | 'status'>]
}>();

// Create local copy of mainan
const localMainan = ref<Omit<Mainan, 'id' | 'createdAt' | 'updatedAt' | 'status'>>({
    title: '',
    description: ''
});

// Watch for changes in props.mainan and update localMainan
watch(() => props.mainan, (newMainan) => {
    localMainan.value = { ...newMainan };
}, { immediate: true, deep: true });

const cancel = () => {
    emit('update:isOpen', false);
    emit('update:editingId', null);
    localMainan.value = {
        title: '',
        description: ''
    };
}

const input = () => {
    emit('submit', localMainan.value);
    cancel();
}
</script>
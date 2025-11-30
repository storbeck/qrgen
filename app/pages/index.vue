<template>
    <v-container>
        <div class="d-flex gap-4">
            <v-text-field 
                v-model="name" 
                :disabled="pending" 
                label="Message to encode" 
                flat
                variant="solo"
                class="border"
                hide-details
            >
                <template #append>
                    <v-btn 
                        :disabled="pending"
                        color="primary"
                        type="submit"
                        @click="generateQRCode"
                    >
                        Generate QRCode
                    </v-btn>
                </template>
            </v-text-field>    
        </div>
        <v-expansion-panels>
            <v-expansion-panel>
                <template #title>
                    Advanced Options
                </template>
                <template #text>
                    <div>
                        <!-- width, lightcolor, darkcolor -->
                        <v-text-field 
                            v-model="width"
                            label="Width (px)" 
                            flat
                            variant="solo"
                            class="border mb-4"
                            hide-details
                        />
                        <v-text-field 
                            v-model="lightColor"
                            label="Light Color (hex)" 
                            flat
                            variant="solo"
                            class="border mb-4"
                            hide-details
                        />
                        <v-text-field 
                            v-model="darkColor"
                            label="Dark Color (hex)" 
                            flat
                            variant="solo"
                            class="border mb-4"
                            hide-details
                        />
                    </div>
                </template>
            </v-expansion-panel>
        </v-expansion-panels>
        <v-progress-circular
            v-if="pending"
            indeterminate
            color="primary"
        />
        <img 
            v-if="data" 
            :src="data" 
            alt="QR Code"
        >
    </v-container>
</template>

<script setup lang="ts">
const name = ref('');
const message = ref('');
const data = ref('')
const pending = ref(false);
const width = ref(300);
const darkColor = ref('#000000');
const lightColor = ref('#FFFFFF');
const errorMessage = ref('');

async function generateQRCode() {
    if (!name.value.trim()) return;
    pending.value = true;
    message.value = '';
    try {
        const widthValue = Number(width.value);
        const hasValidWidth =  Number.isFinite(widthValue) && widthValue >= 50 && widthValue <= 1000;
        const dark = darkColor.value.trim();
        const light = lightColor.value.trim();

        const body: QRCodeObject = {
            message: name.value.trim(),
        };

        if (hasValidWidth) {
            body.width = widthValue;
        }

        // Only send colors if both are provided; zod requires both keys when color is present.
        if (dark && light) {
            body.color = { dark, light };
        }

        const response = await $fetch(`/api/encode`, {
            method: 'POST',
            body,
        });
        data.value = response ?? '';
    } catch (error) {
        message.value = 'Error generating QR code';
        errorMessage.value = '';
        if (error.statusCode === 400) {
            errorMessage.value = error.statusMessage || 'Invalid input';
        }
        console.error(error);
    } finally {
        pending.value = false;
    }
}
</script>

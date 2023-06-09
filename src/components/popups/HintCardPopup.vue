<script setup lang="ts">
import type { HintCardData } from "@/config/cards";
import { getOptionallyTranslatedText, HintCardFormat } from "@/config/cards";
import Encryption from "@/components/encryption/Encryption.vue";
import { ref, watch } from "vue";
import { useI18n } from "vue-i18n";

const props= defineProps({
  card: {
    type: Object,
  },
  justCollected: {
    type: Boolean,
  },
  previouslyDecoded: {
    type: Boolean,
  }
});
const emit = defineEmits(["close", "earned"]);

const { locale } = useI18n({ useScope: "global" });

const currentCardIndex = ref(0);

watch([props.card], ([currentCard]) => {
  if (currentCard) {
    console.info('Current card:', currentCard);
    console.info('Encryption present?', currentCard.hintCards.find( (x: any) => x.encryption))
    const hasEncryption = currentCard.hintCards.find( (x :any) => x.encryption);
    if (!hasEncryption) {
      emit('earned');
    }
  }
});

</script>
<template>
  <div
    class="fixed top-0 left-0 w-full z-30 h-full bg-black text-white flex flex-col"
  >
    <!-- header -->
    <div
      class="flex flex-row items-center bg-gradient-to-b from-brown to-brown/75 space-between w-full px-3 py-2"
    >
      <a class="flex flex-col flex-1 text-orange" target="_blank">
        <h1>
          {{ getOptionallyTranslatedText(props.card?.title ?? "", locale) }}
        </h1>
        <span class="text-white/75"
          >{{ getOptionallyTranslatedText(props.card?.name ?? "", locale) }}
          <template v-if="props.card?.externalSite?.href">
            —
            <a
              class="text-orange/75"
              :href="props.card?.externalSite?.href"
              target="_blank"
              >{{ $t("popups.hintCard.follow") }}</a
            ></template
          ></span
        >
      </a>
      <button
        v-if="!justCollected || (props.card?.hintCards?.length ?? 0 > 1)"
        class="h-full border border-orange bg-gradient-to-b from-orange/25 to-orange/20 text-orange px-6 flex items-center justify-center"
        @click="emit('close')"
      >
        {{ $t("popups.close") }}
      </button>
      <button
        v-else
        class="h-full border border-orange bg-gradient-to-b from-orange/25 to-orange/20 text-orange px-6 flex items-center justify-center"
        @click="emit('close')"
      >
        {{ $t("popups.close") }}
      </button>
    </div>

    <!-- body -->
    <div class="flex flex-1 flex-shrink-1 overflow-auto">
      <!-- card type: text -->
      <template
        v-if="
          props.card?.hintCards[currentCardIndex].cardFormat ===
          HintCardFormat.Text
        "
      >
        <div class="font-hand w-full h-full overflow-auto p-4 flex-col">
          <div
            v-html="
              getOptionallyTranslatedText(
                props.card?.hintCards[currentCardIndex].text ?? '',
                locale
              )
            "
          ></div>
        </div>
        <!-- encryption -->
        <div
          v-if="props.card?.hintCards[currentCardIndex].encryption"
          class="flex flex-col w-full h-full"
        >
          <Encryption
            :correctPassword="props.card?.hintCards[currentCardIndex].encryption?.correctPassword"
            :decodedMessage="props.card?.hintCards[currentCardIndex].encryption?.decodedMessage"
            :collected="props.previouslyDecoded"
            @decrypted="emit('earned')"
          />
        </div>
      </template>

      <!-- card type: text + image | image is 1:1 aspect ratio -->
      <template
        v-else-if="props.card?.hintCards[currentCardIndex].cardFormat === HintCardFormat.ImageText"
      >
        <div class="flex flex-col w-full h-full">
          <!-- image -->
          <div class="card-image-text frame flex-grow-0 flex-shrink-0 p-4">
            <div
              class="card-image-text image"
              :style="{
                'background-image': `url(${props.card?.hintCards[currentCardIndex].imageUrl})`,
              }"
            ></div>
          </div>

          <!-- text -->
          <div class="card-image-text text-frame flex-1 overflow-auto m-4 mt-0 pt-4 border-t border-orange">
            <div
              v-html="
                getOptionallyTranslatedText(
                  props.card?.hintCards[currentCardIndex].text ?? '',
                  locale
                )
              "
            ></div>
          </div>
          <!-- encryption -->
          <div
            v-if="props.card?.hintCards[currentCardIndex].encryption"
            class="flex flex-col w-full h-full"
          >
            <Encryption
              :correctPassword="props.card?.hintCards[currentCardIndex].encryption?.correctPassword"
              :decodedMessage="props.card?.hintCards[currentCardIndex].encryption?.decodedMessage"
              :collected="props.previouslyDecoded"
              @decrypted="emit('earned')"
            />
          </div>
        </div>
      </template>

      <!-- card type: image only -->
      <template v-else>
        <div class="card-image frame w-full h-full flex-col">
          <div
            class="card-image image w-full h-full"
            :style="{
              'background-image': `url(${props.card?.hintCards[currentCardIndex].imageUrl})`,
            }"
          ></div>
          <!-- encryption -->
          <div
            v-if="props.card?.hintCards[currentCardIndex].encryption"
            class="flex flex-col w-full h-full"
          >
            <Encryption
              :correctPassword="props.card?.hintCards[currentCardIndex].encryption?.correctPassword"
              :decodedMessage="props.card?.hintCards[currentCardIndex].encryption?.decodedMessage"
              :collected="props.previouslyDecoded"
              @decrypted="emit('earned')"
            />
          </div>
        </div>
      </template>
    </div>
    <!-- card has encryption -->
    <!-- navigation in case of multi-cards -->
    <div
      v-if="props.card?.hintCards?.length ?? 0 > 1"
      class="flex flex-row justify-between bg-gradient-to-b from-brown to-brown/75 h-[3rem] items-center w-full px-4 leading-none"
    >
      <div
        class="w-[2rem] h-[2rem] border-orange bg-gradient-to-b from-orange/25 to-orange/20 text-orange flex items-center justify-center"
        @click="currentCardIndex = Math.max(0, currentCardIndex - 1)"
      >
        ←
      </div>

      <div>{{ currentCardIndex + 1 }} / {{ props.card?.hintCards.length }}</div>

      <div
        class="w-[2rem] h-[2rem] border-orange bg-gradient-to-b from-orange/25 to-orange/20 text-orange flex items-center justify-center"
        @click="currentCardIndex = Math.max(0, currentCardIndex + 1)"
      >
        →
      </div>
    </div>
  </div>
</template>
<style lang="postcss" scoped>
.card-image .image,
.card-image-text .image {
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
}

.card-image-text .image {
  display: block;
  width: 100%;
  aspect-ratio: 1;
}
</style>

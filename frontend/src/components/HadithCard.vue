<template>
  <article class="glass-panel rounded-xl p-6 transition-all duration-300 relative overflow-hidden group mb-6">
    <div class="absolute top-0 right-0 w-24 h-1 bg-gradient-to-l from-primary to-transparent opacity-50"></div>
    
    <header class="flex justify-between items-start mb-4">
      <div class="flex items-center gap-2">
        <div class="w-10 h-2 bg-primary/20 rounded-full overflow-hidden relative">
          <div class="absolute top-0 left-0 h-full bg-primary" :style="{ width: scorePercentage }"></div>
        </div>
        <span class="font-label-sm text-label-sm text-on-surface-variant">Skor: {{ formatScore(hadith.skor) }}</span>
      </div>
      <div class="flex items-center gap-3">
        <span class="font-label-sm text-label-sm text-primary uppercase tracking-wider bg-primary/10 px-3 py-1 rounded-full border border-primary/20">
          {{ hadith.kitap || 'Sahih Bukhari' }} #{{ hadith.hadis_no }}
        </span>
      </div>
    </header>

    <div class="flex flex-col gap-6">
      <!-- Arabic -->
      <div class="text-right" dir="rtl" v-if="hadith.arapca">
        <p class="font-label-sm text-label-sm text-on-surface-variant mb-2" v-if="hadith.arapca.sanad">
          {{ hadith.arapca.sanad }}
        </p>
        <p class="font-arabic-display text-arabic-display text-on-surface leading-relaxed">
          {{ hadith.arapca.hadith_detail || 'Arapça metin bulunamadı.' }}
        </p>
      </div>

      <div class="w-full h-px bg-gradient-to-r from-transparent via-primary/30 to-transparent" v-if="hadith.arapca && hadith.ingilizce"></div>

      <!-- English -->
      <div class="text-left" dir="ltr" v-if="hadith.ingilizce">
        <p class="font-label-sm text-label-sm text-on-surface-variant mb-2" v-if="hadith.ingilizce.sanad">
          {{ hadith.ingilizce.sanad }}
        </p>
        <p class="font-body-main text-body-main text-on-surface-variant leading-relaxed">
          {{ hadith.ingilizce.hadith_detail || 'English text not available.' }}
        </p>
      </div>
    </div>

    <!-- Hover Menu -->
    <footer class="mt-6 pt-4 border-t border-outline-variant/30 flex justify-between gap-2 opacity-0 group-hover:opacity-100 transition-opacity duration-300">
      <a v-if="hadith.kaynak_link" :href="hadith.kaynak_link" target="_blank" class="text-xs text-primary/70 hover:text-primary transition-colors flex items-center">
        Sunnah.com'da Görüntüle &rarr;
      </a>
      <div class="flex justify-end gap-2">
        <button class="p-2 rounded-full hover:bg-surface-container-high text-on-surface-variant hover:text-primary transition-colors flex items-center justify-center">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"></path></svg>
        </button>
        <button class="p-2 rounded-full hover:bg-surface-container-high text-on-surface-variant hover:text-primary transition-colors flex items-center justify-center">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8.684 13.342C8.886 12.938 9 12.482 9 12c0-.482-.114-.938-.316-1.342m0 2.684a3 3 0 110-2.684m0 2.684l6.632 3.316m-6.632-6l6.632-3.316m0 0a3 3 0 105.367-2.684 3 3 0 00-5.367 2.684zm0 9.316a3 3 0 105.368 2.684 3 3 0 00-5.368-2.684z"></path></svg>
        </button>
        <button class="p-2 rounded-full hover:bg-surface-container-high text-on-surface-variant hover:text-primary transition-colors flex items-center justify-center">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z"></path></svg>
        </button>
      </div>
    </footer>
  </article>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  hadith: {
    type: Object,
    required: true
  }
});

const formatScore = (score) => {
  return Number(score).toFixed(4);
};

// Skoru 0 ile 0.1 arasında varsayarak yüzdelik bar ayarlayalım
const scorePercentage = computed(() => {
  const score = Number(props.hadith.skor) || 0;
  let percent = (score / 0.1) * 100;
  if(percent > 100) percent = 100;
  if(percent < 5) percent = 5;
  return `${percent}%`;
});
</script>

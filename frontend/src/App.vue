<template>
  <div class="bg-[#0F1923] text-on-background font-body-main antialiased min-h-screen flex flex-col">
    <!-- TopNavBar -->
    <nav class="fixed top-0 w-full z-50 bg-[#0F1923]/80 backdrop-blur-xl border-b border-[#C8A96E]/20 shadow-2xl shadow-black/50">
      <div class="flex justify-between items-center h-20 px-6 lg:px-12 max-w-[1140px] mx-auto">
        <div class="text-2xl font-bold text-[#C8A96E] tracking-tight font-['Inter'] antialiased">
          حديث | HadisAra
        </div>
        <!-- Desktop Nav -->
        <div class="hidden md:flex items-center gap-8">
          <a class="text-[#C8A96E] border-b-2 border-[#C8A96E] pb-1 font-body-main" href="#">Ana Sayfa</a>
          <a class="text-gray-400 hover:text-[#C8A96E] transition-colors font-body-main hover:bg-[#C8A96E]/10 duration-300 rounded px-2 py-1" href="#">Favoriler</a>
        </div>
        <!-- Actions -->
        <div class="hidden md:flex items-center gap-4">
          <button class="text-[#C8A96E] border border-[#C8A96E] px-4 py-2 rounded-full hover:bg-[#C8A96E]/10 transition-colors font-label-sm uppercase tracking-wider">Login</button>
          <button class="bg-[#C8A96E] text-[#0F1923] px-4 py-2 rounded-full hover:bg-[#e3c285] transition-colors font-label-sm uppercase tracking-wider font-bold">Signup</button>
        </div>
      </div>
    </nav>

    <!-- Main Content -->
    <main class="flex-grow pt-24 px-6 md:px-12 max-w-[1140px] mx-auto w-full flex flex-col gap-12 pb-24">
      
      <!-- HERO SECTION -->
      <section class="flex flex-col items-center justify-center min-h-[409px] text-center pt-12">
        <h1 class="font-headline-lg text-headline-lg text-[#C8A96E] mb-4">Hadis Arama Sistemi</h1>
        <p class="font-body-main text-on-surface-variant mb-8 max-w-2xl">
          Sahih Bukhari külliyatında AI tabanlı, çift dilli (Arapça & İngilizce) semantik arama
        </p>

        <!-- SearchBox Component -->
        <SearchBox :isLoading="isLoading" @search="onSearch" />

        <!-- Önerilen Kelimeler -->
        <div class="flex flex-wrap justify-center gap-3">
          <button @click="onSearch({query: 'Niyet', language: 'auto'})" class="px-4 py-2 rounded-full border border-outline-variant text-on-surface-variant font-label-sm hover:bg-primary/10 hover:border-primary hover:text-primary transition-all duration-300">Niyet</button>
          <button @click="onSearch({query: 'Prayer / Namaz', language: 'auto'})" class="px-4 py-2 rounded-full border border-outline-variant text-on-surface-variant font-label-sm hover:bg-primary/10 hover:border-primary hover:text-primary transition-all duration-300">Prayer / Namaz</button>
          <button @click="onSearch({query: 'الصبر', language: 'auto'})" class="px-4 py-2 rounded-full border border-outline-variant text-on-surface-variant font-label-sm hover:bg-primary/10 hover:border-primary hover:text-primary transition-all duration-300" dir="rtl">الصبر</button>
          <button @click="onSearch({query: 'Fasting', language: 'auto'})" class="px-4 py-2 rounded-full border border-outline-variant text-on-surface-variant font-label-sm hover:bg-primary/10 hover:border-primary hover:text-primary transition-all duration-300">Fasting</button>
        </div>
      </section>

      <!-- RESULTS SECTION -->
      <section v-if="searchResults.length > 0 || hasSearched" class="grid grid-cols-1 gap-6 w-full">
        <div v-if="error" class="p-4 bg-error-container text-on-error-container rounded-lg border border-error mb-4">
          🚨 {{ error }}
        </div>
        
        <div v-if="!isLoading && searchResults.length === 0 && hasSearched && !error" class="text-center py-12 text-on-surface-variant glass-panel rounded-xl">
          "{ { lastQuery } }" için hiçbir sonuç bulunamadı. Lütfen farklı kelimeler deneyin.
        </div>

        <div v-if="searchResults.length > 0 && !isLoading" class="flex justify-between items-center mb-2 px-2">
          <span class="text-on-surface-variant font-label-sm">{{ totalResults }} Hadis Bulundu</span>
          <span class="font-label-sm text-primary bg-primary/10 px-3 py-1 rounded-full border border-primary/20">Mod: {{ searchMode.toUpperCase() }}</span>
        </div>

        <!-- Hadith Cards -->
        <HadithCard 
          v-for="hadith in searchResults" 
          :key="hadith.id" 
          :hadith="hadith" 
        />
      </section>
    </main>

    <!-- Footer -->
    <footer class="bg-[#0F1923] border-t border-[#C8A96E]/10 mt-auto">
      <div class="flex flex-col md:flex-row justify-between items-center py-12 px-6 max-w-[1140px] mx-auto gap-6 text-sm font-['Inter']">
        <div class="text-lg font-bold text-[#C8A96E]">حديث | HadisAra</div>
        <div class="flex flex-wrap gap-6 justify-center">
          <a class="text-gray-500 hover:text-[#C8A96E] transition-colors cursor-pointer" href="#">Privacy Policy</a>
          <a class="text-gray-500 hover:text-[#C8A96E] transition-colors cursor-pointer" href="#">Terms of Service</a>
          <a class="text-gray-500 hover:text-[#C8A96E] transition-colors cursor-pointer" href="#">API Documentation</a>
          <a class="text-gray-500 hover:text-[#C8A96E] transition-colors cursor-pointer" href="#">Contact</a>
        </div>
        <div class="text-gray-500">
          © 2024 HadisAra. All rights reserved.
        </div>
      </div>
    </footer>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import SearchBox from './components/SearchBox.vue';
import HadithCard from './components/HadithCard.vue';
import { searchHadiths } from './services/api';

const searchResults = ref([]);
const totalResults = ref(0);
const searchMode = ref('');
const isLoading = ref(false);
const error = ref(null);
const hasSearched = ref(false);
const lastQuery = ref('');

const onSearch = async ({ query, language }) => {
  if (!query) return;
  
  isLoading.value = true;
  error.value = null;
  hasSearched.value = true;
  lastQuery.value = query;

  try {
    const data = await searchHadiths(query, language);
    searchResults.value = data.sonuclar;
    totalResults.value = data.toplam;
    searchMode.value = data.mod;
    
    // Smooth scroll down to results
    setTimeout(() => {
      window.scrollTo({ top: 400, behavior: 'smooth' });
    }, 100);
  } catch (err) {
    error.value = err.message || 'Sunucuya ulaşılamadı. API bağlatısını kontrol edin.';
  } finally {
    isLoading.value = false;
  }
};
</script>

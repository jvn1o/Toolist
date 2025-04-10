<template>
  <main class="main-content">
    <div class="container mt-4 p-1">
      <!-- 제목 + 필터 -->
      <div
          class="d-flex flex-column flex-md-row justify-content-between align-items-start align-items-md-center gap-2 mb-3 p-2">
        <!-- 제목 -->
        <div class="h2 mb-0">Algorithm & Data Structures</div>

        <!-- 필터 -->
        <div class="d-flex align-items-end">
          <label for="filter" class="me-2 fs-6">Filter by :</label>

          <!-- 필터 드롭다운 -->
          <div class="custom-dropdown me-3">
            <select v-model="selectedFilter">
              <option value="newest">Newest</option>
              <option value="popular">Popular</option>
              <option value="alphabetical">Alphabetical</option>
            </select>
            <span>{{ sortLabel }}</span>
          </div>

          <!-- 가격 필터 드롭다운 -->
          <div class="custom-dropdown">
            <select v-model="selectedPrice">
              <option value="all">Free + Paid</option>
              <option value="free">Free</option>
              <option value="paid">Paid</option>
            </select>
            <span>{{ priceLabel }}</span>
          </div>
        </div>
      </div>


      <!-- 카드 리스트 -->
      <div class="row">
        <div
            v-for="(item, index) in filteredItems"
            :key="index"
            class="col-md-3 col-sm-6 mb-4"
        >
          <div class="card h-100">
            <div class="card-img-wrapper">
              <img :src="item.image" class="card-img-top" :alt="item.title"/>
            </div>
            <div class="card-body d-flex flex-column">
              <h5
                  class="card-title fs-6 text-truncate"
                  :title="item.title"
              >{{ item.title }}</h5>
              <p class="card-text text-muted text-truncate">
                {{ item.category }}
              </p>
              <p class="text-warning" v-if="item.rating">
                <span class="bi bi-bookmark"></span> ({{ item.rating }})
              </p>
              <p
                  class="fw-bold mt-auto"
                  :class="{
            'text-danger': item.price !== 'Free',
            'text-primary': item.price === 'Free',
          }"
              >
                {{ item.price }}
              </p>
              <button class="btn btn-sm">
                <i class="bi bi-bookmark"></i>
              </button>
            </div>
          </div>
        </div>
      </div>


      <!-- 페이지네이션 -->
      <nav>
        <ul class="pagination justify-content-center mt-3">
          <li v-for="page in totalPages" :key="page" class="page-item" :class="{ active: currentPage === page }">
            <button class="page-link" @click="currentPage = page">{{ page }}</button>
          </li>
        </ul>
      </nav>
    </div>
  </main>
</template>

<script setup>
import {ref, computed} from 'vue';

// 필터 상태
const selectedFilter = ref("newest");
const selectedPrice = ref("all");
const currentPage = ref(1);
const itemsPerPage = 8;

// API에서 데이터 불러오기
const {data: items} = await useFetch('/api/resources'); // 백엔드 API 주소

// 정렬 라벨
const sortLabel = computed(() => {
  return selectedFilter.value === "newest" ? "Newest" :
      selectedFilter.value === "popular" ? "Popular" : "Alphabetical";
});

// 가격 라벨
const priceLabel = computed(() => {
  return selectedPrice.value === "all" ? "Free + Paid" :
      selectedPrice.value === "free" ? "Free" : "Paid";
});

// 필터링 및 페이지네이션
const filteredItems = computed(() => {
  if (!items.value) return [];

  const filtered = items.value.filter(item => {
    if (selectedPrice.value === 'free') return item.price === 'Free';
    if (selectedPrice.value === 'paid') return item.price === 'Paid';
    return true;
  });

  return filtered.slice((currentPage.value - 1) * itemsPerPage, currentPage.value * itemsPerPage);
});

const totalPages = computed(() => {
  if (!items.value) return 1;
  const filtered = items.value.filter(item => {
    if (selectedPrice.value === 'free') return item.price === 'Free';
    if (selectedPrice.value === 'paid') return item.price === 'Paid';
    return true;
  });
  return Math.ceil(filtered.length / itemsPerPage);
});
</script>

<style scoped>
.main-content {
  display: flex;
  flex-direction: column;
  min-height: 100vh; /* 전체 화면 기준 최소 높이 확보 */
}

.container {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.card {
  border-radius: 10px;
  overflow: hidden;
}

.card img {
  height: 150px;
  object-fit: cover;
}

/* 공통 스타일 */
.custom-dropdown {
  position: relative;
  display: inline-block;
  cursor: pointer;
  padding-right: 20px;
  font-weight: 500;

}

/* 화살표 스타일 추가 */
.custom-dropdown::before {
  content: "▼";
  position: absolute;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
  font-size: 12px;
  color: #333;
}

/* select 숨기기 */
.custom-dropdown select {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  opacity: 0;
  pointer-events: auto;
}

.row {
  flex-grow: 1; /* 카드 리스트가 남는 공간을 차지하게 함 */
}
</style>
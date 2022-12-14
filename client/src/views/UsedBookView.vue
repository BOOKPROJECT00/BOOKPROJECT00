<template>
  <div class="container">
    <section class="mt-4 d-flex sect-brief col-10 m-auto">
      <div class="col-2 me-auto">
        <img :src="bookData.cover" alt="책이미지예시" />
      </div>
      <div class="col-9">
        <div class="text-muted mb-2">{{ bookData.categoryName }}</div>
        <h3 class="h3">{{ bookData.title }}</h3>
        <div class="mb-2">{{ bookData.author }}</div>
        <div class="mb-4">{{ bookData.publisher }}</div>
        <p class="fs-5 mb-4">
          <span class="text-success">{{
            usedBookData.seller_user_nickname
          }}</span
          >님이 <span class="text-success"> {{ usedBookData.location }}</span
          >에서 판매중이에요
        </p>
        <hr />
        <dl class="d-flex flex-wrap col-6">
          <dt class="col-6 text-center fw-bold">정가</dt>
          <dd class="col-6 text-center">{{ bookData.priceStandard }}원</dd>
          <dt class="col-6 text-center fw-bold">중고판매가</dt>
          <dd class="col-6 text-center">{{ usedBookData.price }}원</dd>
        </dl>
        <hr />
        <table class="table table-striped table-bordered text-center">
          <thead>
            <tr class="">
              <th scope="col" class="col-2"></th>
              <th scope="col" class="col">나쁨</th>
              <th scope="col" class="col">보통</th>
              <th scope="col" class="col">좋음</th>
            </tr>
          </thead>
          <tbody class="">
            <tr class="">
              <th scope="row">책 상태</th>
              <td>{{ usedBookData.total_status === "L" ? "O" : "" }}</td>
              <td>{{ usedBookData.total_status === "M" ? "O" : "" }}</td>
              <td>{{ usedBookData.total_status === "U" ? "O" : "" }}</td>
            </tr>
            <tr class="">
              <th scope="row">찢어짐</th>
              <td>{{ usedBookData.tear_status === "L" ? "O" : "" }}</td>
              <td>{{ usedBookData.tear_status === "M" ? "O" : "" }}</td>
              <td>{{ usedBookData.tear_status === "U" ? "O" : "" }}</td>
            </tr>
            <tr class="">
              <th scope="row">오염 여부</th>
              <td>{{ usedBookData.pollution_status === "L" ? "O" : "" }}</td>
              <td>{{ usedBookData.pollution_status === "M" ? "O" : "" }}</td>
              <td>{{ usedBookData.pollution_status === "U" ? "O" : "" }}</td>
            </tr>
            <tr class="">
              <th scope="row">낙서 여부</th>
              <td>{{ usedBookData.doodle_status === "L" ? "O" : "" }}</td>
              <td>{{ usedBookData.doodle_status === "M" ? "O" : "" }}</td>
              <td>{{ usedBookData.doodle_status === "U" ? "O" : "" }}</td>
            </tr>
          </tbody>
        </table>
        <div class="float-end">
          <button
            class="btn btn-outline-success me-1"
            @click="divertAddOrDelete">
            <i v-show="!isLibraryCart" class="bi bi-bag-check"></i>
            <i v-show="isLibraryCart" class="bi bi-bag-check-fill"></i>
          </button>
          <button class="btn btn-success me-1">구매하기</button>
          <button class="btn btn-success">채팅하기</button>
        </div>
      </div>
    </section>
    <main class="sect-detail col-10 m-auto">
      <h1 class="h4">물품 소개글</h1>
      <hr class="mb-2 mt-0" />
      <div>
        <div class="col-3 me-2 d-inline-block">
          <img :src="usedBookData.image_url_1" alt="책이미지예시" />
        </div>
        <!-- TODO: v-if? -->
        <div class="col-3 me-2 d-inline-block">
          <img src="@/assets/book-image-sample.webp" alt="책이미지예시" />
        </div>
        <div class="col-3 d-inline-block">
          <img src="@/assets/book-image-sample.webp" alt="책이미지예시" />
        </div>
      </div>
      <span class="lh-base">{{ usedBookData.description }}</span>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onUpdated } from "vue";
import axios from "axios";
import { useRoute } from "vue-router";
import { useStore } from "vuex";

const store = useStore();

const usedBookData = ref({
  product_id: 0,
  book_id: "",
  seller_user_id: 0,
  seller_user_nickname: "",
  total_status: "",
  tear_status: "",
  pollution_status: "",
  doodle_status: "",
  price: 0,
  location: "",
  coordination: "",
  sold_out: 0,
  is_online: 0,
  description: "",
  created_at: "",
  image_url_1: "",
  image_url_2: null,
  image_url_3: null,
});

const bookData = ref({
  title: "",
  author: "",
  cover: "",
  categoryName: "",
  publisher: "",
  priceStandard: "",
});

const isLibraryCart = computed(() => {
  return store.state.likeUsedBookList.includes(usedBookData.value.product_id);
});

const divertAddOrDelete = () => {
  if (isLibraryCart.value === false) {
    addLibrary();
  } else {
    deleteLibrary();
  }
};

const addLibrary = async () => {
  store.commit("addLikeUsedBookList", usedBookData.value.product_id);
  // TODO: api로 등록 삭제 if문으로 --면 등록 --면 삭제 셋타임아웃으로 지연시키기
  const result = await axios.post("http://localhost:3000/library/create", {
    param: {
      book_id: usedBookData.value.book_id,
      user_id: store.state.userInfo.user_id,
      product_id: usedBookData.value.product_id,
    },
  });
  alert("찜하기 완료");
};

const deleteLibrary = async () => {
  store.commit("deleteLikeUsedBookList", usedBookData.value.product_id);
  const result = await axios.delete("http://localhost:3000/library/delete", {
    params: {
      book_id: usedBookData.value.book_id,
      user_id: store.state.userInfo.user_id,
      product_id: usedBookData.value.product_id,
    },
  });
  alert("찜하기 취소");
};

const route = useRoute();
const usedBookId = route.query.id;
const getUsedBookData = async () => {
  // TODO: id에 의거한 요청, 에러핸들링, 에외처리
  const result = await axios.get("http://localhost:3000/used-book", {
    params: {
      id: usedBookId,
    },
  });
  usedBookData.value = result.data[0];
  usedBookData.value.seller_user_nickname = result.data.user_nickname;
  getBookData(result.data[0].book_id);
};
const getBookData = async (bookId: string) => {
  const result = await axios.get("http://localhost:3000/book/detail", {
    params: {
      id: bookId,
    },
  });
  const { title, author, cover, categoryName, publisher, priceStandard } =
    result.data.item[0];
  bookData.value.title = title;
  bookData.value.author = author;
  bookData.value.cover = cover;
  bookData.value.categoryName = categoryName;
  bookData.value.publisher = publisher;
  bookData.value.priceStandard = priceStandard;
};
getUsedBookData();
</script>

<style scoped>
img {
  width: 100%;
}
.sect-brief {
  margin-bottom: 6vw;
}

.table-bordered th,
.table-bordered td {
  border: 1px solid #bdbdbd;
}
</style>

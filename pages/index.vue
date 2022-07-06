<template>
  <div class="container">
    <div
      v-if="articles.length && page > 10"
      v-observe-visibility="handleScrolledToTop"
      class="observe-view"
    />

    <h2 class="container__page-info">PAGE: {{ page }}</h2>
    <div v-if="articles.length" class="example">
      <div v-for="item in articles" :key="item._id" class="example-item">
        <span>{{ item._id }}</span>
        <span>{{ item.name }}</span>
        <span>{{ item.trips }}</span>
      </div>
    </div>

    <div v-if="page === lastPage" class="example__last-page">
      This is Last page
    </div>

    <BaseLoader full :visible="isLoading" />

    <div
      v-if="articles.length"
      v-observe-visibility="{
        callback: handleScrolledToBottom,
        throttle: 300,
      }"
      class="observe-view"
    />
  </div>
</template>

<script>
import axios from 'axios'
import BaseLoader from '@/components/BaseLoader'

export default {
  name: 'IndexPage',

  components: {
    BaseLoader,
  },

  data() {
    return {
      articles: [],
      page: 1,
      isLoading: false,
      lastPage: 1,
    }
  },

  watch: {
    isLoading: {
      immediate: true,
      handler(val) {
        const className = 'no-scroll'
        if (val) {
          document.body.classList.add(className)
        } else {
          document.body.classList.remove(className)
        }
      },
    },
  },

  mounted() {
    this.fetch()
  },

  methods: {
    async fetch(topStatus = false) {
      this.isLoading = true

      await axios
        .get(
          `https://api.instantwebtools.net/v1/passenger?page=${this.page}&size=5`
        )
        .then((res) => {
          if (res.status === 200) {
            if (!topStatus) {
              this.articles.push(...res.data.data)
              this.lastPage = res.data.totalPages
            } else {
              // Если колличество страниц привышает 10, все обнуляем!!!
              this.articles = []
              this.page = 1
              this.articles.push(...res.data.data)
            }
          }
        })
        .catch((err) => {
          console.log(err)
        })
        .finally(() => {
          this.isLoading = false
        })
    },

    handleScrolledToBottom(isVisible, entry) {
      // обзервер для верхней позиции скрола!!!
      if (!isVisible) return
      if (this.page >= this.lastPage) return

      console.log(entry)

      this.page++
      this.fetch()
    },

    handleScrolledToTop(isVisible) {
      if (!isVisible) return

      const topStatus = true
      if (this.page > 10) {
        this.fetch(topStatus)
      }
    },
  },

  // На основании обсервера, можно сделать loadZone!!!
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
  background-color: gray;
}

body.no-scroll {
  overflow: hidden;
}

.container {
  width: 900px;
  margin: 0 auto;
  color: white;
  padding: 20px;
  overflow-y: scroll;
  position: relative;
}

.example-item {
  background-color: orange;
  display: flex;
  flex-direction: column;
  padding: 10px;
  border-radius: 10px;
  margin-bottom: 20px;
  box-shadow: rgba(50, 50, 93, 0.25) 0px 30px 60px -12px inset,
    rgba(0, 0, 0, 0.3) 0px 18px 36px -18px inset;
}

.example-item span {
  font-size: 20px;
}

.example__last-page {
  margin-top: 20px;
  text-align: center;
  background-color: red;
  padding: 20px;
  border-radius: 10px;
  box-shadow: rgba(50, 50, 93, 0.25) 0px 30px 60px -12px inset,
    rgba(0, 0, 0, 0.3) 0px 18px 36px -18px inset;
}

.example-item span:not(:last-child) {
  margin-bottom: 20px;
}

.container__page-info {
  position: fixed;
  top: 50%;
  transform: translateY(-50%);
  left: 20px;
}

.observe-view {
  width: 100%;
  height: 5px;
  background-color: red;
}
</style>

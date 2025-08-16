<template>
  <div class="blog-list-container" ref="mainContainer" v-loading="isLoading">
    <ul>
      <li v-for="item in data.rows" :key="item.id">
        <div class="thumb">
          <RouterLink :to="{ name: 'BlogDetail', params: { id: item.id } }">
            <img v-lazy="item.thumb" />
          </RouterLink>
        </div>
        <div class="main">
          <div class="aside">
            <RouterLink :to="{ name: 'BlogDetail', params: { id: item.id } }">
              <h2>{{ item.title }}</h2>
            </RouterLink>
            <span>日期： {{ item.createData }}</span>
            <span>浏览： {{ item.scanNumber }}</span>
            <span>评论： {{ item.commentNumber }}</span>

            <!-- 空值保护 + 统一字符串 id -->
            <RouterLink
              v-if="item.category"
              :to="{
                name: 'CategoryBlog',
                params: { categoryId: item.category.id },
              }"
            >
              <span>{{ item.category.name }}</span>
            </RouterLink>
            <span v-else>未分类</span>
          </div>
          <div class="desc">
            {{ item.description }}
          </div>
        </div>
      </li>
    </ul>

    <Pager
      :total="data.total"
      :limit="routeInfo.limit"
      :current="routeInfo.page"
      :visibleNumber="10"
      @pagechange="handlepagechange"
      style="margin-top: 200px"
    />
    <button @click="handleclick">aads</button>
  </div>
</template>

<script>
import Pager from "@/components/Pager/index.vue";
import fetchData from "@/mixins/fetchData";
import { getBlogs } from "@/api/blog.js";
import { formatDate } from "@/utils";
import mainScroll from "@/mixins/mainScroll.js";

export default {
  mixins: [fetchData({ total: 0, rows: [] }), mainScroll("mainContainer")],
  components: { Pager },
  computed: {
    routeInfo() {
      // 统一用字符串，-1 代表全部
      const categoryId = this.$route.params.categoryId || "-1";
      const page = +this.$route.query.page || 1;
      const limit = +this.$route.query.limit || 10;
      return { categoryId, page, limit };
    },
  },
  watch: {
    async $route() {
      this.isLoading = true;
      this.data = await this.fetchData();
      this.$refs.mainContainer.scrollTop = 0;
      this.isLoading = false;
    },
  },
  methods: {
    handleclick() {
      console.log(this.data);
    },
    formatDate,
    async fetchData() {
      const result = await getBlogs(
        this.routeInfo.page,
        this.routeInfo.limit,
        this.routeInfo.categoryId
      );
      result.rows.forEach(
        (item) => (item.thumb = "http://localhost:7001" + item.thumb)
      );
      return result;
    },
    handlepagechange(newpage) {
      const query = { page: newpage, limit: this.routeInfo.limit };
      if (this.routeInfo.categoryId === "-1") {
        this.$router.push({ name: "Blog", query });
      } else {
        this.$router.push({
          name: "CategoryBlog",
          query,
          params: { categoryId: this.routeInfo.categoryId },
        });
      }
    },
  },
};
</script>

<style scoped lang="less">
@import "~@/styles/var.less";
.blog-list-container {
  line-height: 1.7;
  position: relative;
  padding: 20px;
  overflow-y: scroll;
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  scroll-behavior: smooth;
  ul {
    list-style: none;
    margin: 0;
    padding: 0;
  }
}
li {
  display: flex;
  padding: 15px 0;
  border-bottom: 1px solid @gray;
  .thumb {
    flex: 0 0 auto;
    margin-right: 15px;
    img {
      display: block;
      max-width: 200px;
      border-radius: 5px;
    }
  }
  .main {
    flex: 1 1 auto;
    h2 {
      color: @dark;
      margin: 0;
    }
  }
  .aside {
    font-size: 12px;
    height: auto;
    color: @gray;
    span {
      margin-right: 15px;
    }
  }
  .desc {
    margin: 15px 0;
    font-size: 14px;
  }
}
</style>
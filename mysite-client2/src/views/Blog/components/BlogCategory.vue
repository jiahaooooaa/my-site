<template>
  <div class="blog-category-container">
    <h2>文章分类</h2>
    <RightList :list="list" @select="handleselect" />
  </div>
</template>

<script>
import RightList from "./RightList";
import fetchData from "@/mixins/fetchData.js";
import { getBlogCategories } from "@/api/blog.js";

export default {
  mixins: [fetchData([])],
  components: { RightList },
  computed: {
    limit() {
      return +this.$route.query.limit || 10;
    },
    // 统一返回字符串 id
    activeCategoryId() {
      const id = this.$route.params.categoryId;
      return id && id !== "-1" ? String(id) : "-1";
    },
    list() {
      const totalArticleCount = this.data.reduce(
        (a, b) => a + b.articleCount,
        0
      );

      const result = [
        { name: "全部", id: "-1", articleCount: totalArticleCount },
        ...this.data,
      ];

      return result.map((it) => ({
        ...it,
        isselect: it.id === this.activeCategoryId,
        aside: `${it.articleCount}篇`,
      }));
    },
  },
  methods: {
    async fetchData() {
      return await getBlogCategories();
    },
    handleselect(item) {
      const query = { page: 1, limit: this.limit };
      if (item.id === "-1") {
        this.$router.push({ name: "Blog", query });
      } else {
        this.$router.push({
          name: "CategoryBlog",
          query,
          params: { categoryId: String(item.id) },
        });
      }
    },
  },
};
</script>

<style scoped lang="less">
.blog-category-container {
  width: 300px;
  box-sizing: border-box;
  padding: 20px;
  position: relative;
  height: 100%;
  overflow-y: auto;

  h2 {
    font-weight: bold;
    letter-spacing: 2px;
    font-size: 1em;
    margin: 0;
  }
}
</style>
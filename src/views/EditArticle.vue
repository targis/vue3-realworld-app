<template>
  <div>
    <mv-loading v-if="isLoading" />
    <mv-article-form
      v-if="initialValues"
      :initial-values="initialValues"
      :errors="validationErrors"
      :is-submitting="isSubmitting"
      @articleSubmit="onSubmit"
    />
  </div>
</template>

<script>
import {mapState} from 'vuex'
import {actionTypes} from '@/store/modules/editArticle'
import MvArticleForm from '@/components/ArticleForm'
import MvLoading from '@/components/Loading'
export default {
  name: 'MvEditArticle',
  components: {
    MvArticleForm,
    MvLoading
  },
  computed: {
    ...mapState({
      isSubmitting: state => state.editArticle.isSubmitting,
      isLoading: state => state.editArticle.isLoading,
      article: state => state.editArticle.article,
      validationErrors: state => state.editArticle.validationErrors
    }),
    initialValues() {
      if (!this.article) {
        return null
      }
      return {
        title: this.article.title,
        description: this.article.description,
        body: this.article.body,
        tagList: this.article.tagList
      }
    }
  },
  mounted() {
    this.$store.dispatch(actionTypes.getArticle, {
      slug: this.$route.params.slug
    })
  },
  methods: {
    onSubmit(articleInput) {
      this.$store
        .dispatch(actionTypes.updateArticle, {
          slug: this.$route.params.slug,
          articleInput
        })
        .then(article => {
          this.$router.push({name: 'article', params: {slug: article.slug}})
        })
    }
  }
}
</script>

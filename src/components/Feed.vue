<template>
  <div>
    <mv-loading v-if="isLoading" />

    <mv-error-message v-if="error" />

    <div v-if="feed">
      {{ feed.articles.length === 0 ? 'No articles are here... yet.' : '' }}
      <div
        class="article-preview"
        v-for="(article, index) in feed.articles"
        :key="index"
      >
        <div class="article-meta">
          <router-link
            :to="{name: 'userProfile', params: {slug: article.author.username}}"
          >
            <img :src="article.author.image" alt="avatar" />
          </router-link>
          <div class="info">
            <router-link
              :to="{
                name: 'userProfile',
                params: {slug: article.author.username}
              }"
              class="author"
            >
              {{ article.author.username }}
            </router-link>
            <span class="date">{{ article.createdAt }}</span>
          </div>
          <div class="pull-xs-right">
            <mv-add-to-favorites
              :is-favorited="article.favorited"
              :article-slug="article.slug"
              :favorites-count="article.favoritesCount"
            />
          </div>
        </div>
        <router-link
          :to="{name: 'article', params: {slug: article.slug}}"
          class="preview-link"
        >
          <h1>{{ article.title }}</h1>
          <p>{{ article.description }}</p>
          <span>Read more...</span>
          <mv-tag-list :tags="article.tagList" />
        </router-link>
      </div>
      <mv-pagination
        :total="feed.articlesCount"
        :limit="limit"
        :current-page="currentPage"
        :url="baseUrl"
      />
    </div>
  </div>
</template>

<script>
import {mapState} from 'vuex'
import {actionTypes} from '@/store/modules/feed'
import MvPagination from '@/components/Pagination'
import MvTagList from '@/components/TagList'
import MvAddToFavorites from '@/components/AddToFavorites'
import MvLoading from '@/components/Loading'
import MvErrorMessage from '@/components/ErrorMessage'
import {limit} from '@/helpers/vars'
import {stringify, parseUrl} from 'query-string'
export default {
  name: 'MvFeed',
  props: {
    apiUrl: {
      type: String,
      required: true
    }
  },
  components: {
    MvPagination,
    MvLoading,
    MvErrorMessage,
    MvTagList,
    MvAddToFavorites
  },
  data() {
    return {
      limit,
      url: '/'
    }
  },
  computed: {
    ...mapState({
      isLoading: state => state.feed.isLoading,
      error: state => state.feed.error,
      feed: state => state.feed.data
    }),
    currentPage() {
      return Number(this.$route.query.page || '1')
    },
    baseUrl() {
      return this.$route.path
    },
    offset() {
      return this.currentPage * limit - limit
    }
  },
  watch: {
    currentPage() {
      this.fetchFeed()
    },
    apiUrl() {
      this.fetchFeed()
    }
  },
  mounted() {
    this.fetchFeed()
  },
  methods: {
    fetchFeed() {
      const parsedUrl = parseUrl(this.apiUrl)
      const stringifiedParams = stringify({
        limit,
        offset: this.offset,
        ...parsedUrl.query
      })
      const apiUrlWithParams = `${parsedUrl.url}?${stringifiedParams}`
      console.log(apiUrlWithParams)
      this.$store.dispatch(actionTypes.getFeed, {apiUrl: apiUrlWithParams})
    }
  }
}
</script>

<template>
  <Layout>
    <div class="content-wrapper writings">
      <navigation />

      <main id="main-content">
        <heading
          :breadcrumb="true"
          titletag="h1"
          :title="$page.Prismic.youtube.title"
          :uppercase="true"
        />

        <div class="filters">
          <button
            v-for="tag in tags"
            :key="tag.tag"
            class="filter"
            :class="{ selected: tag.selected }"
            @click="tagClick(tag)"
          >
            {{ tag.tag }}
          </button>
        </div>
        <div class="posts videos">
          <article
            v-for="video in filteredVideos"
            :key="video.slug"
            class="post video"
          >
            <g-link :to="`/videos/${video.slug}/`">
              <lazy-image
                ratio="16/9"
                :alt="$prismic.asText(video.title)"
                :url="video.image.url"
                :caption="false"
                :widths="[300, 400, 500, 600, 680]"
                sizes="(max-width: 400px) 100vw, (min-width: 700px) 210px"
              />
            </g-link>

            <div class="post-content-wrap">
              <p class="post-title">
                <g-link :to="`/videos/${video.slug}/`">
                  {{ $prismic.asText(video.title) }}
                </g-link>
              </p>
              <p class="post-tags">
                <span v-for="(tag, index) in video.tags" :key="tag">
                  <span class="tag">{{ tag }}</span>
                  <template v-if="index + 1 < video.tags.length">, </template>
                </span>
              </p>

              <p class="post-date fancy-title yellow">
                {{ $prismic.asDay(video.date) }}
                {{ $prismic.asMonth(video.date) }}
                {{ $prismic.asYear(video.date) }}
              </p>
            </div>
          </article>
        </div>
      </main>
    </div>
  </Layout>
</template>

<page-query>
query {
  Prismic {
    allVideos(sortBy:publication_date_DESC) {
      edges {
        node {
          _meta {
            uid
            tags
          }
          title
          image
          publication_date
        }
      }
    },
    youtube(uid: "youtube", lang: "en-us") {
      title
      body {
        ... on Prismic_YoutubeBodyGeneral_card {
          type
          primary {
            title
            description
            image
          }
        }
        ... on Prismic_YoutubeBodyTwitter_card {
          type
          primary {
            twitter_handle
            title
            description
            image
          }
        }
        __typename
      }
    }
  }
}
</page-query>

<script>
import Navigation from '../components/navigation.vue'
import Heading from '../components/heading.vue'
import LazyImage from '../components/lazy-image.vue'
import mapMetaInfo from '../prismic/mapMetaInfo'
import { mapGetters, mapActions } from 'vuex'

export default {
  components: {
    Navigation,
    Heading,
    LazyImage,
  },

  metaInfo() {
    return mapMetaInfo(
      this.$page.Prismic.youtube,
      'videos',
      this.$router.currentRoute
    )
  },

  computed: mapGetters(['filteredVideos', 'tags']),

  mounted() {
    this.setInitalVideos(this.$page.Prismic.allVideos.edges)
  },

  methods: {
    tagClick(tag) {
      this.filter(tag)
    },

    ...mapActions(['setInitalVideos', 'setInitalTags', 'filter']),
  },
}
</script>

<style lang="scss" scoped>
.videos {
  display: grid;
  grid-gap: 1rem;
  min-width: 0;

  // prettier-ignore
  @include responsive('grid-template-columns', (xs: 100%, sm: repeat(2, 47.5%), m: repeat(3, 32%)));

  .video {
    width: auto;
    margin: 0;
    flex-direction: column;

    figure {
      float: none;
      width: 100%;
      height: auto;
      margin: rem(0 0 16px 0);
    }

    .post-tags {
      font-weight: 400;
      color: $blue-light;
      font-size: 0.8rem;
      text-transform: uppercase;
      margin: rem(0 0 10px 0);
    }

    .post-date {
      float: none;
      text-align: left;
      margin: 0;
      font-size: rem(13px);
    }

    .post-title {
      font-size: 1rem;
      line-height: 1.3;
    }
  }
}
</style>

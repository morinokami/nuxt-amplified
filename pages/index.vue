<template>
  <div class="container">
    <main class="main">
      <h1 class="title">Amplify + Nuxt</h1>

      <p class="description">
        <code class="code">{{ posts.length }}</code>
        posts
      </p>

      <div class="grid">
        <a
          v-for="post in posts"
          :key="post.id"
          class="card"
          :href="`/posts/${post.id}`"
        >
          <h3>{{ post.title }}</h3>
          <p>{{ post.content }}</p>
        </a>

        <div class="card">
          <h3 class="title">New Post</h3>

          <amplify-authenticator>
            <form @submit.prevent="handleCreatePost">
              <fieldset>
                <legend>Title</legend>
                <input
                  :value="`Today, ${new Date().toLocaleTimeString()}`"
                  name="title"
                />
              </fieldset>

              <fieldset>
                <legend>Content</legend>
                <textarea name="content">
I built an Amplify app with Nuxt!</textarea
                >
              </fieldset>

              <button>Create Post</button>
              <button type="button" @click="signOut">Sign out</button>
            </form>
          </amplify-authenticator>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import {
  defineComponent,
  useAsync,
  useContext,
  useMeta,
} from '@nuxtjs/composition-api'
import { API, Auth, withSSRContext } from 'aws-amplify'
import { createPost } from '@/graphql/mutations'
import { listPosts } from '@/graphql/queries'

export default defineComponent({
  setup() {
    useMeta({ title: 'Amplify + Nuxt' })

    const { req } = useContext()
    const SSR = withSSRContext({ req })
    const posts = useAsync(async () => {
      const response = await SSR.API.graphql({ query: listPosts })
      return response.data.listPosts.items
    })

    const handleCreatePost = async (event) => {
      const form = new FormData(event.target)

      try {
        const { data } = await API.graphql({
          authMode: 'AMAZON_COGNITO_USER_POOLS',
          query: createPost,
          variables: {
            input: {
              title: form.get('title'),
              content: form.get('content'),
            },
          },
        })

        window.location.href = `/posts/${data?.createPost.id}`
      } catch ({ errors }) {
        // eslint-disable-next-line no-console
        console.error(...errors)
        throw new Error(errors[0].message)
      }
    }

    const signOut = () => Auth.signOut()

    return {
      posts,
      handleCreatePost,
      signOut,
    }
  },
  head: {},
})
</script>

<style scoped>
.container {
  min-height: 100vh;
  padding: 0 0.5rem;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.main {
  padding: 5rem 0;
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.title a {
  color: #0070f3;
  text-decoration: none;
}

.title a:hover,
.title a:focus,
.title a:active {
  text-decoration: underline;
}

.title {
  margin: 0;
  line-height: 1.15;
  font-size: 4rem;
}

.title,
.description {
  text-align: center;
}

.description {
  line-height: 1.5;
  font-size: 1.5rem;
}

.code {
  background: #fafafa;
  border-radius: 5px;
  padding: 0.75rem;
  font-size: 1.1rem;
  font-family: Menlo, Monaco, Lucida Console, Liberation Mono, DejaVu Sans Mono,
    Bitstream Vera Sans Mono, Courier New, monospace;
}

.grid {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-wrap: wrap;
  max-width: 800px;
  margin-top: 3rem;
}

.card {
  margin: 1rem;
  flex-basis: 45%;
  padding: 1.5rem;
  text-align: left;
  color: inherit;
  text-decoration: none;
  border: 1px solid #eaeaea;
  border-radius: 10px;
  transition: color 0.15s ease, border-color 0.15s ease;
}

.card:hover,
.card:focus,
.card:active {
  color: #0070f3;
  border-color: #0070f3;
}

.card h3 {
  margin: 0 0 1rem 0;
  font-size: 1.5rem;
}

.card p {
  margin: 0;
  font-size: 1.25rem;
  line-height: 1.5;
}

@media (max-width: 600px) {
  .grid {
    width: 100%;
    flex-direction: column;
  }
}
</style>

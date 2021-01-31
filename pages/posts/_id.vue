<template>
  <div class="container">
    <main class="main">
      <h1 class="title">{{ post.title }}</h1>

      <p class="description">{{ post.content }}</p>
    </main>

    <footer class="footer">
      <button @click="handleDelete">💥 Delete post</button>
    </footer>
  </div>
</template>

<script>
import {
  defineComponent,
  useAsync,
  useContext,
  useMeta,
} from '@nuxtjs/composition-api'
import { API, withSSRContext } from 'aws-amplify'
import { deletePost } from '@/graphql/mutations'
import { getPost } from '@/graphql/queries'

export default defineComponent({
  setup() {
    const { route } = useContext()
    const SSR = withSSRContext()
    const post = useAsync(async () => {
      const response = await SSR.API.graphql({
        query: getPost,
        variables: {
          id: route.value.params.id,
        },
      })
      return response.data.getPost
    })

    useMeta(() => ({ title: `${post.value.title} - Amplify + Nuxt` }))

    const handleDelete = async () => {
      try {
        await API.graphql({
          authMode: 'AMAZON_COGNITO_USER_POOLS',
          query: deletePost,
          variables: {
            input: { id: post.value.id },
          },
        })

        window.location.href = '/'
      } catch ({ errors }) {
        // eslint-disable-next-line no-console
        console.error(...errors)
        throw new Error(errors[0].message)
      }
    }

    return { post, handleDelete }
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

.footer {
  width: 100%;
  height: 100px;
  border-top: 1px solid #eaeaea;
  display: flex;
  justify-content: center;
  align-items: center;
}

.footer img {
  margin-left: 0.5rem;
}

.footer a {
  display: flex;
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
</style>

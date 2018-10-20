<template>
  <div class="my-app">
    <v-app>
      <v-toolbar>
        <v-toolbar-title>Book SPA</v-toolbar-title>
      </v-toolbar>
      <v-content>
        <h1>{{ appName }}</h1>

        <input v-model="keyword" placeholder="keyword">
        <button id='book-search' @click='bookSearch'>Search</button>
        <p>keyword is {{ keyword }}</p>

        <v-container fuild>
          <v-layout align-center justify-space-around row wrap>
            <v-flex xs12>
              <!-- Card layouts -->
              <v-card v-for="book in books" :key="book.isbn13" hover>
                <v-layout row wrap my-2 align-center justify-center>
                  <v-flex xs3>
                    <v-img
                      :src='book.imageLink'
                      alt='Thumbnail Image.'
                      height='128px'
                      contain
                      >
                    </v-img>
                  </v-flex>
                  <v-flex xs9>
                    <v-card-title primary-title>{{ book.title }}</v-card-title>
                    <v-card-text>{{ book.shortDescription }}</v-card-text>
                    <v-card-action>
                      <v-btn
                        class='book-rental'
                        name='isbn13'
                        :value='book.isbn13'
                        @click='onRentalClicked'>
                        Rental
                      </v-btn>
                    </v-card-action>
                  </v-flex>
                </v-layout>
              </v-card>
              <!-- End layout card -->
            </v-flex>
          </v-layout>
        </v-container>
      </v-content>
      <v-footer>Developers: #ext-serverless-spa</v-footer>
    </v-app>
  </div>
</template>

<script>
import * as axios from 'axios'
import * as _ from 'lodash'

let SAMPLE_BOOKS = [
  {
    id: '9af55a0a-ab3a-462e-b7e2-05cb521018b9',
    title: '数学ガール (数学ガールシリーズ 1)',
    author: '結城 浩',
    isbn13: '978-4797341379',
    icon_url: ''
  },
  {
    id: '614cef29-0344-4d41-90bf-7aa5cbb6b367',
    title: '数学ガールの秘密ノート/式とグラフ',
    author: '結城 浩',
    isbn13: '978-4797374148',
    icon_url: ''
  }
]

export default {
  name: 'MyApp',
  data () {
    return {
      appName: 'Welcome to Boo Koo',
      keyword: '',
      books: [],
      sample_books: SAMPLE_BOOKS
    }
  },
  computerd: {
  },
  methods: {
    bookSearch: function () {
      /*
        TODO:
        - [ ] 空文字で検索した場合、不正なクエリになりレスポンスが400になる
      */
      console.log('bookSearch called')
      var self = this
      axios.get('https://www.googleapis.com/books/v1/volumes', {
        params: {
          q: this.keyword
        }
      }).then(result => {
        /*
          TODO:
          - [ ] 検索の一致が0件だった場合のハンドリングが必要。今はforeachで例外が出る
        */
        console.log(result)
        // self.books.splice(0, self.books.length, ...books)
        if (result.status === 200) {
          var books = []
          result.data.items.forEach(item => {
            books.push({
              title: _.get(item, 'volumeInfo.title', 'No title'),
              author: _.get(item, 'volumeInfo.authors', ['No author']).join('/'),
              categories: _.get(item, 'volumeInfo.categories', []),
              shortDescription: _.get(item, 'searchInfo.textSnippet', 'No description'),
              description: _.get(item, 'volumeInfo.description', 'No descroption'),
              imageLink: _.get(item, 'volumeInfo.imageLinks.smallThumbnail', undefined),
              isbn13: _.get(item, 'id') // volumeInfo.industryIdentifiers
            })
          })

          console.log('books!')
          console.log(books)
          self.books.splice(0, self.books.length, ...books)
        }
      }).catch(error => {
        alert('error: bookSearch\n' + error)
      })
    },
    onRentalClicked: function (event) {
      console.log('onRentalClicked')
      console.log(event.srcElement.value)
      console.log(event.value)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

.book-info {
  text-align: left;
}

.book-detail {
  text-align: left;
}

h3.book-title {

}

h4.book-author {

}

p.book-description {

}

button.book-rental {

}
</style>

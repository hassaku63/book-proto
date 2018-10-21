<template>
  <div class="my-app">
    <v-app>
      <v-toolbar>
        <v-toolbar-title>Welcome to Book SPA</v-toolbar-title>
      </v-toolbar>
      <v-content>
        <input v-model="keyword" placeholder="keyword">
        <button id='book-search' @click='bookSearch'>Search</button>
        <p>keyword is {{ keyword }}</p>

        <v-container fuild>
          <!-- Displaying search result -->
          <v-layout align-center justify-space-around row wrap>
            <v-flex xs12>
              <!-- Card layouts -->
              <v-card v-for="book in books" :key="book.isbn13" hover>
                <v-layout row wrap my-2 align-center justify-center>
                  <v-flex xs3>
                    <v-img
                      :src='book.imageLink'
                      alt='Thumbnail Image.'
                      height='144px'
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
          <!-- End displaying search result -->
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
    icon_url: '',
    description: '心ときめく数学の世界をあなたに。 『プログラマの数学』の結城浩が贈る『数学ガール』は魅惑の数学物語。美少女ミルカさん、元気少女テトラちゃん、それに僕の三人の高校生が数学にチャレンジ。数学を楽しみ、学ぶことについて考え、異性へほのかな思いに心を動かす......。オイラー生誕300年記念出版。 『数学ガール』は、 ミルカさん+テトラちゃん+「僕」という三人の高校生が、 数学にチャレンジする楽しい《数学・青春・物語》です。 読み物形式でありながら、取り扱う数学的内容は本格的。 数学クイズが好きな一般の方から、理系の大学生、社会人まで楽しめるはずです。 ■目次 ●あなたへ ●プロローグ ●第1章数列とパターン ●第2章数式という名のラブレター ●第3章 ωのワルツ ●第4章フィボナッチ数列と母関数 ●第5章相加相乗平均の関係 ●第6章ミルカさんの隣で ●第7章コンボリューション ●第8章ハーモニック・ナンバー ●第9章テイラー展開とバーゼル問題 ●第10章分割数 ●エピローグ ●あとがき ●参考文献と読書案内 ●索引 ■本書の想定読者 本書は、以下のような方にお勧めできる本です。 ・学校とは違う視点で数学を見てみたいと思う高校生・大学生... ・自分から進んで学ぶってどういうことだろうと思っている人 ・数学に対して《わかっていない感じ》がする人 ・やさしい数学読み物ではちょっと物足りないなあと思っている人 ・数学のおもしろさに触れてみたいと思っている人 ・学生時代は数学をやったけれど、最近はあまり数学的なことを考えてないなと思った社会人 ・《理系にとって最強の萌え》っていったい何だろうと思ったすべての人 (C)ソフトバンククリエイティブ株式会社/SOFTBANK Creative Corp.'
  },
  {
    id: '614cef29-0344-4d41-90bf-7aa5cbb6b367',
    title: '数学ガールの秘密ノート/式とグラフ',
    author: '結城 浩',
    isbn13: '978-4797374148',
    icon_url: '',
    description: '『数学ガール』の新シリーズ誕生！ ※この電子書籍は固定レイアウト型で配信されております。固定レイアウト型は文字だけを拡大することや、文字列のハイライト、検索、辞書の参照、引用などの機能が使用できません。 「僕」と三人の少女が、中学・高校レベルの数学をやさしく解き明かす。新たな発見に出会う新シリーズ、第一弾。 新シリーズ『数学ガールの秘密ノート』の第一作です。 「僕」と三人の少女（ミルカさん、テトラちゃん、ユーリ）が、家と学校で、楽しい数学トークを繰り広げます。中学・高校レベルの数学が中心ですが、 やさしい数学の中にも思いがけない発見があります。各章の最後に掲載した「問題」を解くことで、読者は自分の理解を確認し、実力を高めることができます。 この『数学ガールの秘密ノート』シリーズで数学の基礎を固め、『数学ガール』シリーズにチャレンジしましょう。 ●登場人物紹介 「僕」 高校二年生、語り手。 数学、特に数式が好き。 ユーリ 中学二年生、「僕」の従妹。 栗色のポニーテール。論理的な思考が好き。 テトラちゃん 高校一年生、いつも張り切っている《元気少女》。 ショートカットで、大きな目がチャームポイント。 ミルカさん 高校二年生、数学が得意な《饒舌才媛》。 長い黒髪にメタルフレームの眼鏡。 母 「僕」の母親。 瑞谷女史 「僕」の高校に勤務する司書の先生。'
  }
]

export default {
  name: 'MyApp',
  data () {
    return {
      appName: 'Welcome to Boo Koo',
      keyword: '',
      books: [],
      sample_books: SAMPLE_BOOKS,
      dialog: false
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

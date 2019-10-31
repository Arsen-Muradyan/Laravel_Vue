<template>
  <div>
    <h1>Articles</h1>
    <br>
    <form @submit.prevent="saveArticle">
      <label for="title">Title:</label>
      <input name="title" type="text" class="form-control" v-model="article.title">
      <br>
      <label for="body">Body:</label>
      <textarea v-model="article.body" class="form-control" name="body"></textarea>
      <br>
      <input type="submit" class="btn btn-default w-100" value="Save">
    </form>
    <br>
    <nav aria-label="Page navigation example">
      <ul class="pagination">
        <li class="page-item" v-bind:class="[{ disabled: !this.pagination.prev_page_url }]" @click="fetchArticles(pagination.prev_page_url)"><a class="page-link" href="#">Previous</a></li>
        <li class="page-item disabled">
          <a href="#" class="page-link">{{ this.pagination.current_page }} of {{ this.pagination.last_page }}</a>
        </li>
        <li class="page-item"  v-bind:class="[{ disabled: !this.pagination.next_page_url }]"><a class="page-link" href="#" @click="fetchArticles(pagination.next_page_url)" v-bind:class="[{ disabled: !this.pagination.next_page_url }]">Next</a></li>
      </ul>
    </nav>
    <div v-for="article in articles" v-bind:key="article.id" class="card card-body">
      <h4>{{ article.title }}</h4>
      <p>{{ article.body }}</p>
      <a class="btn btn-warning" href="#" @click="editArticle(article)">Edit</a>
      <form @submit.prevent="deleteArticle(article.id)">
        <br>
        <input type="submit" class="btn btn-danger w-100" value="DELETE">
      </form>
    </div>
  </div>  
</template>
<script>
export default {
  data() {
    return {
      articles: [],
      pagination: {},
      article: {
        title: '',
        body: ''
      },
      article_id: ''
    }
  },
  created() {
    this.fetchArticles();
  },
  methods: {
    editArticle(article) {
      this.article_id = article.id,
      this.article.title = article.title,
      this.article.body = article.body
    },
    fetchArticles(page_url) {
      var page_url = page_url || '/api/articles';
      fetch(page_url)
        .then(res => res.json())
        .then(res => {
          this.articles = res.data;
          this.makePagination(res.meta, res.links);
        })
    },
    makePagination(meta, links) {
      let pagination = {
        next_page_url: links.next,
        prev_page_url: links.prev,
        current_page: meta.current_page,
        last_page: meta.last_page
      }
      this.pagination = pagination
    },
    saveArticle() {
      if (this.article_id == '') {
        fetch('/api/articles', {
          method: "POST",
          body: JSON.stringify(this.article),
          headers: {
            'content-type':'application/json'
          }
        })
          .then(res => res.json())
          .then(res => {
            if (res) {
              this.fetchArticles();
              this.article.title = '';
              this.article.body = ''
            }
          })
      } else {
        fetch(`/api/articles/${this.article_id}`, {
          method: "PUT",
          body: JSON.stringify(this.article),
          headers: {
            'content-type':'application/json'
          }
        })
          .then(res => res.json())
          .then(res => {
            if (res) {
              this.fetchArticles();
              this.article.title = '';
              this.article.body = ''
            }
          })
      }
    },
    deleteArticle(id) {
      var quest = confirm("Are You Sure");
      if (quest) {
         fetch(`/api/articles/${id}`, {
          method: "delete"
          })
          .then(res => res.json())
          .then(res => {
            if (res) {
              alert("Article Removed")
            }
            this.fetchArticles();
          })
      }
      return false;
    }
  }
}
</script>
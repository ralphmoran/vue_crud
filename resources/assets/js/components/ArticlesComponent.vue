<template>
    <div>
        <h2>Articles - CRUD actions with VueJS Components</h2>
        <form class="mb-3" @submit.prevent="addArticle()">
            <div class="form-group">
                <input type="text" class="form-control" placeholder="Title" v-model="article.title">
            </div>
            <div class="form-group">
                <textarea class="form-control" rows="5" v-model="article.body" placeholder="Body"></textarea>
            </div>
            <button type="submit" class="btn btn-ligth btn-block">Save</button>
        </form>
        <nav aria-label="Page navigation example">
        <ul class="pagination">
            <li :class="[{disabled: !pagination.prev}]" class="page-item">
                <a class="page-link" href="#" @click="fetchArticles(pagination.prev)">
                    Previous
                </a>
            </li>

            <li class="page-item active">
                <a class="page-link" href="#">{{pagination.current_page}} of {{pagination.last_page}} <span class="sr-only">(current)</span></a>
            </li>

            <li :class="[{disabled: !pagination.next}]" class="page-item">
                <a class="page-link" href="#" @click="fetchArticles(pagination.next)">Next</a>
            </li>
        </ul>
        </nav>

        <div class="card card-body mb-2" v-for="article in articles" :key="article.id">
            <h3>{{article.title}}</h3>
            <p>{{article.body}}</p>
            <hr>
            <button @click="editArticle(article)" class="btn btn-warning mb-2">Edit</button>
            <button @click="deleteArticle(article.id)" class="btn btn-danger">Delete</button>
        </div>

    </div>
</template>

<script>
export default {
    data(){
        return {
            articles: [],
            article: {
                id: '',
                title: '',
                body: ''
            },
            article_id: '',
            pagination: {},
            edit: false
        }
    },
    created(){
        this.fetchArticles();
    },
    methods: {
        fetchArticles(page){
            
            let vm = this;
            page = page || 'api/articles';

            // Makes an AJAX call to DB
            fetch(page)
                .then(res => res.json())
                .then(res => {
                    this.articles = res.data; //<-- Changing articles state. This line updates our articles list
                    vm.makePagination( res.meta, res.links );
                })
                .catch(e => console.log(e));
        },
        makePagination(meta, links){
            this.pagination = {
                next: links.next,
                prev: links.prev,
                current_page: meta.current_page,
                last_page: meta.last_page
            }
        },
        deleteArticle(id){
            if(confirm("Are you sure?...")){
                fetch(`api/article/${id}`, {
                    method: 'delete'
                })
                .then( res => res.json() )
                .then( res => {
                    this.fetchArticles();
                })
                .catch( e => consol.log(e) );
            }
        },
        addArticle(){
            if(this.edit === false){
                // Add
                fetch('api/article', {
                    method: 'post',
                    body: JSON.stringify(this.article),
                    headers: {
                        'content-type': 'application/json'
                    }
                })
                .then( res => res.json() )
                .then( res => {
                    this.article.title = '';
                    this.article.body = '';
                    this.fetchArticles();
                })
                .catch( e => console.log(e) );
            }else{
                // Update
                fetch('api/article', {
                    method: 'put',
                    body: JSON.stringify(this.article),
                    headers: {
                        'content-type': 'application/json'
                    }
                })
                .then( res => res.json() )
                .then( res => {
                    this.article.title = '';
                    this.article.body = '';
                    this.fetchArticles();
                })
                .catch( e => console.log(e) );
                
            }
        },
        editArticle(article){
            let art = this.article;

            art.title      = article.title  ;
            art.body       = article.body;
            art.article_id = article.id  ;
            art.id         = article.id  ;

            this.edit = true;
        }
    }
}
</script>

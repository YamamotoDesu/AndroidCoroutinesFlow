# AndroidCoroutinesFlow
<img width="300" src="https://user-images.githubusercontent.com/47273077/188294001-9d14d5e1-40fa-4982-a88e-ce1cb404b1be.gif">

NewsRepository
```kt
class NewsRepository {
    companion object {
        private const val BASE_URL = "https://raw.githubusercontent.com/DevTides/NewsApi/master/"
        private const val NEWS_DELAY = 3000L
    }

    private val newsService = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create())
        .build()
        .create(NewsService::class.java)

    fun getNewsArticles(): Flow<NewsArticle> {
        return flow {
            var newsSource = newsService.getNews()
            newsSource.forEach {
                emit(it)
                delay(NEWS_DELAY)
            }
        }
    }

}
```

ListViewModel
```kt
class ListViewModel: ViewModel() {

    val newsArticles = NewsRepository().getNewsArticles().asLiveData()

}
```

MainActivity
```kt
class MainActivity : AppCompatActivity() {

    lateinit var viewModel: ListViewModel
    private val newsListAdapter = NewsListAdapter()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        viewModel = ViewModelProviders.of(this).get(ListViewModel::class.java)

        newsList.apply {
            layoutManager = LinearLayoutManager(this@MainActivity)
            adapter = newsListAdapter
        }

        observeViewModel()
    }

    private fun observeViewModel() {
        viewModel.newsArticles.observe(this, Observer { article ->
            loading_view.visibility = View.GONE
            newsList.visibility = View.VISIBLE
            newsListAdapter.onAddNewsItem(article)
            newsList.smoothScrollToPosition(0)
        })
    }
}
```

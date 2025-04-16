
# 🪙 Android Crypto Monitor - Checkpoint 2

_Lucas de Lima Bezerra - RM98632_

APP de monitoração de cryptos utilizando API Rest do MercadoBitcoin

Jump to: [Documentação de classes](#-documentacao)

## 📲 Testando

<img src="images/Screen_recording.gif" alt="Screenshot do APP" width="200">

## 🛠️ Como você pode testar

- Clone o repositório

```
git clone https://github.com/lucaslimb/Android-crypto-monitor.git
```

- Utilize um emulador via Android Studio ou IDE própria
- Garanta que o Android emulado seja 8.1 (API 27) ou superior 

## 💻 Stack utilizada

- Kotlin
- Android Studio
- Gradle
- Destaque de dependências:
  - Retrofit2
  - Retrofit2 Gson Converter
  - Kotlinx Coroutines
- APIs:
  - [MercadoBitcoin](https://api.mercadobitcoin.net/api/v4/docs)

## 📚 Estrutura do projeto (simplificada)

```
├───app.src.main
│    └───java
│         └───lucaslimb.com.github.android_crypto_monitor
│                ├───model           
│                ├───service    
│                ├───ui        
│                └───MainActivity.kt                             
└───res
    ├───drawable
    ├───layout
    ├───values
    └───xml
```

## 📖 Documentação

### MainActivity
Path: `app/src/main/java/lucaslimb/com/github/android_crypto_monitor/MainActivity.kt`

Código executado ao inciar a activity (onCreate)
  - Faz a conexão entre o layout em xml e o código Kotlin
  - Busca a ToolBar no layout e executa a função para configura-la
  - Busca o botão atualizar no layout e adiciona um Listener de cilick com a função para a chamada da API
```kotlin
  override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val toolbarMain: Toolbar = findViewById(R.id.toolbar_main)
        configureToolbar(toolbarMain)

        val btnRefresh: Button = findViewById(R.id.btn_refresh)
        btnRefresh.setOnClickListener{
            makeRestCall()
        }
    }
```

Função configureToolbar()
  - Define a barra de tarefas, assim como sua cor de fundo, título e cor da fonte
```kotlin
private fun configureToolbar(toolbar: Toolbar) {
        setSupportActionBar(toolbar)
        toolbar.setTitleTextColor(getColor(R.color.white))
        supportActionBar?.setTitle(getText(R.string.app_title))
        supportActionBar?.setBackgroundDrawable(getDrawable(R.color.success))
    }
```

Função makeRestCall() em detalhe
  - Inicia a coroutine para requisição assíncrona
```kotlin
  CoroutineScope(Dispatchers.Main).launch {
```
  - Cria a service faz a requisição para os dados do ticker
```kotlin
val service = MercadoBitcoinServiceFactory().create()
val response = service.getTicker()
```
  - Se a requisição foi bem-sucecida, recupera o body de resposta
  - Busca as label de valor e data no layout e define lastValue com o atributo last no ticker
  - Se last não for null, formata para real e insere na label de valor
  - Formata a data para o formato padrão nacional e insere na label de data
```kotlin
if(response.isSuccessful){
    val tickerResponse = response.body()
    val lblValue: TextView = findViewById(R.id.lbl_value)
    val lblDate: TextView = findViewById(R.id.lbl_date)
    val lastValue = tickerResponse?.ticker?.last?.toDoubleOrNull()
  
    if(lastValue != null) {
        val numberFormat = NumberFormat.getCurrencyInstance(Locale("pt", "BR"))
        lblValue.text = numberFormat.format(lastValue)
    }
    val date = tickerResponse?.ticker?.date?.let{ Date( it * 1000L) }
    val sdf = SimpleDateFormat("dd/MM/yyyy HH:mm:ss", Locale.getDefault())
    lblDate.text = sdf.format(date)
```
  - Se a requisição não for bem-sucedida, trata o erro com base no código HTTP
  - Envia uma notificação do tipo Toast exibindo o erro
```kotlin
} else {
    val errorMessage = when (response.code()) {
        400 -> "Bad Request"
        401 -> "Unauthorized"
        403 -> "Forbidden"
        404 -> "Not Found"
        else -> "Unknown error"
    }
    Toast.makeText(this@MainActivity, errorMessage,
                    Toast.LENGTH_LONG).show()
}
```
  - Trata exceções que podem acontecer durante a chamada e exibe a notifição do tipo Toast
```kotlin
} catch (e: Exception) {
    Toast.makeText(this@MainActivity, "Falha na chamada: ${e.message}",
                    Toast.LENGTH_LONG).show()
}
```

### MercadoBitcoinServiceFactory
Path: `app/src/main/java/lucaslimb/com/github/android_crypto_monitor/service/MercadoBitcoinServiceFactory.kt`

Constrói e retorna a instância com Retrofit para a API MercadoBitcoin
  - Utiliza o Builder para buscar a URL da API e converter Json para Gson (Kotlin)
  - Retorna a interface que será usada para as requisições
```kotlin
class MercadoBitcoinServiceFactory {
    fun create(): MercadoBitcoinService {
        val retrofit = Retrofit.Builder()
            .baseUrl("https://www.mercadobitcoin.net/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()

        return retrofit.create(MercadoBitcoinService::class.java)
    }
}
```

### MercadoBitcoinService
Path: `app/src/main/java/lucaslimb/com/github/android_crypto_monitor/service/MercadoBitcoinService.kt`

Interface que define o contrato da API
  - Utiliza o metódo GET para buscar o ticker da moeda Bitcoin (BTC) no endpoint da API
```kotlin
interface MercadoBitcoinService {
    @GET("api/BTC/ticker")
    suspend fun getTicker(): Response<TickerResponse>
}
```

### TickerResponse
Path: `app/src/main/java/lucaslimb/com/github/android_crypto_monitor/model/TickerResponse.kt`

Payload de resposta da API
  - Classe com o atributo Ticker (classe Ticker)
```kotlin
class TickerResponse(
    val ticker: Ticker
)
```
  - Classe com os atributos do ticker
```kotlin
class Ticker(
    val high: String,
    val low: String,
    val vol: String,
    val last: String,
    val buy: String,
    val sell: String,
    val date: Long
)
```

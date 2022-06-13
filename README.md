<img src="https://user-images.githubusercontent.com/89693955/173196650-5da9c1c4-c5ee-407c-85c6-0f491dfe8c5e.png" alt="Logo of the project" align="right">

# Mobile Developement

## How Yourney Works

- Register and Login with an account
- Share your condition today
- Choose between talk with chatbot or counselor
- Send your message and wait for the response

## Features

- Save your persona data
- Get your condition today
- Counseling menu
- Answer your question with chatbot

## Api Service

```Kotlin
    @FormUrlEncoded
    @POST("predict")
    fun postMessage(
        @Field("question") question: String
    ): Call<AnswerResponse>
```

## Chatbot Code

```Kotlin

class ChatbotActivity : AppCompatActivity() {

    private lateinit var adapterMessage: MessageAdapter
    private lateinit var adapterRecommend: RecommendAdapter
    private lateinit var chatbotBinding: ActivityChatbotBinding
    private val list = ArrayList<String>()
    private val timeStamp = Time.timeStamp()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        chatbotBinding = ActivityChatbotBinding.inflate(layoutInflater)
        setContentView(chatbotBinding.root)
        supportActionBar?.title = "Chatbot"

        // rv untuk chat
        recyclerViewMessage()

        // rv untuk recommend
        list.addAll(resources.getStringArray(R.array.data_recommend))
        recyclerViewRecommend()

        // jika item di click
        clickEvents()


    }


    private fun clickEvents() {
        chatbotBinding.layoutSend.setOnClickListener {
            val messageEditText = chatbotBinding.edMessage.text.toString()
            sendMessage(messageEditText)
        }

        chatbotBinding.edMessage.setOnClickListener {
            GlobalScope.launch {
                delay(1000)
                withContext(Dispatchers.Main) {
                    chatbotBinding.rvMessage.scrollToPosition(adapterMessage.itemCount - 1)
                }
            }
        }
    }

    private fun recyclerViewMessage() {
        adapterMessage = MessageAdapter()
        chatbotBinding.rvMessage.adapter = adapterMessage
        chatbotBinding.rvMessage.layoutManager = LinearLayoutManager(applicationContext)
    }

    private fun recyclerViewRecommend() {
        adapterRecommend = RecommendAdapter(list)
        chatbotBinding.rvRecommend.adapter = adapterRecommend
        chatbotBinding.rvRecommend.layoutManager =
            LinearLayoutManager(this, LinearLayoutManager.HORIZONTAL, false)
        adapterRecommend.setOnItemClickCallback(object : RecommendAdapter.OnItemClickCallback {
            override fun onItemClicked(dataRecommend: String) {
                sendMessage(dataRecommend)
            }
        })
    }

    private fun sendMessage(message: String) {

        if (message.isNotEmpty()) {
            chatbotBinding.edMessage.setText("")

            adapterMessage.insertMessage(Message(message, timeStamp, SEND_ID))
            chatbotBinding.rvMessage.scrollToPosition(adapterMessage.itemCount - 1)

            botResponse(message)
        }

    }

    private fun botResponse(message: String) {

        GlobalScope.launch {
            delay(1000)

            withContext(Dispatchers.Main) {
                val client = ApiConfig.getApiService().postMessage(message)
                client.enqueue(object : retrofit2.Callback<AnswerResponse> {
                    override fun onResponse(
                        call: Call<AnswerResponse>,
                        response: Response<AnswerResponse>
                    ) {
                        val responseBody = response.body()
                        if (response.isSuccessful && responseBody != null) {
                            adapterMessage.insertMessage(
                                Message(
                                    responseBody.answer,
                                    timeStamp,
                                    RECEIVE_ID
                                )
                            )
                            chatbotBinding.rvMessage.scrollToPosition(adapterMessage.itemCount - 1)
                        } else {
                            Log.e("ChatbotActivity", "onFailure: ${response.message()}")
                        }
                    }

                    override fun onFailure(call: Call<AnswerResponse>, t: Throwable) {
                        Log.e("ChatbotActivity", "onFailure: ${t.message}")
                    }

                })
            }
        }
    }

    override fun onStart() {
        super.onStart()

        GlobalScope.launch {
            delay(1000)
            withContext(Dispatchers.Main) {
                chatbotBinding.rvMessage.scrollToPosition(adapterMessage.itemCount - 1)
            }
        }
    }

}
```

## Add data to firebase

```Kotlin
//  save ke realtime database
            val ref = FirebaseDatabase.getInstance().getReference("users")
            val usersId = ref.push().key

            //  get Data
            val username = bundle?.getString("username")
            val email = bundle?.getString("email")
            val password = bundle?.getString("password")
            val study = bundle?.getString("study")
            val phone = bundle?.getString("phone")
            val city = bundle?.getString("city")
            val born = bundle?.getString("born")
            val gender = bundle?.getString("gender")

            val inputUser = Users(
                usersId,
                username,
                email,
                password,
                city,
                born,
                gender,
                study,
                phone,
                dream,
                hobby
            )

            if (usersId != null) {
                ref.child(usersId).setValue(inputUser).addOnCompleteListener {
                    aboutThreeViewModel.saveUsername(username)
                    Toast.makeText(
                        applicationContext,
                        "Data Added Suuccesfully",
                        Toast.LENGTH_SHORT
                    ).show()
                    showDialog()

                }
            } else {
                Toast.makeText(applicationContext, "There are data that have not been filled in", Toast.LENGTH_SHORT)
                    .show()
            }
```

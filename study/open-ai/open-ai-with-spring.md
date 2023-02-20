# 🤖 Open Ai

## build.gradle

```groovy

dependencies {
	implementation 'com.theokanning.openai-gpt3-java:client:0.8.0'
}
```



## 간단한 사용법

```java
@RestController
public class OpenAiGround {

    @Value("${openAi.apiKey}")
    private String key;

    @PostMapping("/openAi")
    public CompletionResult ApiTest(@RequestBody String text) {
        OpenAiService service = new OpenAiService(key);
        CompletionRequest request = CompletionRequest.builder()
            .model("text-davinci-002")
            .temperature(0.7)
            .prompt(text)
            .maxTokens(256)
            .topP(1.0)
            .frequencyPenalty(0.0)
            .presencePenalty(0.0)
            .build();
        return service.createCompletion(request);
    }

}
```



## apiKey

{% embed url="https://beta.openai.com/account/api-keys" %}

sk-...Y29p 형식의 키이다.

회원가입 후 프로필 -> view API keys

무료로 18달러만큼 사용가능하다.



## 결과

### Request

```json
POST http://localhost:8080/openAi
Content-Type: application/json

{
  "text": "what do you think about ai?"
}
```

### Response

```json
{
  "id": "cmpl-6EsAA9g3Zfne5adn7YdfNvGaZNB59",
  "object": "text_completion",
  "created": 1669002814,
  "model": "text-davinci-002",
  "choices": [
    {
      "text": "\n\n{\n  \"text\": \"I think AI is amazing! It has the ability to do so many things and help us in so many ways.\"",
      "index": 0,
      "logprobs": null,
      "finish_reason": "stop"
    }
  ]
}
```



> ```
> I think AI is amazing! It has the ability to do so many things and help us in so many ways.
> ```

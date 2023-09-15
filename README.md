# About
I see there are quite a few NuGet packages for Visual Studio to implement ChatGPT. The reality is that it's a lot easier than people may realize. This guide is designed to teach people how to use it using a basic "I want one response". This code is used in my [OutlookGPT](https://github.com/rkrehn/OutlookGPT) plugin.

# Requirements (from NuGet)

* Microsoft.AspNet.WebApi.Client
* System.Text.Json
* Newtonsoft.Json (not required, but recommended)

# Setup

1. First add the following lines at the top of your project where declare 'using":

```C
using System.Net.Http;
using System.Net;
using System.Text;
using Newtonsoft.Json;
```

2. Declare your API key (insert your [OpenAI API key](https://platform.openai.com/account/api-keys) in the quotes)

```C
string apiKey = "[api key]";
```

# Calling the Chat

Setting the TLS to 1.2 will ensure a secure connection. The next part is calling the API from the Chat Completions. Finally, we'll create the client and add a header with your API key from above.

```C
ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;
string requestUrl = $"https://api.openai.com/v1/chat/completions";
HttpClient client = new HttpClient();
client.DefaultRequestHeaders.Add("Authorization", $"Bearer {apikey}");
```

# Setting up the request

The following code creates the JSON request to OpenAI. You can review all the options you can include here: [https://platform.openai.com/docs/api-reference/chat/create](https://platform.openai.com/docs/api-reference/chat/create)0

It's worth noting that only the "model" and the "messages" are required in the body. There are [many models](https://platform.openai.com/docs/models), but I'm using **gpt-3.5-turbo** in this example. Likewise, there are a few roles you can use such as system, user, assistance, or function. Typically, the **system** role defines what you want to accomplish. In our example, we'll tell ChatGPT **sytem** role that is acting as a travel agent (used on my [ReisPlan](https://www.reisplan.net) site).

I'm using max_tokens at 1024 to prevent a high cost. The **gpt-3.5-turbo** model can only support 4,096 tokens. I also use a [temperature](https://platform.openai.com/docs/guides/gpt/how-should-i-set-the-temperature-parameter) of 0.7, which gives a good sense of creativity, without being too consistent. Generally, temperature should be between 0.05 and 1, where 1 is very creative and 0.05 is very consistent.

```C
  var requestJson = new
  {
      messages = new[]
      {
          new
          {
              role = "system",
              content = "You are a travel agent. Please provide a detailed itinerary based on the user's input."
          },
          new
          {
              role = "user",
              content = "I am traveling to Denver, CO for three days and I like beer."
          }
      },
      max_tokens = 1024,
      temperature = 0.7,
      model = "gpt-3.5-turbo"
  };
```

Next, build a **HTTP content** using the serialized **requestJson** variable that we created above:

```C
StringContent content = new StringContent(JsonConvert.SerializeObject(requestJson), Encoding.UTF8, "application/json");
```

# Receiving the response

This next part is sending the message over to OpenAI and retrieving a JSON response.

```C
// Send the request and receive the response
HttpResponseMessage response = client.PostAsync(requestUrl, content).Result;
string responseJson = response.Content.ReadAsStringAsync().Result;
```

Then, we'll deserialize the response into a readable format.

```C
// Extract the completed text from the response
dynamic responseObject = JsonConvert.DeserializeObject(responseJson);
```

You will receive a JSON response like this in the responseObject (don't copy this):

```JSON
{
  "id": "chatcmpl-123",
  "object": "chat.completion",
  "created": 1677652288,
  "model": "gpt-3.5-turbo-0613",
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "content": "\n\nSure, I can recommend you check out Livin the Dream brewery in Littleton and Denver Beer Co in Englewood.",
    },
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 9,
    "completion_tokens": 12,
    "total_tokens": 21
  }
}
```

* choices[0] is the first response
* message is the node within the choices[0]
* content is the response you're looking for

The **completedText** string below is created from those three nodes above in the responseObject:

```C
string completedText = responseObject.choices[0].message.content;
```

**completedText** will look like:

> Sure, I can recommend you check out Livin the Dream brewery in Littleton and Denver Beer Co in Englewood.

# Everything together

Now, put it together in all its glory:

```C
// setup variables and HTTPClient
string apiKey = "[api key]";
ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;
string requestUrl = $"https://api.openai.com/v1/chat/completions";
HttpClient client = new HttpClient();
client.DefaultRequestHeaders.Add("Authorization", $"Bearer {apikey}");

// build JSON request
  var requestJson = new
  {
      messages = new[]
      {
          new
          {
              role = "system",
              content = "You are a travel agent. Please provide a detailed itinerary based on the user's input."
          },
          new
          {
              role = "user",
              content = "I am traveling to Denver, CO for three days and I like beer."
          }
      },
      max_tokens = 1024,
      temperature = 0.7,
      model = "gpt-3.5-turbo"
  };

StringContent content = new StringContent(JsonConvert.SerializeObject(requestJson), Encoding.UTF8, "application/json");

// Send the request and receive the response
HttpResponseMessage response = client.PostAsync(requestUrl, content).Result;
string responseJson = response.Content.ReadAsStringAsync().Result;

// Extract the completed text from the response
dynamic responseObject = JsonConvert.DeserializeObject(responseJson);
string completedText = responseObject.choices[0].message.content;
```

> Sure, I can recommend you check out Livin the Dream brewery in Littleton and Denver Beer Co in Englewood.

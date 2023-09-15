# About
I see there are quite a few NuGet packages for Visual Studio to implement ChatGPT. The reality is that it's a lot easier than people may realize. This guide is designed to teach people how to use it using a basic "I want one response". This code is used in my [OutlookGPT](https://github.com/rkrehn/OutlookGPT) plugin.

# Requirements (from NuGet)

* Microsoft.AspNet.WebApi.Client
* System.Text.Json
* Newtonsoft.Json (not required, but recommended)

# Setup

1. First add the following lines at the top of your project where declare 'using":

```
using System.Net.Http;
using System.Net;
using System.Text;
using Newtonsoft.Json;
```

2. Declare your API key (insert your OpenAI API key in the quotes)

```
string apiKey = "[api key]";
```

# Calling the Chat

Setting the TLS to 1.2 will ensure a secure connection. The next part is calling the API from the Chat Completions. Finally, we'll create the client and add a header with your API key from above.

```
ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;
string requestUrl = $"https://api.openai.com/v1/chat/completions";
HttpClient client = new HttpClient();
client.DefaultRequestHeaders.Add("Authorization", $"Bearer {apikey}");
```

# Setting up the request

The following code creates the JSON request to OpenAI. You can review all the options you can include here: [https://platform.openai.com/docs/api-reference/chat/create](https://platform.openai.com/docs/api-reference/chat/create)0

It's worth noting that only the "model" and the "messages" are required in the body. There are [many models](https://platform.openai.com/docs/models), but I'm using gpt-3.5-turbo in this example. Likewise, there are a few roles you can use such as system, user, assistance, or function. Typically, the system is the one role that defines what you want to accomplish. In our example, we'll tell ChatGPT that is acting as a travel agent (used on my [ReisPlan](https://www.reisplan.net) site).

In the example below, I'm using max_tokens at 1024 to prevent a high cost. The gpt-3.5-turbo model can only support 4,096. I also use a [temperature](https://platform.openai.com/docs/guides/gpt/how-should-i-set-the-temperature-parameter) of 0.7, which gives a good sense of creativity, without being too consistent. Generally, temperature should be between 0.05 and 1, where 1 is very creative and 0.05 is very consistent.

```
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

Next, build a **web content** using the serialized **requestJson** variable that we created above:

```
StringContent content = new StringContent(JsonConvert.SerializeObject(requestJson), Encoding.UTF8, "application/json");
```

# Receiving the response

This next part is sending the message over to OpenAI and retrieving a JSON response.

```
// Send the request and receive the response
HttpResponseMessage response = client.PostAsync(requestUrl, content).Result;
string responseJson = response.Content.ReadAsStringAsync().Result;
```

Then, we'll deserialize the response into a readable format.

```
// Extract the completed text from the response
dynamic responseObject = JsonConvert.DeserializeObject(responseJson);
string completedText = responseObject.choices[0].message.content;
```

The **completedText** string contains the response from ChatGPT.

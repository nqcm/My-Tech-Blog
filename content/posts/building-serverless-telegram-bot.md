---
title: "🚀 Building a Telegram Bot with AWS API Gateway and AWS Lambda"
date: 2018-11-18T09:45:28+02:00
tags: ["Bots", "Telegram", "AWS", "Lambda", "API Gateway", "Python"]
summary: In this guide I will demonstrate making a simple serveless Telegram bot that echoes everything a user inputs, using AWS API Gateway and AWS Lambda.
cover:
  image: "/img/telegram-bot-aws/telegram-bot-with-aws-lambda.png"
  alt: "Image of a bot with computer"
---

<br>

![Telegram bot with AWS Lambda and AWS API Gateway](/img/telegram-bot-aws/telegram-bot-with-aws-lambda.png#center)

Over the past year I have made my fair share of bots. I have made bots using services like Chatfuel and Flowxo, I have made bots from scratch in Python and Nodejs and I have used frameworks like Microsoft Bot Framework. I have also experimented with different deploying options for my coded-from-scratch bots including VPS, web servers, and serverless architecture.

Out of all of these options, I keep coming back to [serverless](https://en.wikipedia.org/wiki/Serverless_computing) deployment. Not only can using a serverless computing architecture be extremely economical in many use cases, it is also very low maintanance. You don't have to worry about maintaining a server at all. Just write your code, upload it to the cloud and then you can pretty much forget about it.

## Echo Telegram bot with AWS API Gateway and AWS Lambda

In this guide I will demonstrate making a simple serveless Telegram bot that echoes everything a user inputs, using AWS API Gateway and AWS Lambda. But first a little about the services we will use.

### What is API Gateway

Amazon API Gateway is a publicly available endpoint for our code that runs on AWS Lambda, Amazon EC2, or other publicly addressable web services. It helps us deliver mobile and web application backends in a secure and scalable way.

### What is AWS Lambda

[AWS Lambda](https://aws.amazon.com/lambda/) is a service that lets us create [stateless](https://en.wikipedia.org/wiki/Stateless_protocol) functions (or blocks of code) in the cloud which can be invoked (or run) through a multitude of triggers. There are many other services beside AWS which let us create cloud functions, like [Google Cloud Functions](https://cloud.google.com/functions/) and [Azure Functions](https://cloud.google.com/functions/).

Bots or chatbots are an ideal candidate for this kind of computing service since in essence chatbots are just a piece of code which is triggered through a user interaction. Many popular chatbot making platforms are already using cloud functions as the backbone of their service.

With all that introduction out of the way, let's get started!

### Registering a bot on Telegram

To follow along this guide, you will need a Telegram bot. Registering a new bot in Telegram is really simple. Just send a "/newbot" command to Telegram's own bot [BotFather](https://t.me/BotFather) and follow the instructions. You will receive a bot token, copy this token and keep it handy.

You can customize the descriptions and about section of your bot, as well as change the profile picture for the bot through BotFather. Once you are happy with your bot's settings, you are ready to proceed.

The way a Telegram bot works is that when a user sends a message to our bot's account, Telegram sends that message to our code. Our code processes that message, and after doing whatever it needs to do, it sends back a message to Telegram servers which is then delivered to the user.

There are two ways we can get new updates from Telegram servers.

1. Long polling in which our code keeps asking Telegram servers if there is a new update.
2. Setting webhooks in which we ask Telegram servers to send any new message/update received to us through a HTTP request.

Setting webhooks is the most resource friendly and scalable way to receive new updates. And this is what we are going to do. But for that we will need a secure URL where Telegram servers can push all updates. Enter AWS API Gateway.

### Creating an API Gateway

If you don't have an Amazon account, sign up for one. You will need to provide your credit card details but Amazon has a very good free tier and to follow along this guide shouldn't cost you anything.

Sign in to the [AWS Console](https://aws.amazon.com) and search API Gateway from the services.

Click on 'Create API'

![Create new API](/img/telegram-bot-aws/telegram-aws-1.jpg#center)

Give your API a name and description and then click Create API.

![Settings of the new API](/img/telegram-bot-aws/telegram-aws-2.jpg#center)

Before defining any resources for this API, let's first create our Lambda function.

### Creating a lambda function

Sign in to the [AWS Console](https://aws.amazon.com) and choose Lambda from the services. Click 'Create Function'.

![Click Create Function](/img/telegram-bot-aws/telegram-aws-3.jpg#center)

Choose 'Author from scratch' option for now. Once you have followed this guide till the end, you can try exploring the different blueprints provided by Amazon.

![Choose author from scratch](/img/telegram-bot-aws/telegram-aws-5.jpg#center)

Give you function a name, choose your favorite runtime (in the case of this guide choose Python3.6).

![Settings for Lambda function](/img/telegram-bot-aws/telegram-aws-4.jpg#center)

Each Lambda function has an IAM role (execution role) associated with it. You specify the IAM role when you create your Lambda function. Permissions you grant to this role determine what AWS Lambda can do when it assumes the role. You can read more about managing permissions [here](https://docs.aws.amazon.com/lambda/latest/dg/intro-permission-model.html#lambda-intro-execution-role).

For the purpose of this guide, we will select 'Create a new role from templates' from the drop down menu. Give your role a name and then search 'basic' in the 'Policy templates' dropdown list. Select 'Basic Lambda @ Edge permissions'. Then click 'Create function'.

![Select lambda role](/img/telegram-bot-aws/telegram-aws-13.jpg#center)

This will take you to your function. Scroll down and familiarize yourself with different options. You need to pay attention to a few options in particular:

1. Environment variables - any environment variables that your code uses must be declared here.
1. Basic settings from where you can change the allocated memory and timeout. Please note that changing these settings may affect the costs. You can learn more about Lambda pricing [here](https://aws.amazon.com/lambda/pricing/).

![Basic settings for Lambda](/img/telegram-bot-aws/telegram-aws-6.jpg#center)

Before we add any code to our function, let's first add a trigger to it. We want that everytime someone sends a message to our bot on Telegram, the Telegram servers should push it to our URL wher our code is hosted. In other words we want a HTTP GET/POST request to trigger our code and we will do this through our API gateway.

You can add the trigger to your Lambda function from Lambda's own page as well but for now we will head over to API Gateway page to connect our Lambda function with the API and to configure the API.

### Connecting Lambda function to API Gateway

Sign in to the [AWS Console](https://aws.amazon.com) and search API Gateway from the services.

Select the API which we created earlier. We will need to add some methods to this API. From the dropdown 'Actions' list select 'Create Method'

![Select Create method](/img/telegram-bot-aws/telegram-aws-7.jpg#center)

You will see another dropdown list appear under "/". From here you can add methods such as "GET" and "POST". Since we are making a really simple bot, we don't want a bunch of different methods. So for simplicity's sake we will just choose "ANY" method. It means that no matter what kind of request is sent to this URL, it will always trigger the same code.

![Select ANY method](/img/telegram-bot-aws/telegram-aws-8.jpg#center)

Hit the small check sign next to "ANY" and it will take you to a new screen where you can choose what to do once this endpoint receives any kind of request. From here choose "Lambda Function" for integration type and the name of your Lambda function from the list. Also check the "Use Lambda Proxy Integration" option which will enable us to access the request details inside our Lambda function.

![Configure your endpoint](/img/telegram-bot-aws/telegram-aws-9.jpg#center)

Click save. Test your API on the next screen to check if everything is working the way it should.

![Test your API](/img/telegram-bot-aws/telegram-aws-10.jpg#center)

Then click "Deploy API" from the "Actions" list.

![Select Deploy API](/img/telegram-bot-aws/telegram-aws-11.jpg#center)

Choose a stage name like "V1" or "beta" for the stage name. Once your API is deployed you will get an invoke URL. If you click on this URL, you should get a simple web page with "Hello from Lambda!". Note down this URL.

### Setting Webhooks

So now we have a secure URL which is connected to our code living in a cloud function. Cool! We can go ahead and tell Telegram to send any future messages received by our bot to this address.

Setting a webhook is as simple as typing the following in your browser and hitting enter. (Make sure to substitute the place holder text)

```
"https://api.telegram.org/bot<your-bot-token>/setWebHook?url=<your-API-invoke-URL>"

```

You will get a confirmation message from Telegram and from now on any messages sent to your bot will be pushed to this URL.

### Adding code to Lambda function

Go ahead and try sending a message from your own Telegram account to the bot.

So apparently nothing happens 🤔 and that is because our code is not yet doing anything. But if you want to make sure that your function was triggered by receiving the webhook, you can do so by checking the CloudWatch logs. Select "Monitoring" tab and then "View logs in CloudWatch" on the Lambda function's page.

![logs in CloudWatch](/img/telegram-bot-aws/telegram-aws-12.jpg#center)

We will write some very basic code which will just return the message sent by the user. Replace the code in your Lambda function's visual editor with the following:

```Python
import json
from botocore.vendored import requests

TELE_TOKEN='<YOUR-BOT-TOKEN>'
URL = "https://api.telegram.org/bot{}/".format(TELE_TOKEN)

def send_message(text, chat_id):
    final_text = "You said: " + text
    url = URL + "sendMessage?text={}&chat_id={}".format(final_text, chat_id)
    requests.get(url)

def lambda_handler(event, context):
    message = json.loads(event['body'])
    chat_id = message['message']['chat']['id']
    reply = message['message']['text']
    send_message(reply, chat_id)
    return {
        'statusCode': 200
    }

```

This is a very simple piece of code as you can see. We are just extracting the message and chat id from the event object and sending it back as a query string along with the chat id in a HTTP GET request to https://api.telegram.org with our token. We also return a JSON object with "statusCode" as "200" so telegram servers can know that the request was received successfully.

You can read more about interacting with the Telegram API [here](https://core.telegram.org/bots/api).

Go ahead and send your bot some messages and it should return the same messages back.

![Telegram bot](/img/telegram-bot-aws/telegram-aws-14.jpg#center)

Of course, for a production bot we will do a few things differently, for example adding the bot token as an environment variable and adding security layers. But that is beyond the scope of this simple tutorial.

### Conclusion

This brings us to the end of this guide. We learned how to create a simple echo bot for Telegram but more than that we learned how to host it completely serverless in the cloud.

Traditionally to run this kind of bot, we would need to write our code not as a simple script but within a micro service framework like Flask. We would need basic scaffolding like a web server to serve the code, a SSL certificate to make our connection secure and so on. But thanks to the powerful Lambda functions and the API Gateway our task was made significantly simpler.

<hr>

_Image credit_ <a href="https://www.freepik.com/free-vector/chatbot-concept-background-with-happy-robot_2411604.htm">Designed by Freepik</a>

<br><br>

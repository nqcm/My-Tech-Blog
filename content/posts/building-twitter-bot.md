---
title: "🤖 Building A Twitter Bot"
date: 2018-10-02T17:05:33+02:00
tags: ["Bots", "Twitter", "AWS", "Lambda", "Python"]
cover:
  image: "/img/twitter-bot-aws/aws-lambda-3.jpg"
summary: "How I built a Twitter bot that tweets Tajweed rules every eight hours using Python and AWS Lambda. #Python #AWS #Twitter #Bots"
---

<br>
I built a Twitter bot that tweets [Tajweed](https://en.wikipedia.org/wiki/Tajwid) rules every eight hours using Python and AWS Lambda.

<br>

<div class="center">
    <blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">The best of you is he who learns the Quran and teaches it. ~ Prophet Muhammad SAW<br> Today&#39;s <a href="https://twitter.com/hashtag/tajweed?src=hash&amp;ref_src=twsrc%5Etfw">#tajweed</a> rule... <a href="https://t.co/SMhyE2Jgi9">pic.twitter.com/SMhyE2Jgi9</a></p>&mdash; TajweedBot (@tajweedbot) <a href="https://twitter.com/tajweedbot/status/1047116062532096001?ref_src=twsrc%5Etfw">October 2, 2018</a></blockquote>
    <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

<br>

I have [previously written](https://blog.naveeraashraf.com/posts/making-tajweed-bot/) about the writing and designing part of the project and in this part outlines the technical details on how the bot was made and works.

<br>

<a href="https://twitter.com/tajweedbot?ref_src=twsrc%5Etfw" class="twitter-follow-button" data-size="large" data-show-count="false">Follow @tajweedbot</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

You can check out the completed code [here](https://github.com/nqcm/tajweedbot)

## Using S3

I decided on using an Amazon S3 bucket to store the image cards as it has a nice free tier. Since I was planning on hosting the bot on AWS Lambda, the integration between different AWS services is seamless.

S3 has a nice web based uploader to upload stuff but if you have huge amount of files to transfer, using their CLI would be the way to go. For my purpose however, I just used the website's uploader.

![S3 Uploader](/img/twitter-bot-aws/aws-s3-upload.jpg#center)

## Testing Boto3

My first step was to test the usage of Amazon's SDK for Python, the [Boto3 library](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html). Strange name, I know. But it is [named after a dolphin](https://github.com/boto/boto3/issues/1023) 'Boto' which navigates the Amazon rainforest's eco system. Cool!

So after installing Boto3 in my virtual environment, I wrote a simple script to test if I could download a file using Boto3. Turned out I could!

```Python
import boto3
import botocore

BUCKET_NAME = 'tajweedbot'
KEY = 'round-zero.jpg'

s3 = boto3.resource('s3')

try:
    s3.Bucket(BUCKET_NAME).download_file(KEY, 'local.jpg')
except botocore.exceptions.ClientError as e:
    if e.response['Error']['Code'] == "404":
        print("The object does not exist.")
    else:
        raise
```

## Choosing A Random Image

My next step was to grab a random image from the bucket with every execution. Using the `bucket.objects.all()` method I could count all the objects in that bucket and feeding it to `random.randint()` function, I was able to get a random number between 1 and the total number of files in that bucket. Adding it with the '.png' extension, I assigned it to the KEY variable replacing the hard-coded KEY.

```Python
obj = str(random.randint(1, sum(1 for _ in bucket.objects.all())))
KEY = obj + '.png'
```

But this is not an elegant solution. For one, my file names had to be numbers like '1.png' etc. Secondly there must be a dedicated bucket for the images being used by this bot.

An ideal solution would be to make a JSON file containing the names of the images and then I could get a random file name from the JSON document with a simple script like:

```Python
s3 = boto3.resource('s3')
bucket= s3.Bucket(BUCKET_NAME)
obj = s3.Object(BUCKET_NAME, "meta.json")
data = obj.get()
js = json.loads(data['Body'].read().decode('utf-8'))
pic = str(random.randint(1, len(js))) + '.png'
```

So in my next iteration over the project, I may implement this method. But for now I am sticking with my not-so-elegant solution.

## Tweeting the Image

Next step was to make my bot tweet this downloaded image. That was easy using [Twython](https://twython.readthedocs.io/en/latest/). First I created a new Twitter account, then registered an application at <https://apps.twitter.com>, and grabed my application's Consumer Key and Consumer Secret. Sending a tweet can be easily done through the following lines of code.

```Python
from twython import Twython
twitter = Twython(APP_KEY, APP_SECRET,
                OAUTH_TOKEN, OAUTH_TOKEN_SECRET)
twitter.update_status(status='See how easy using Twython is!')
```

You can read more about the dynamic arguments of Twitter API [here](https://developer.twitter.com/en/docs/tweets/post-and-engage/api-reference/post-statuses-update).

Putting everything together my code looked like this:

```Python
import boto3
import botocore
import os
import random
import tempfile
from twython import Twython, TwythonError

BUCKET_NAME = 'YOU-BUCKET-NAME'
CONSUMER_KEY = 'TWITTER-APP-CONSUMER-KEY'
CONSUMER_SECRET = 'TWITTER-APP-CONSUMER-SECRET'
ACCESS_TOKEN = 'ACCESS-TOKEN'
ACCESS_SECRET = 'ACCESS-SECRET'


s3 = boto3.resource('s3')
bucket= s3.Bucket(BUCKET_NAME)

obj = str(random.randint(1, sum(1 for _ in bucket.objects.all())))
KEY = obj + '.png'

twitter = Twython(CONSUMER_KEY, CONSUMER_SECRET, ACCESS_TOKEN, ACCESS_SECRET)


tmp_dir = tempfile.gettempdir()
path = os.path.join(tmp_dir, KEY)
print("created directory at " + path)
try:
    s3.Bucket(BUCKET_NAME).download_file(KEY, path)
    print('file moved to temp directory')
    with open(path, 'rb') as img:
        try:
            twit_resp = twitter.upload_media(media=img)
            twitter.update_status(status="The best of you is he who learns the Quran and teaches it. ~ Prophet Muhammad SAW" , media_ids=twit_resp['media_id'])
            print("image tweeted")
        except TwythonError as e:
            print(e)
except botocore.exceptions.ClientError as e:
    if e.response['Error']['Code'] == "404":
        print("The object does not exist.")
    else:
        raise
```

So at this point I had a working script that downloaded a random image from S3, and tweeted it with a status. Now to make this script into a bot, I needed some automation. Enter AWS Lambda!

## AWS Lambda

Don't confuse [AWS Lambda](https://aws.amazon.com/lambda/?p=tile) with Python Lambda functions. If you have never heard about [AWS Lambda](https://aws.amazon.com/lambda/?p=tile) before, this service allows you to upload some code in a zip format to Amazon servers, set some triggers which will execute this code and then forget about it until the end of month when Amazon bills you for the service. 😉 You don't even need to know anything about the machine that runs them. No servers to maintain and comparatively cheaper.

This is what [Lambdas look like in Python](https://docs.aws.amazon.com/lambda/latest/dg/python-programming-model-handler-types.html):

```Python
def handler_name(event, context):
    ...
    return some_value
```

The event is what you decide to trigger the Lambda function. And the context sets up all the runtime information needed to interact with AWS and run the function. So now all I had to do was to put my script within the above function. Like so:

```Python
import boto3
import botocore
import os
import random
import tempfile
from twython import Twython, TwythonError

BUCKET_NAME = 'YOU-BUCKET-NAME'
CONSUMER_KEY = 'TWITTER-APP-CONSUMER-KEY'
CONSUMER_SECRET = 'TWITTER-APP-CONSUMER-SECRET'
ACCESS_TOKEN = 'ACCESS-TOKEN'
ACCESS_SECRET = 'ACCESS-SECRET'


s3 = boto3.resource('s3')
bucket= s3.Bucket(BUCKET_NAME)

def lambda_handler(event, context):

    obj = str(random.randint(1, sum(1 for _ in bucket.objects.all())))
    KEY = obj + '.png'

    twitter = Twython(CONSUMER_KEY, CONSUMER_SECRET, ACCESS_TOKEN, ACCESS_SECRET)


    tmp_dir = tempfile.gettempdir()
    path = os.path.join(tmp_dir, KEY)
    print("created directory at " + path)
    try:
        s3.Bucket(BUCKET_NAME).download_file(KEY, path)
        print('file moved to temp directory')
        with open(path, 'rb') as img:
            try:
                twit_resp = twitter.upload_media(media=img)
                twitter.update_status(status="The best of you is he who learns the Quran and teaches it. ~ Prophet Muhammad SAW" , media_ids=twit_resp['media_id'])
                print("image tweeted")
            except TwythonError as e:
                print(e)
    except botocore.exceptions.ClientError as e:
        if e.response['Error']['Code'] == "404":
            print("The object does not exist.")
        else:
            raise
```

My code was already making use of a temporary directory so I didn't need to make any changes for Lambda there, since it is [possible to access the local filesystem in Lambda](https://stackoverflow.com/questions/35641994/accessing-local-filesystem-in-aws-lambda/35642189). All I needed now was to [package my code](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python-how-to-create-deployment-package.html) in a zip format with all the dependencies.

I created a new Lambda function from my AWS console. It is fairly easy to setup a new Lambda. I will try to capture the procedure with images below:

![Create Lambda function](/img/twitter-bot-aws/aws-lambda-1.jpg#center)

![setting up Lambda function](/img/twitter-bot-aws/aws-lambda-2.jpg#center)

On my Lambda function's page, I didn't add any triggers. Scrolling down to the 'function code' section, I uploaded my zipped code.

![Upload code](/img/twitter-bot-aws/aws-lambda-3.jpg#center)

I tested my Lambda function from the top of the page. Once my code was executing successfully, I went to the [AWS Cloudwatch service](https://aws.amazon.com/cloudwatch/) from within my console. And added a new cron job through the steps from the image below:

![Add trigger event](/img/twitter-bot-aws/aws-lambda-4.jpg#center)

And ta-da! My bot started tweeting.

## Conclusion

So this is how I created a Twitter bot in Python and deployed it using AWS Lambda. Of course there are still many improvements, for example, tweaking the code so no image repeats more than once every few days etc. But for now I am happy with this bot and see this project as a [MVP](https://en.wikipedia.org/wiki/Minimum_viable_product).

This was a simple project and should take any sane human just a couple of hours to make and deploy. Note: For harassed homeschooling moms, it can take up to two weeks to make and deploy. 😬

<br>

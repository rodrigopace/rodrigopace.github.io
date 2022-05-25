---
layout: page
name: Episode 2 - Non-Public Buckets Can Still Leak Information
---

# Episode 2 - Non-Public Buckets Can Still Leak Information

<div class="video-container"><iframe src="https://www.youtube.com/embed/qJzwwC-aZfs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

Welcome to Episode 2 of the “Head in the Clouds” Video Series.

Today, I want to talk about AWS S3 Buckets.

We have all read the horror stories about misconfigured Amazon S3 Buckets that expose
millions of customer records and other sensitive information. Something that you may
not know is that a bucket that is not public can still expose sensitive information.

How? Via the name of the bucket.

I once had a client that had created a bucket for everyone of their customers and
thereby leaked their customer list. Granted, this is not the most serious issue in
the cloud, but let’s take a look and you may learn more about AWS S3 in the process.

Imagine, for a moment, we are a company called “ACME Manufacturing.” I always like to
use ACME from the Road Runner cartoons that played on Saturday morning television when
I was a kid.

Let’s also imagine that we plan to create a bucket for each of our customers. And, let’s
say that Walmart is our first customer. So, let’s make a bucket for Walmart:

```
$ aws s3 mb s3://acme-customers-walmart
make_bucket: acme-customers-walmart
```

Now upload some sensitive information to the bucket:

```
$ echo "Juicy Sensitive Data" > customerData.txt
$ aws s3 cp  customerData.txt s3://acme-customers-walmart/customerData.txt
upload: ./customerData.txt to s3://acme-customers-walmart/customerData.txt
```

In the AWS Web Console, we can see that the URL do our data is:
https://acme-customers-walmart.s3.amazonaws.com/customerData.txt

![Image of the AWS S3 Console showing bucket metadata](/images/ep2/Picture1.png)

Next attempt to open the link in a new browser window. I like to use Incognito mode in
Chrome so that I am sure that there is no session data in memory to authenticate me to
the AWS endpoint.

We should get an XML message stating that access is denied:

![Image of the access denied error message](/images/ep2/Picture2.png)

Ok, that is expected. It is not a public bucket after all. Let’s change the name of the
file we are trying to fetch to “customerData2.txt”

![Image of the access denied error message](/images/ep2/Picture3.png)

Still Denied. Ok, lets play at the bucket level, by removing the file name from the URL:

![Image of the access denied error message](/images/ep2/Picture4.png)

Access denied, as expected. One more experiment. Try a non-existent bucket, such as
“**acme-customers-kmart**”

![Image of the no such bucket error message](/images/ep2/Picture5.png)

Interesting. Now we are getting a different error message.

This is a variation of a user account enumeration attack. (For more information,
see [http://bit.ly/OWASP-4-03-04](http://bit.ly/OWASP-4-03-04))

Because we can get different error messages, we now might have a means to enumerate the
buckets for customer names.

Rather than using a browser, let’s use curl:

```
curl -i https://acme-customers-walmart.s3.amazonaws.com/
curl -i https://acme-customers-kmart.s3.amazonaws.com/
```

![Image of the output of the previous commands](/images/ep2/Picture6.png)

Note that the HTTP response codes are also different.

We can tweak our command to just return the HTTP Response code:

```
curl -I -s  https://acme-customers-walmart.s3.amazonaws.com/ | grep HTTP
```

![Image of the output of the previous command](/images/ep2/Picture7.png)

We can even parameterize the customer name and create a FOR loop:

```
for NAME in kmart walgreens walmart aldi
do
    RESPONSE=$(curl -I -s  https://acme-customers-$NAME.s3.amazonaws.com/ | grep HTTP)
    echo $NAME": "$RESPONSE
done
```

![Image of the output of the previous command](/images/ep2/Picture8.png)

Interesting! We can see the expected response codes for Kmart and Walmart. But it
looks like ACME made their Walgreens bucket public!!

What is going on with the **acme-customers-aldi** bucket?

Let’s run the curl command again and fetch the complete response:

```
curl -i https://acme-customers-aldi.s3.amazonaws.com/
```

![Image of the output of the previous command](/images/ep2/Picture9.png)

That is nice of Amazon. Not only do we know that the bucket exists, but that it is hosted in the Frankfurt, Germany region (eu-central-1) because it uses the S3 endpoint at **s3.eu-central-1.amazonaws.com** rather than Northern Virginia, which uses **s3.amazonaws.com**. More potentially useful recon info.

## Acknowledgements

Today’s topic builds upon the work of Robin Woods (“DigiNija”) and his [Bucket Finder](https://digi.ninja/projects/bucket_finder.php) tool. There have been other bucket scanning tools, but for today’s topic, I wanted to make sure people understand how these tools work under the covers.

## Important Take-Aways
1.	It is generally unwise to have guessable bucket names. For example, a bucket name of acme-customers-walmart-282901034857378292 would be better, as long as the numerical string appended to the end is random and not something like your AWS account number.
2.	It would be better to avoid using meaningful words and names such as “ACME”, “Walmart” or even “customers” for bucket names. Remember that you can always use tags and tags are not exposed via DNS. Nothing wrong with a bucket named “cx2821023238238” assuming it is unique.
3.	If you want to have meaningful names and phrases, do that at the folder level inside a bucket as folders are not exposed by default (like bucket names are).
4.	 Lastly, remember that just because there is a bucket name that contains the name of a well-known company, doesn’t mean that that bucket is controlled by that company. In the examples used in today’s talk, I created buckets for a couple of top retailers even though I have no affiliation with them.

## Wrap Up

And that is it for today’s episode.

If you have thoughts or comments on today’s episode, feel free to chime in on our moderated Google Group by shooting a note to [head-in-the-clouds-security@googlegroups.com](mailto://head-in-the-clouds-security@googlegroups.com).

Tune in next time for another installment of “Head in the Clouds.” Announcements of new episodes are made on the [SANS Cloud Security Twitter feed](https://twitter.com/SANSCloudSec).

Meanwhile, be sure to check out the other great videos on the [SANS Cloud Security YouTube Channel](https://www.youtube.com/c/SANSCloudSecurity).

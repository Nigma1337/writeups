## Level 1
The initial hint is: 

```
This level is *buckets* of fun. See if you can find the first sub-domain.
```

buckets, s3 buckets, you know the deal.

After installing the [aws-cli](https://aws.amazon.com/cli/), i tried running `aws s3 ls s3://flaws.cloud/`, but was met my aws telling me the access key i specified was not in their system.
The solution was to use the argument `--no-sign-request`

Cool

## Level 2
Initial hint: 

```
The next level is fairly similar, with a slight twist. You're going to need your own AWS account for this. You just need the free tier.
```


So first off, i tried running the previous commands again, didn't work.
Then, i went and got myself an account on AWS, created myself an access token, configured my CLI and poof, `aws s3 ls s3://<level-2-domain>/` worked

## level 3
Initial hint: 

```
The next level is fairly similar, with a slight twist. Time to find your first AWS key! I bet you'll find something that will let you list what other buckets are.
```

Cool, lets try listing the level 3 bucket, `aws s3 ls s3://<level-3-domain>/`.
In this bucket, we find a ".git" folder, cool let's download it `aws s3 cp --recursive s3://<level-3-domain>/`
Looking inside the git folder, more specifically "logs/HEAD", we see a commit with the message "Oops, accidentally added something I shouldn't have", lets restore to before that commit and check what he left, eh?
`git chechout <hash of the first commit>`leaves us with an access_keys.txt file. Using the initial hint, it mentions listing other buckets, so after configuring the profile, we run `aws s3 ls`, which shows us all of the buckets. Lets go to the URL for level 4

## level 4
Initial hint: 

```For the next level, you need to get access to the web page running on an EC2 at 4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud
It'll be useful to know that a snapshot was made of that EC2 shortly after nginx was setup on it.
```

Alright, so we gotta check out EC2 and volume snapshots now. Just running `aws ec2 describre-snapshots` spams your terminal with public snapshots, so lets find a way to limit it a bit.
Listing EC2 instances via `aws ec2 describe-instances` shows us one instance, and this like: `EBS	2017-02-12T22:29:25.000Z	True	attached	vol-04f1c039bc13ea950`
which is what we're interested in.
Now, lets filter our describe-snapshots call. `aws ec2 describe-snapshots --filters Name=volume-id,Values=vol-04f1c039bc13ea950` will show us this line:
`SNAPSHOTS		False	975426262029	100%	snap-0b49342abd1bdcb89	2017-02-28T01:35:12.000Z	completed	standard	vol-04f1c039bc13ea950	8
TAGS	Name	flaws backup 2017.02.27`, pog that's our snapshot id.

Create a volume on our account from the snapshot via `aws ec2 create-volume --availability-zone us-west-2a --region us-west-2  --snapshot-id  snap-0b49342abd1bdcb89`
Then, create a ec2 instance and attach the volume we just created.

ssh into the ec2, `mount /dev/<device> /mnt`, hop into /mnt/etc/nginx, read available-sites.d/default, realise you cant just read the password from .htpasswd because its a hash, cat the files in /mnt/var/www/html and poof, level 5 time

## level 5
initial hint: 

```
This EC2 has a simple HTTP only proxy on it. Here are some examples of it's usage:
http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/flaws.cloud/
http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/summitroute.com/blog/feed.xml
http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/neverssl.com/
See if you can use this proxy to figure out how to list the contents of the level6 bucket at <level-6-url> that has a hidden directory in it.
```

Cool, so we have what basically amounts to SSRF on the server. First, i tried just querying the s3 on level6 via the level5 url, like `aws s3 ls s3://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/<level-6-url>/` which didn't work.
After googling it, i learnt about `169.254.169.254`, which is a metadata provider for many cloud providers. Querying http://169.254.169.254/latest/user-data/iam/security-credentials/flaws gives us a new token, which we can use to access the s3 and the next level

## level 6 
initial hint:

```
For this final challenge, you're getting a user access key that has the SecurityAudit policy attached to it. See what else it can do and what else you might find in this AWS account.
Access key ID: <access id>
Secret: <secret>
```

Alright, so after googling, we see that the SecurityAudit group can list shit, pog. Lets get our user via `aws iam get-user`, and attached policies via `aws iam list-attached-user-policies --user-name <username gotten from previous command>`, this shows us another policy, named "list_apigateways".
If we get that one via `aws iam get-policy --policy-arm <arn gotten from previous command>`, and then get the current version via adding --version-id v4 onto the previous command, we are told we can call "apigateway:GET", which invokes lambda functions.

SecurityAudit lets us list lambda functions, so lets do that via `aws --region us-west-2 lambda list-functions`, which shows us a function named "Level6".
Now running `aws --region us-west-2 lambda get-policy --function-name Level6` giives us a string, which has a rest-api-id towards the end (before /*/METHOD) Now we can check the stages of the lambda function: `aws --region us-west-2 apigateway get-stages --rest-api-id "<rest-api-id>"` This gives us the stage name "Prod".
To call the lambda function create a url like `https://<rest-api-id>.execute-api.us-west-2.amazonaws.com/<stage-name>/<final part of url (after /*/METHOD/)>`. Visiting this url points us to the end.
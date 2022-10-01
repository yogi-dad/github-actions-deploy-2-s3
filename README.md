### Deploy React App on s3 buckets using github actions.

Let's start with creating a new react app:

open terminal and run the follow:
```
npx create-react-app name-of-your-app
cd name-of-your-app
git remote add origin url-of-git-repo
git push --set-upstream origin main
```

now we are pretty much done on local system.
let's move to github and perpare our workflow for an automating deployment process to s3 buckets.

go to settings of your repository and click on secrets and add a couple of keys:

![image](https://user-images.githubusercontent.com/2582649/193400089-f318efee-140a-43ec-be49-5af7e254ea64.png)

now let's move back to our application and create workflow file

`mkdir ./.git/workflows && touch ./.git/workflows/main.yml`

and write the following code to yml file:

```
name: CI/CD Pipeline
on:
  push:
    branches: [develop]
jobs:
  continuous-integration:
    runs-on: ubuntu-latest
    steps:
      # Step 1
      - uses: actions/checkout@v3
      - name: Set up Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - name: Install deps
        run: npm install    
      - name: Build Application and Run unit Test
        run: CI=false npm run build
      - name: Configure AWS credentials
        uses: jakejarvis/s3-sync-action@master
        with:
          args: --delete
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: ${{ secrets.AWS_REGION }}
          SOURCE_DIR: ${{ secrets.SOURCE }}
          AWS_S3_BUCKET: ${{ secrets.AWS_BUCKET }}
```
**make sure you push this file.**

**Explanation:**

We have named our workflow as **CI/CD Pipeline** and this work will be executed every time we push code on develop branch.
We have defined a couple of steps:
- **actions/checkout@v3** checkout develop branch.
- **actions/setup-node@v3** setup nodejs and install matrix version.
- install dependencies and build application. **note** I have added CI=false just to avoid warnings shown as errors.
- **jakejarvis/s3-sync-action@master** takes care of pushing code to specified bucket.

alright we done all done with github and workflows, now let's move to **AWS** and configure our **S3** and **CloudFront**.

**S3 Bucket setup:**
- go to https://s3.console.aws.amazon.com/s3/buckets?region=us-west-2
- Choose or create a new bucket
- Go to the permissions tab and ensure bucket is publicly accessible like following:

![image](https://user-images.githubusercontent.com/2582649/193400120-29e87438-3159-4804-af7f-c3d012ab934e.png)

- and edit bucket policies:
```{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowPublicReadAccess",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::your-bucket-name/*"
        },
        {
            "Sid": "AllowCloudFrontServicePrincipal",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudfront.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::your-bucket-name/*",
            "Condition": {
                "StringEquals": {
                    "AWS:SourceArn": "arn:aws:cloudfront::your-account-id:distribution/your-cloud-front-distribution-id"
                }
            }
        }
    ]
}
```
now go back to properties tab and scroll down to static website hosting and enable it with following settings:
![image](https://user-images.githubusercontent.com/2582649/193400137-7fa36c32-2ee9-4517-85a4-fb5e464c8bc3.png)

you can replace index.html with whatever the page you want, but for my app I've kept it index.html.

now let's configure cloudfront for delivering website:

**cloudfront**

- go to https://us-east-1.console.aws.amazon.com/cloudfront/v3/home?region=us-west-2#/distributions
- and create a new distribution with the following settings.
  ![image](https://user-images.githubusercontent.com/2582649/193400234-228a3a97-0cc7-4429-9667-d25857fa4424.png)

once it's done and deployed, you should be able to see your website on cloudfront domain. You can also create error pages and invalidation policies to invalidate cache but that for another day.

**Happy Deployment**

:)




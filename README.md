# lambda-serverless-website
This repo explains how to create a website using AWS Lambda, AWS API Gateway, AWS Route 53 and AWS S3. The website is serverless. Domain registrar is GoDaddy and website is transferred to AWS Route 53.

# Step 1) Transfer domain from GoDaddy to AWS Route 53
Move website from GoDaddy to AWS Route 53. For this, we do the following:  
1. Go to Route 53
2. Click on "Hosted Zones" from left side panel
3. Click on "Create hosted zone"
4. Under Domain name field, enter your website e.g. harqat.com
5. Keep the remaining field at default and click on "create hosted zone". This should create a hosted zone with 2 records: NS and SOA.
6. Switch to GoDaddy and open "My Products" to see all your websites
7. From the list, select the website that you want to move to Route 53 e.g. harqat.com and click on "DNS"
8. Under "Nameservers", click on "Change" to change the nameservers
9. Click on "Enter my own nameservers (advanced)"
10. Copy all 4 nameserver values from Route 53 in NS record to GoDaddy. Hint: Make sure not to copy the fullstop at the end of each nameserver in Route 53.
11. Hit "Save"
12. Click on "Yes, I consent to update Nameservers for the selected domain(s)." and hit "Continue"  
    Note: it usually takes an hour to update the nameservers. However, it may take up to 48 hours for changes to take effect in some cases.
13. Now to make sure that the domain is switched over to Route 53, we will do one check. We first need to flush DNS cache and browser cache. Then,
we visit the website e.g. http://harqat.com and see that nothing shows up. This is good because previously, GoDaddy was showing a parked page by default 
but Amazon doesn't show one. Hence, we make sure that we have now switched over to Route 53, which is exactly what we wanted to achieve.  
Hint: to flush DNS cache, you can open a terminal and on Mac, run the following command:
    > sudo dscacheutil -flushcache;sudo killall -HUP mDNSResponder  

    To clear browser cache, simply click on Settings and depending on your browser, find the option to clear cache.
    Note: as mentioned in previous step, it will take some time till the nameservers are updated. Therefore, if you still see GoDaddy parked website, I 
    would recommend you to wait for the update of nameservers.
    
Resources: https://www.youtube.com/watch?v=zFuluVTsF14

# Step 2) Host a serverless website using AWS Lambda
1. Locate Lambda from AWS services and open it
2. Click on "Create function" button to create a new lambda function
3. Give a name under the "Function name" field e.g. harqat-homepage
4. In "Runtime", select "Python 3.9"
5. Keep the rest at default and click on "Create function"
6. In lambda_function.py, replace the code with the following:
``` python
import json

    
def lambda_handler(event, context):
    body = '''
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Homepage</title>
    </head>
    <body>
        <h1>Home</h1>
        <p>Hi! This is my serverless website.</p>
        <img src="https://website-content-as.s3.us-east-2.amazonaws.com/publicprefix/abdullahshafin_400x400.jpeg" width="500" />
        <button> <a href="/blog">Blog</a></button>
    </body>
    </html>'''
    
    response = {
        'statusCode': 200,
        'headers': {"Content-Type": "text/html",},
        'body': body
    }
    
    return response
```
7. Hit "Deploy" to deplot the code to the Lambda function
8. Next, we will add a trigger to invoke this Lambda function. For that, click on "Add trigger" under "Function overview"
9. In "Select a source", choose "API Gateway"
10. Click on "Create a new API"
11. Choose "HTTP API"
12. Under Security, choose "Open"
13. Click "Add" to create the trigger
14. This should create an HTTP API endpoint. Under "Configuration", choose "Triggers" and copy the API endpoint URL.
15. Open a new tab in your browser, paste the URL and hit Enter
16. This should show you the created webpage hosted serverlessly in AWS Lambda
Congrats! You have now created a serverless website. Next, we will look at how to use your custom domain name instead of random API Gateway URL. 

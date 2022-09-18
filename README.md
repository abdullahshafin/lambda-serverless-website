# lambda-serverless-website
This repo explains how to create a website using AWS Lambda, AWS API Gateway, AWS Route 53 and AWS S3. The website is serverless. Domain registrar is GoDaddy and website is transferred to AWS Route 53.

# Topic 1) Transfer domain from GoDaddy to AWS Route 53
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

# Topic 2) Host a serverless website using AWS Lambda
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

# Topic 3) Create a custom domain name
1. Open "API Gateway" from the list of AWS services
2. Hit "Custom domain names" from the left side panel
3. Click on "Create" button
4. Put the name of your domain name in the field under "Domain name" e.g. harqat.com
5. Jump to "ACM certificate" and click on "Create a new ACM certificate". This opens a new window.  
   Note: Creating a certificate will allow us to have secure connection using HTTPS.
6. Click on "Request" to create a new certificate
7. Click on "Request a public certificate" and hit "Next"
8. In the text field under "Fully qualified domain name", enter your domain name e.g. harqat.com
9. Keep the rest at default and hit "Request"  
   Now you should see a new certificate listed under the "Certificates" in AWS Certificate Manager. In "Status" column, you should see "Pending validation" status.
10. Click on the value in "Certificate ID" colum to see more details about the certificate
11. Under section "Domains", click on "Create records in Route 53"
12. Once prompted, click on "Create records"
13. Head back to your domain name inside "Hosted zones" in Route 53 to verify that a new record of type CNAME has been added.  
    This proves to AWS that you are indeed the owner of this domain.
14. In AWS Certificate Manager under Certificates, you should see the Status change from "Pending validation" to "Issued"  
    Note: it might take a few minutes for status to change.
15. Now that the certificate is issued, we go back to API Gateway (from step 5) and click on the small refresh button next to the drop-down in ACM certificate
16. Clicking on the drop down should now list the newly created certificate
17. Select the certificate 
18. Hit "Create domain name"  
    Now, you should see that a domain name has been successfully created. 
19. Click on "API mappings" next to "Configurations"
20. Click on "Configure API mappings"
21. Click on "Add new mapping"
22. In drop-down under API, select the API that you created in Topic 2) in step 14
23. Select "default" in Stage 
24. Hit "Save" to create this API mapping  
    From API Gateway side, we have done the setup. However, we still to configure DNS settings in order to complete this process. For this purpose, we need to add one last record to hosted zone inside Route 53. This allows us to point the domain to API Gateway domain name. This will allow us to use our custom domain name to get to our Lambda function. For this, follow the steps below:
25. Switch back to "Configurations" from "API mappings" and copy the value under "API Gateway domain name"
26. Go to Route 53 inside your hosted zone and click on your "Domain name" to open the records. Click on "Create record"
27. In "Routing policy", click on "Simple routing" and hit "Next"  
    Note: If you do not see the "Simple routing" option, click on "Switch to wizard" to see the view with the above mentioned option
28. Click on "Define simple record"
29. In "Record name", keep subdomain as empty and keep "Record type" as "A"
30. Under "Value/Route traffic to", select "Alias to API Gateway API"
31. In region, select the region you are working. For example, us-east-2
32. In "Choose endpoint" drop-down, find the option that matches the value you copied in step 25 and select it
33. Hit "Define simple record"
34. Click on "Create records"
35. To verify if things are set up correctly from DNS perspective, copy your domain name e.g. harqat.com
36. Open a new tab, enter URL "dnschecker.org" and hit enter
37. Paste your domain name, keep record type as A and hit "Search". This should show you a lot of green check marks.  
    Note: this means that DNS is indeed setup correctly. 
38. Finally, enter your domain name in a browser and hit enter. This should show you the same webpage as you saw in Section 2) step 15. This means that everything worked correctly and your domain name points to AWS Lambda to host the website serverlessly with custom domain name.  
    Note: It may take a few hours before the final step works. Therefore, be patient and come back after a couple of hours if it doesn't work right away.

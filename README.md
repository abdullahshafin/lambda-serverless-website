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


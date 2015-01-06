#Hosting Nancy with Azure
##Overview
This almost isn't worth writing, since there is little extra work to do in order to host Nancy on Azure.  Azure VMs now run full-trust IIS and so are capable of running just about anything IIS can locally.  However there are some Azure specific things you have to do to actually create an Azure project and deploy it to the cloud.  While these things are not Nancy specific, I will post the walkthrough here for the convenience of people who want to try it.
##Quick Start
First set up Nancy to run under ASP.NET as documented .  Again there is nothing special to deploying ASP.NET sites to Azure as it supports full IIS deployments.  I'll briefly describe the steps but anyone deploying to Azure for the first time should be sure to walk through the [[Windows Azure Training Kit|http://www.microsoft.com/downloads/en/details.aspx?FamilyID=413e88f8-5966-4a83-b309-53b7b77edf78&displaylang=en]] (I apologize if that link breaks, you know how reliable microsoft.com links are...) on how to setup an Azure project and deploy it.  

These steps should help you set up your project to run locally on your DevFabric and once that's working you'll be ready to deploy Nancy to Azure.

1. Create an Windows Azure Cloud Service project in your Solution
2. Add an ASP.NET Web Role Project
3. Configure the Nancy to run under ASP.NET hosting exactly as outlined [[here|Hosting nancy with asp.net]]
4. Profit!!! (Seriously, that's it)
 
That's all there is to setting up, now first write a little hello world handler and map it to `Get["/"]` (feel free to copy the source code from the sample included with Nancy).  Press F5 and the Azure DevFabric should spin up and should open your web browser to your main page.

If everything is working you should see your Hello World result and we are now ready to deploy.  Again refer to the Windows Azure Training Kit on how to deploy to your Windows Azure account (of course you need to have an account first).

I apologize, as this may seem that this isn't a step-by-step walkthrough, but I promise there is no tricks or gotchas to look out for (at least that I've seen).


## Hosting Nancy on "Azure Websites" 

If you're trying to host it on the Azure Websites (Not Azure Cloud Services), the previous steps aren't necessary. Here are the steps which will deploy it to Azure Websites

1. Right Click on your Nancy Project
2. Click "Publish"
3. ??? (VS will do it's magic here)
4. Profit!! (Seriously, that's it)


# Add Lead to Campaign Flow

A simple way to add a lead to campaigns based on a custom campaign id text or lookup field on the lead object.

This project is inspired by [Derek Hays](https://www.linkedin.com/in/derekhays) who had some follow up questions when implementing my other solution to [automate adding leads or contacts to campaigns via report subscriptions](https://douglascayers.wordpress.com/2015/07/06/salesforce-automate-adding-contacts-to-campaign-via-report-subscription/). This is also one of the solutions highlighted on my blog post [8 Ways to Add Leads to Campaigns](https://douglascayers.com/2016/07/31/8-ways-to-add-leads-to-campaigns/).

<a href="https://githubsfdeploy.herokuapp.com">
  <img alt="Deploy to Salesforce"
       src="https://raw.githubusercontent.com/afawcett/githubsfdeploy/master/deploy.png">
</a>

# Use Case

You're integrating leads to Salesforce with a third-party app. And based on actions the lead takes on your website (download whitepaper, subscribe to email newsletter, click certain links, make purchase, etc.) you want them added to different marketing campaigns.

Ideally, the third-party app you're using would know how to do this out of the box and add the lead as campaign members appropriately. But if not, then you have to get a little creative.

# Web-2-Lead

Standard [Web-2-Lead](https://help.salesforce.com/apex/HTViewSolution?id=000006417) supports adding new leads to a camapign using a hidden input field. However, one reason you might be using a third-party app and not Web-2-Lead is to avoid creating duplicate leads for every action they take on your website. The Web-2-Lead form always creates new leads, so it's up to you to implement a proper dedupe and merge process in Salesforce with that approach. Also, you might not want to present a web form for the website visitor to fill out and submit for each event you're tracking.

# Solution

A creative non-code solution is to use Process Builder and add a custom text or lookup field on your Lead object (e.g. "Campaign ID") that is mapped by the third-party marketing app as the person is taking actions on your website. This third-party app is already intelligent enough to create or update an existing lead, and for each event you're tracking, the lead can be updated with a new Campaign ID field value. For example, when clicking a link to download a whitepaper, that might update the lead's campaign id value to `7011a000000xxx1`; and if the lead subscribes to a newsletter then the third-party app might update the lead's campaign id value to `7011a000000xxx2`. 

Process Builder is used to monitor the lead object for create and edit events, specifically for when the custom "Campaign ID" field value changes. Process Builder then runs a Flow that can identify if the lead already exists as a member of the desired campaign, and if not, then creates the CampaignMember record, otherwise does nothing (or do something else you care about like log an activity).

For example, consider these events taken by the same website visitor:

1. Visitor downloads white paper (third-party app creates new lead and sets campaign id to `7011a000000xxx1`)
2. Process Builder detects lead create event and launches Flow to assign lead as campaign member
3. Visitor registers for webinar (third-party app updates same lead and changes campaign id to `7011a000000xxx2`)
4. Process Builder detects lead edit event and launches Flow to assign lead as campaign member
5. Visitor downloads white paper (third-party app updates same lead and changes campaign id to `7011a000000xxx1`)
6. Process Builder detects lead edit event and launches Flow, but Flow realizes lead is already a campaign member and does nothing

The beauty of this solution is, well, it's fairly straight forward to implement, and it requires no code! (I know, I know... as a developer that's speaking blasphemy, but I love the problem solving aspect of using Salesforce platform too!!)

# Bells and Whistles

The campaign member response status is configurable. When Process Builder launches the Flow, a third variable can be set `varCampaignMemberStatus` to a value relevant to the specific campaign. If you leave this variable blank or unset then the Flow will grab the first listed response status on the campaign.

# Screen Shots

![screenshot](/images/process-01-overview.png)

![screenshot](/images/process-02-launch-flow.png)

![screenshot](/images/process-03-flow.png)

![screenshot](/images/process-04-responded-status.png)

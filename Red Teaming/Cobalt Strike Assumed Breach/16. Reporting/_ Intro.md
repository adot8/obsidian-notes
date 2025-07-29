
The final report is arguably the most important aspect of an engagement.  It must accurately communicate to the business whether the objectives were met, what the findings were, what the potential business impacts are, as well as any recommendations.

## Cobalt Strike reports

Cobalt Strike is capable of generating reports based on the activity that it recorded.

- #### Activity report
    
    Provides an overall timeline of post-exploitation activity.
    
- #### Hosts report
    
    Summarises information collected on each host including services, credentials, and sessions.
    
- #### IOC report
    
    Provides technical information akin to a threat intelligence report.  It includes an analysis of your C2's traffic profile, domains, and file hashes.
    
- #### Sessions report
    
    Summarises post-exploitation activity by session.
    
- #### Social engineering report
    
    Reports activity from Cobalt Strike's built-in spear phishing capabilties, i.e. who clicked on a malicious link and what information was gathered by the system profiler.
    
- #### TTP report
    
    Maps activity recorded by Cobalt Strike with MITRE's ATT&CK Matrix.

These can be accessed from the **Reporting** menu.  If the client is connected to multiple team servers, then it will automatically aggregate and order data from them all.  You wouldn't typically hand these over directly to the client, unless in the form of appendices with the main report.

## Red team report

The final report should be constructed using all the logs and notes taken during the engagement, including the Cobalt Strike reports and individual operator logs.  The following is a high-level summary of the types of sections a report may have:

- #### Executive Summary
    
    This section is aimed at business executives and should provide a concise overview of the report's content.  It should highlight the key findings, outcomes and any business risks, without being technical.
    
- #### Goals & Scenario
    
    This section should provide a summary of the red team's testing methodology, the goals and scope of the assessment, and the testing model that was used.  Most of this can be re-hashed from other documents such as the ROE.  
    
- #### Attack Narrative
    
    This section will likely be the bulk of the report's content and should outline the sequence of events that allowed the red team to reach their objective.  Since this is the technical portion, it can include detailed commands, output, and screenshots, etc.
    
- #### Observations & Recommendations
    
    This section should discuss the key steps from the attack narrative and how they impacted the engagement.  An observation will generally describe a deficiency in the organisation's ability to prevent, detect, and/or respond to an activity.  For example, if a red team gained access to credentials through kerberoasting, observations might include weak credentials, insufficient logging to detect the attack, and so on.  Each observation may then have one or more recommendations to mitigate the deficiency.
    
- #### Conclusion
    
    This section simple summarises the key findings, reiterates their significance, and provide a statement of business risk.
    

The following template from redteam.guide is great for getting started.

## Out-briefs

It's also commonplace to have out-briefs with different personnel from the organisation to run through the content of the report.  An executive brief is performed soon after engagement completes and is tailored toward management.  Since the recommendations provided by a red team may require organisational changes to staffing and funding, etc, getting management buy-in, by ensuring they understand the potential business impact, is essential if they are to implement improvements.

One or more technical out-briefs may occur between the red team and other technical teams of the organisation, such as a blue team.  These meetings is where the activity and outcomes of the engagement can be reviewed in detail.  This is the best opportunity for offensive and defensive teams to learn from each other.
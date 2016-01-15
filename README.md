# Salesforce Generic Streaming v2 Demo
This repository contains all the code you need to set up a Generic Streaming v2 client inside of a Visualforce page in your Salesforce Org.  The Generic Streaming v2 adds the capability to replay any of the messages sent to the Generic Channel for the last 24 hours.

This is handled through the use of an Event ID.  Every event sent through the Generic Streaming system will have an associated ID.  With v2, you will have the ability to provide an Event ID (ostensibly the last ID that your client has received) and the system will send every event that has happened in the last 24 hours after that specific ID.

If you do not provide and ID, or the ID you provide is -1, your client will be subscribed to the tip of the queue (you will receive all new messages.  If you provide an ID of -2, we resend ALL events that happened in the last 24 hours.  

Please note that this is a current Beta feature in Salesforce, and in order to sucessfully subscribe to the v2 Channel, you will need to have your org enabled for the new feature.  To do so, please contact jhurst@salesforce.com. 

## Demo Instructions
1. Download this repo as zip file
    * This contains all of the needed code to run the client inside of Salesforce
2. Deploy the code to an enabled Winter '16 org using the MdAPI through Workbench, Force.com IDE, or Ant Migration Tool
3. Assign the included StreamingV2Demo permission set to your user
4. Create the `/u/TestStreaming` StreamingChannel by subscribing to that channel name
    * You can create the channel by using the Workbench and going to Queries > Streaming Push Topics and selecting Generic Subscriptions.  Enter the subscription as `/u/TestStreaming` and click 'Subscribe' (this will create the channel)
    * You can also create the channel by using the "Streaming Channels" tab (you probably have to click the + to pull up the main tab list), and then from the "Streaming Channels" section create a new record.
5. Push an event to the new channel
    * Query the ID of the channel by using the Workbench REST Explorer and doing a GET to `/services/data/v35.0/sobjects/StreamingChannel`.  In thenRecentItems response, you should see the ID for `/u/TestStreaming`
    * Do a POST to `/services/data/v35.0/sobjects/StreamingChannel/<CHANNEL_ID>/push` with a payload of:
        * `{ "pushEvents": [{"payload": "first push"} ]}`  
6. Navigate to `/apex/StreamingV2Demo`.  This Visualforce page will auto-subscribe to `/u/TestStreaming` using Generic Streaming v2 
    * This could fail if you haven't created the streaming channel and haven't pushed at least 1 event to that channel)
7. Generate new events using on-page controls. 
    * You can also push new events through the REST API per the standard [Generic Streaming documentation](https://developer.salesforce.com/docs/atlas.en-us.api_streaming.meta/api_streaming/resources_push.htm)
    * Observe events being received in bottom panel. 
8. Update replay subscription settings by changing the Replay From attribute. This will change where in the event stream you are subscribed to
    * -2 will replay from eariest available event
    * -1 (or no ID) will replay only new events
    * Provide a specific Event ID to replay all events after that specific ID

## Open Issues to Remember
1. This is still a Salesforce Beta Feature
    * It needs to be enabled manually on every org
    * Issues should be communicated directly to te Product Manager, Jay Hurst (jhurst@salesforce.com)
2. You have to push at least one event to a channel in order for the channel to work with v2.  This is resolved with the Spring'16 release
3. You cannot replay using an ID outside of your event stream window (if you provide an ID from 2 weeks ago, it will be invalid)
4. The current event stream window is 24 hours

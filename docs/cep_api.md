The Complex Event Processing API allows you to analyze data from your IoT device and trigger actions.

The following action types are available:

- update: update an entity's attribute
- sms: send a SMS
- email: send an email
- post: make an HTTP POST
- twitter: send a tweet

This API allows you for example to define rules to trigger email notifications based on the data value thresholds or the the lack of updates from a certain device.

These rules are expressed as an EPL sentence. [EPL](http://www.espertech.com/esper/index.php) is a domain language of Esper, the engine for processing events used. This EPL sentence matches an incoming event if satisfies the conditions and generates an "action-event" that will be sent back to FIWARE IoT Stack to execute the associated action. 


# Activate rules

You have to choose which data from your IoT device is relevant to be evaluated by your rules at the FIWARE IoT Stack.

In order to do so, you have to select the Data API entity attributes. On the following example, the temperature attribute will be selected:

```
POST /v1/subscribeContext HTTP/1.1
Host: <cb_host>:<cb_port>
Accept: application/json
Content-Type: application/json
Fiware-Service: {{Fiware-Service}} 
Fiware-ServicePath: {{Fiware-ServicePath}} 
X-Auth-Token: {{user-token}}

{
    "entities": [
        {"type": "device",
        "isPattern": "false",
        "id": "mydevice"
        }
    ],
    "reference": "http://<cep_host>:<cep_port>/notices",
    "duration": "P1Y",
    "notifyConditions": [
           {
                "type": "ONCHANGE", 
                "condValues": ["TimeInstant" ]
} ],
"throttling": "PT1S" }
```

If you are familiar with FIWARE components, on this request you are using the Data API subscription operation to notify new data from your device to the FIWARE CEP component providing the Complex Event Processing API.

# Create rule to send emails

Once you have activated the processing for your data, you can create a rule to trigger actions as follows:

```
POST /rules HTTP/1.1
Host: <cep_host>:<cep_port>
Accept: application/json
Content-Type: application/json
Fiware-Service: {{Fiware-Service}} 
Fiware-ServicePath: {{Fiware-ServicePath}} 
X-Auth-Token: {{user-token}}

{
   "name":"temperature-rule",
   "text":"select *,\"temperature-rule\" as ruleName from pattern [every ev=iotEvent(cast(cast(ev.temperature?,String),float)>40.0)]",
    "action": {
        "type": "email",
        "template": "Alert! temperature is now ${ev.temperature}.",
        "parameters": {
            "to": "someone@yourclient.com",
            "from": "notificactions@yourdomain.com"
            "subject": "Alert! high pression detected"
        }
    }
}
```



# Create rule to send an  HTTP POST

You can also trigger an HTTP POST to an URL specified sending a body built from template:

```
POST /rules HTTP/1.1
Host: <cep_host>:<cep_port>
Accept: application/json
Content-Type: application/json
Fiware-Service: {{Fiware-Service}} 
Fiware-ServicePath: {{Fiware-ServicePath}} 
X-Auth-Token: {{user-token}}

{
   "name":"temperature-rule",
   "text":"select *,\"temperature-rule\" as ruleName from pattern [every ev=iotEvent(cast(cast(ev.temperature?,String),float)>40.0)]",
    "action": {
        "type": "post",
        "template": "Alert! temperature is now ${ev.temperature}.",
        "parameters": {
            "url": "http://yourcustomer.com"
        }
    }
}
```

# Create rule to send a Tweet

You can send a tweet from your account with the text build from the template field. 

Remember you must create your Twitter app and get its OAuth credentials.

```
POST /rules HTTP/1.1
Host: <cep_host>:<cep_port>
Accept: application/json
Content-Type: application/json
Fiware-Service: {{Fiware-Service}} 
Fiware-ServicePath: {{Fiware-ServicePath}} 
X-Auth-Token: {{user-token}}

{
    "name":"temperature-rule",
    "text":"select *,\"temperature-rule\" as ruleName from pattern [every ev=iotEvent(cast(cast(ev.temperature?,String),float)>40.0)]",
    "action": {
        "type": "twitter",
        "template": "Alert! temperature is now ${ev.temperature}.",
        "parameters": {
          "consumer_key": "xvz1evFS4wEEPTGEFPHBog",
          "consumer_secret": "L8qq9PZyRg6ieKGEKhZolGC0vJWLw8iEJ88DRdyOg",
          "access_token_key": "xvz1evFS4wEEPTGEFPHBog",
          "access_token_secret": "L8qq9PZyRg6ieKGEKhZolGC0vJWLw8iEJ88DRdyOg"
        }
    }
}

```

# In more detail ...

You can get more information about the FIWARE component providing this functionalty, reference API documentation and source code at [CEP](cep.md).

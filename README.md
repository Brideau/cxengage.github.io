Getting Started 
---------------

Here is some information on how to setup your instance of CxEngage 

### Event Records

CxEngage uses **Event Records** to help you create patterns and to allow the system to react as needed to various events. Event Records can be configured via the UI or via API. An Event Record contains various attributes to describe it's content, which are divided into the following types:

1. **Key Attribute**: A key attribute is what is used to identify an event record. For example, a key attribute to describe a specific customer or user could be _CustomerID_ or _UserID_. Key attributes are always the first item in an Event Record. You will only have one key attribute.
   
2. **Attributes**: A regular attribute would be something that you would like to use for your pattern. For example, if you want to write a pattern for a contact center, you could use attributes like _CallAction_ or _ToPhoneNumber_ to track when a call is placed and what number should called when a pattern is matched.


These attributes can now be used in a pattern. As an example, here is the pattern where a call gets abandoned from the IVR, and an agent calls the customer back using our integration with twilio: 

```clojure 
;;When
(event (= CallAction "AbandonedCall")

;;Then
(send twilio call {:to-phone-number *To-PhoneNumber*}))
```

If you have any further questions, do not hesitate to reach us by creating a ticket, or e-mailing CxEngage Support at [support@cxengage.com](mailto:support@cxengage.com).

### Using the Echo Endpoint

The Echo endpoint service can be used to test out your patterns. There are no mandatory parameters for the echo service. There is one optional parameter, which is 
* message

Here is a way to use the echo endpoint in your Then
```clojure
(send echo message {:message "send message"})
```

You could also test out your message template with the echo
```clojure
(send echo message {:message "send message"}
(message-template {:message +MT1}))
```

### Using the Twilio endpoint

With CxEngage you can use a Twilio service, with which you can use to send either SMS or Phone calls. To start up the Twilio endpoint service, you need the following items  

* Twilio SID
* Twilio token

With the Twilio service, you can send either sms or phone calls. The keyword to use for each of it is

* sms
* call

The mandatory parameters that the Twilio endpoint needs are 

* to-phone-number - The phone number to call via the endpoint
* from-phone-number - The from phone number that your would like the call to originate from. You get this from Twilio
* message - The message that you want 

So, to be able to use the Twilio endpoint, you need to pass in these parameters. You can do this a few different ways. 

You can use any of the event record parameters that sends in each of these values. So, if you would like to use the following event record parameters toPhoneNumber, fromPhoneNumber and message, your Then in your pattern would look like this - 

```clojure
(send twilio sms {:to-phone-number *toPhoneNumber*
                  :from-phone-number *fromPhoneNumber*
                  :message *message*})
```
You could also set these values. Lets say that your from-phone-number is always the same, you don't need to send that in, you could just write your pattern this way. 

```clojure
(send twilio call {:to-phone-number *toPhoneNumber*
                  :from-phone-number "15068009013"
                  :message *message*})
```
Note that in the pattern above, we are sending a phone call instead of sms. 

You could also use message templating instead of setting it in the pattern

```clojure
(send twilio call {:to-phone-number *toPhoneNumber*
                  :from-phone-number "15068009013"} 
      (message-template {:message +MT1}))
```

### Sendgrid endpoint

With CxEngage, you can use Sendgrid to send emails as a pattern reaction. To start up the Sendgrid Endpoint Service, you need the following items

* Sendgrid Account name
* Sendgrid Password

With the Sendgrid service, you can send emails. The keyword to use for it is

* email

The mandatory parameters that the Sendgrid endpoint needs are

* to - to email address
* from - from email address
* subject - subject of the email
* message - body of your email

So, to be able to use the Sendgrid endpoint, you need to pass in these parameters. You can do this a few different ways.

You can use any of the event record parameters that sends in each of these values. So, if you would like to use the following event record parameters to-email-address, from-email-address, subject and message, your Then in your pattern would look like this -

```clojure
(send sendgrid email {:to *to-email-address*
                      :from *from-email-address*
                      :subject *subject*
                      :message *message*})
```

You could also set your values. Let's says you want the from email address to always be no-reply@cxengage.com. You can write the pattern this way

```clojure
(send sendgrid email {:to *to-email-address*
                      :from "no-reply@cxengage.com"
                      :subject *subject*
                      :message *message*})
```

You could also use message templating for setting it in the pattern

```clojure
(send sendgrid email {:to *to-email-address*
                      :from "no-reply@cxengage.com"
                      :subject *subject*
                      :message +MT1})
```


Setting up your patterns
------------------------

Next, we get to the fun part of creating patterns for things that matter to your enterprise. To enable this we have a simple, yet powerful DSL.

### Intro to the CxEngage DSL

CxEngage's customer Domain-Specific Language (DSL) offers incredible flexibility, providing the ability to set up almost any type of pattern of events to match. At it's most basic, every pattern is of the form: **when** _"events happen"_ **then** _"notify or do something"_.

### "Whens"

The following keywords are available to describe the **"when"** portion of the pattern:

| When keywords|
|:-------------:|
| =             |
| >             |
| <             |
| not           |
| and           |
| or            |
| Within        |
| inSequence    |
| allOf         |
| anyOf         |
| count         |
| failWhen      |
| seconds       |
| minutes       |
| hours         |
| days          |

For example, the following is a pattern where if a customer calls into a contact center twice and abandons the call twice within an hour, then we would call the customer back:

```clojure
;;When
(within 1 hours
        (count 2 (event (= CallAction "abandoned")))
        
;Then
(send echo message {:message "Call abandoned twice, call customer back"})
```
_(Echo is used for convenience in this case to illustrate the example, but is also useful for testing your patterns.)_

Now, what if a customer calls a 3rd time and gets an agent before the agent calls back? Since we don't want the agent to call him or her back, we can account for this scenario by using the FailWhen keyword as follows: 

```clojure
;;When
(allOf (within 1 hours
                 (count 2 (event (= "CallAction" "abandoned")))))
       (failWhen (count 1 (event (= "CallAction" "answered"))))
       
;;Then
(send echo message {:message "Call abandoned twice , call customer back"}))
```



As another example, what if when a customer calls in once, talks to an agent, and then calls back within 30 minutes, we want the customer to be routed to a more experienced second-tier agent? That pattern would look like this:

```clojure
;;When
(within 30 minutes
        (inSequence (event (= "CallAction" "answered")
                    (event (= "CallAction" "inqueue"))))
                    
;Then
(send echo message {:message "Transfer call to senior agent"})
```
More info on [Writing Whens](Writing-Whens.md)

### "Thens"

The following keywords are available to describe the **"then"** portion of the pattern:

| Then keywords|
| :-----------: |
| par        |
| seq        |
| delay      |
| send       |
| if         |
| on-success |
| on-failure |
| timeout    |
| message-template |
| set |

Continuing the examples from above, we can replace the _echos_ with the above _"thens"_. Now, when a caller abandons the call twice, we can use our Twilio integration to trigger an outbound call:

```clojure
;;When
(allOf (within 1 hours
               (count 2 (event (= "CallAction" "abandoned"))))

;;Then
(send twilio call {:to-phone-number *phone-number*}))
```

If we would like to send an SMS to a customer after the two abandons to let them know they will receive a call shortly, we can write the pattern using _seq_ (short for _sequence_) as: 

```clojure
;;When
(allOf (within 1 hours
               (count 2 (event (= "CallAction" "abandoned"))))
                
;;Then
(seq
  (send twilio sms {:to-phone-number *phone-number*
                    :message "We apologize for the long waits, an agent
                              will call you back shortly"})
  (send twilio call {:to-phone-number *phone-number*}))
```

In this example, we use the _seq_ keyword because we want the SMS to go out before the agent calls
back. One thing to note here is that if the SMS fails, the next send does not execute.

### Writing Whens 

Let's start with trying to write a simple expression. A pattern I use a lot is of the form when CustomerSegment is Platinum, do "something". The pattern for that is written this way 

```clojure
(event (= customerSegment "platinum"))
```

* All UETL expressions are of the form - When trigger Then
* So, let's go through all the keywords from the expression above
   1.  When keyword - This is what is used to put in before the expression you want to match
   3.  event - this keyword just means event
   5.  = - this keyword or symbol just means equal
   6.  the values of the events that the pattern is looking for is shown here
   7.  then - this keyword is used before the reaction you would like the system to perform

Now, let's say you want the pattern to match for anyone with customer Segment Platinum with a failed check-in, you will write the expression pretty much the same way but add another event in the list. 

```clojure
(event (and (= customerSegment "platinum")
            (= eventType "flcheck")))
```
As you can see, we use the **and** keyword to do this. Here is a list of those type of operators

| Event keywords| 
| ------------- |
| =             | 
| >             | 
| <             |
| Not           |
| And           |
| Or            | 

Ok, now let's say if we would like the pattern to match if two events happen. For example, a pattern that matches if a person with Customer Segment platinum has 2 failed checkin events. To do this, all we need to do is use the **count** keyword

```clojure
(count 2 (event (and (= customerSegment "platinum")
                     (= eventType "flcheck")))) 
```
The other option for items similar to the count keyword are 

| Event keywords| Usage                   |
| ------------- |-------------------------|
| Within        |  Within Duration Trigger| 
| InSequence    | InSequence [Trigger]    |
| AllOf         | AllOf [Trigger]         |
| AnyOf         | AnyOf [Trigger]         | 
| Count         | Count Int Trigger       |
| FailWhen      | FailWhen Trigger        |



Now, if we want to add another event type to the pattern, for example, you would like the pattern to match when there is 2 failed checkins and 1 cancelled ticket. We use the **allOf** keyword for this. The UETL is written this way - 
```clojure
(allOf (count 2 (event (and (= customerSegment "platinum") 
                            (= eventType "flcheck")))) 
       (count 1 (event (and (= customerSegment "platinum") 
                            (= eventType "cnclticket")))))
```

Now, for this to be more useful, we would only want this to happen for a particular time window, for example, 2 failed checkins and 1 cancelled ticket in the last hour. To do this we use the **within** keyword and one of the following duration keywords, **hour**. 

```clojure
(within 1 hours 
        (allOf (count 2 (event (and (= customerSegment "platinum") 
                                    (= eventType "flcheck")))) 
               (count 1 (event (and (= customerSegment "platinum") 
                                    (= eventType "cnclticket"))))))
```

The options for duration are the following. The unit used is integer

| Duration      | 
| ------------- |
| Seconds       | 
| Minutes       | 
| Hours         |
| Days          | 

Now, if we want the pattern to only match if a cancelled ticket happens after a failed check-in, we can use the **inSequence** keyword. The previous pattern would match if there is a cancelled ticket first and then 2 failed check ins. We only want the pattern to match if a cancel ticket event happens after a failed check in event

```clojure
(allOf (inSequence (event (= eventType "flcheck")) 
                   (event (= eventType "cnclticket"))))
```

All the more commonly used operators are shown being used above. You can use other keywords such as **FailWhen** etc similarly. 

### Writing Thens

As of CxEngage 3.0 the Notification Service consumes a new format for describing reactions,
the Notification DSL (real name pending). This language is necessarily more complex than
UETL, as the requirements for reactions are quite robust.

### Execution Blocks

Reactions are formed from two primitive building blocks: **par** and **seq**.
These two keywords describe the method in which their children should be evaluated.
Both of the execution blocks can be nested within one another.

#### par

```clojure
; Syntax
(par <members> <options>)

; Send three messages to echo in parallel
(par
  (send echo message {:message "Hello World1"})
  (send echo message {:message "Hello World2"})
  (send echo message {:message "Hello World3"}))
```

**par** blocks will evaluate in parallel. One must be conscious of the repercussions of
evaluating their blocks in parallel. Results will be returned in a non-deterministic manner,
thus all members of the parallel block should be independent of one another. Also to note is
that if one member of the block fails, the entire block fails. If a retry is defined, the entire
block will be resent regardless of existing inflight invocations.

#### seq

```clojure
; Syntax
(seq <members> <options>)

; Send one message to echo, delay one minute, then send another message to echo
(seq
  (send echo message {:message "Hello World1"})
  (delay 1 minutes)
  (send echo message {:message "Hello World2"}))
```

**seq** blocks will be evaluated sequentially. As the sequential block is executed in a deterministic
manner, it is safe to create dependencies between elements (in a sequential manner). As with the parallel
block, the block will fail if the current member fails. On failure, if a retry is defined, the sequential
block will restart from its first member.

### Variables

The Notification DSL supports a simple form of variables and assignment. All variables defined in
a reaction are global in scope. This means they can be accessed by any member of the reaction after
definition.

Values can come from one of four sources in the Notification DSL:
* Canonical Event Record
* Endpoint Response
* Reaction Variable
* Constant

Value access of the above types is done as follows:
```clojure
; Access the value custId, in the canonical event record
*custId*

; Access the value call-id, in an endpoint response
$call-id

; Access the value success-call, in the reaction
success-call

; Use a constant value.
; Constants can be strings, boolean, or numbers
"gold"
42
false
```

User defined variables can be created using the **set** command, which can be nested within execution blocks.

```clojure
; Create a variable message-sent, using the call-id value from an endpoint response
(set message-sent $call-id)

; Create a variable cust-seg, using a constant
(set cust-seg "gold")

; Create a variable id, using the custId value in the canonical event record
(set id *custId*)
```

### Actions

Within an execution block the following actions can be taken:

#### send

Example:
```clojure
; Syntax
(send <endpoint> <type> {<params>} <options>)

; Send a message notification to the echo endpoint, with the message parameter
; mapped to the constant value "Hello World"
(send echo message {:message "Hello World"})

; Send an sms message to the twilio endpoint, with to-phone-number from the event record;
; from-phone-number as a constant; and message from a reaction variable.

(send twilio sms {:to-phone-number *phone-number*
                  :from-phone-number "1-506-555-1234"
                  :message sms-message})
```

The **send** command is used to send notifications to specific endpoints. The endpoint, type, and params are
mandatory fields. Additional options will are outlined in the options section. The params must include all
mandatory fields outlined in the definition of the endpoint.

A single send may be used outside of an execution block. This can be useful when using the **if** command.

#### event

Example:
```clojure
; Syntax
(event {<params>} <options>)

; Send a new event with the same key attr (id) and the type "b"
(event {:id *id* :type "b"})
```

The **event** command will create and send a new event into the rest receiver. This functionality is useful
for creating chains of events. Events support message templates and the control flow operators: **on-success** and **on-failure**.

#### delay

Example:
```clojure
; Syntax
(delay <duration> <unit>)

; Create a delay of five minutes
(delay 5 minutes)

; Create a delay of 30 seconds
(delay 30 seconds)
```

The **delay** command will delay further evaluation of a reaction. This delay respects the execution block
in which it is evaluated. In a sequential block, it will block further evaluation until the delay expires.
In a parallel execution block the delay is evaluated in parallel to the other members of the block, thus
causing the delay to act as a *minimum evaluation time* command rather than a strict delay.

Valid time durations are:
* seconds
* minutes
* hours
* days

#### expect

Example:
```clojure
; Syntax
(expect <predicate>
  (on-success <then>)
  (on-failure <then>)
  <options>)

; Wait 5 minutes for an event with type "b" and the current id
(expect (and (= $id *id*)
             (= $type "b"))
  (within 5 minutes)
  (on-success (set got-event true))
  (on-failure (set got-event false)))
```

The **expect** command will pause execution of the reaction until an event enters the notification service which matches the provided predicate. It is recommended to use a **within** when using **expect**, otherwise the reaction may never finish.

#### await

Example:
```clojure
(seq
  (await twilio call-status {:sid 12345}
    (if (= "answered" $call-status)
        (on-success (set msg "The call was answered"))
        (on-failure (set msg "No one was there")))
    (within 10 minutes))
  (send twilio sms {:message msg ...}))
```

The *await* command is used to perform asynchronous communication with an endpoint. In truth,
*await* is syntactic sugar to perform a poll against a command in an endpoint. The body of the
*await* must consist of an *if* statement, whose predicate represents what you are waiting for.
The *on-success* and *on-failure* within the *if* will be executed based on the final outcome of
the await. A *within* option must also be present to denote the total time the poll should last.
The poll cycle is hard coded to wait one second between polls.

#### if

Example:
```clojure
; Syntax
(if <predicate> (on-success <then>) (on-failure <else>))
; If the cust-seg event record value is "gold" then send message-one to echo otherwise send
; message-two

(if (= *cust-seg* "gold")
  (on-success (send echo message {:message message-one}))
  (on-failure (send echo message {:message message-two})))
```

The **if** command is the most complex in the notification DSL.
Predicates can be composed of the following commands:
```clojure
; AND
(and <predicate> <predicate>)

; OR
(or <predicate> <predicate>)

; NOT
(not <predicate>)

; =
(= <predicate> <predicate>)

; >
(> <predicate> <predicate>)

; <
(< <predicate> <predicate>)

; or a value of any type
*id*
42
"Hello World"
```

The *then* and *else* elements can be either an execution block or a single command.

### Options

The following options can be applied to **seq**, **par**, **send**, and **await** commands

#### retries

Example:
```clojure
; Syntax
(retries <count>)

; Send a message to echo with 2 retries
(send echo message {:message "Hello World"}
  (retries 2))
```

If a command has a defined retry option, when the command fails, it will be retried. A command will
only be treated as *failed* when all of its retries have been exhausted.

#### within

Example:
```clojure
; Syntax
(within <duration> <unit>)

; Send three messages to echo in parallel with a five second timeout
(par
  (send echo message {:message "Hello World"})
  (send echo message {:message "Hello World"})
  (send echo message {:message "Hello World"})
  (within 5 seconds))
```

When a command is evaluated which has a defined timeout, if it does not complete within the defined duration,
the event will be failed. Retries will be respected by timeouts, if the timeout occurs, and retries remain,
the command will be retried.

#### message-template

Example:
```clojure
; Syntax
(message-template {<params>})

;Use a message template when sending an sms to twilio
(send twilio sms {:to-phone-number *phone-number*
                  :from-phone-number "1-506-555-1234"}
  (message-template {:message "Hello {{first-name}}"}))

(send twilio sms {:to-phone-number *phone-number*
                  :from-phone-number "1-506-555-1234"}
  (message-template {:message +MT1
                     :twil-ml +MT3}))
```

Parameters defined using the **message-template** option will be sent to the endpoint as the rendered
version of the provided template. The template will have access to the canonical event record when
being rendered. The value for the template can be any of the value types defined in the Notification DSL.
Parameters which are to be templated, do not need to be defined in the main send parameter definition.

#### on-success and on-failure

Example:
```clojure
; Syntax
(on-success <then>)
(on-failure <then>)

; Send an sms, and send a different message to echo depending on the success of the sms
(send twilio sms {:to-phone-number *phone-number*
                  :from-phone-number "1-506-555-1234"}
  (on-success (send echo message {:message "It sent"}))
  (on-failure (send echo message {:message "No it didn't"})))
```

The **on-success** and **on-failure** options can be used to provide additional actions to be
taken based upon the success and/or failure of a command. One does not need to define both success
and failure option if not required.

In the presence of an **on-failure** option the failure of the command is swallowed and the reaction
will continue processing after evaluating the *then* commands. The **on-failure** option will only be
evaluated once a command has exhausted all of its defined retries.

The **on-success** block is the only scope in which commands have access to the endpoint response
values (those prefaced with a **$**). If the persistence of a response value is required for the
duration of the reaction, a **set** command can be used to persist the value in a reaction variable.

### Big Example

```clojure
(par (send twilio sms {:to-phone-number *to-phone-number*}
           (message-template {:message *template3*}))
     (seq (send echo message {:message *eventType*}
                (on-success
                 (set twilio-message *template1*))
                (on-failure
                 (set twilio-message *template2*)))
          (send twilio sms {:to-phone-number *to-phone-number*}
                (message-template {:message twilio-message})
                (on-success
                 (set sms-id $call-id)))
          (send vht set-top-pop {:message "Hello"
                                 :target "127.0.0.1"}
                (within 5 minutes)
                (on-success
                 (seq
                  (send echo message {:message $pop-id})
                  (set pressed-ok $pop-id))))
          (if pressed-ok
            (send echo message {:message "Pressed OK"}))
          (retries 3)
          (within 30 minutes)
          (on-failure
           (seq
            (send echo message {:message *custId*})))))
```

Integrate with CxEngage
=======================

CxEngage has APIs you can use to interact or integrate with

---------

#### Sending Events to CxEngage

Once an event record and pattern have been configured, you can start sending events into CxEngage.

## Getting the Event Record

```
GET events/new
```

### Description

This will return to you the event record which has been configured for your instance.

These are the fields which may be included in your request when you send an event.

### Example

**Request**

```
GET http://api.cxengage.com:8086/events/new
```

**Return**

```json
{
    "custId": "",
    "eventType": "",
    "customerSegment": "",
    "program": "",
    "device": "",
    "kbtopic": "",
    "phoneNumber": "",
    "emailAddress": "",
    "firstName": "",
    "lastName": "",
    "deviceId": "",
    "contact": ""
}
```

## Sending an Event

```
POST events/create
```

### Description

This resource allows you to send in events to CxEngage. Events must be a JSON object.

**NOTE**: Make sure that your `Content-Type` is set to `application/json`.
**NOTE**: You must include the key attribute which was defined when you setup your event record.


### Example

**Request**

```
POST http://api.cxengage.com:8086/events/create
Content-Type: application/json
```

```json
{
"custId" : "101",
"customerSegment" : "bronze",
"contact" : "15062592237"
}
```

**Return**

```json
{
    "status": "partial",
    "contributed": false,
    "created": "2013-06-12T20:07:41.569Z",
    "id": "EV1-99",
    "matched": {
        "custId": "101",
        "customerSegment": "brsonze",
        "contact": "15062592237"
    },
    "missing": [
        "eventType",
        "program",
        "device",
        "kbtopic",
        "phoneNumber",
        "emailAddress",
        "firstName",
        "lastName",
        "deviceId"
    ]
}
```

## Accounts

Resource | Description
--- | ---
[<code>GET</code> account](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/account/GET.md) | Returns the authenticated user's account information
[<code>GET</code> account/instances](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/account/GET_instances.md) | Returns the active instaces the authenticated user has permission to use
[<code>POST</code> account/change_password](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/account/POST_change_password.md) | Updates the authenticated user’s password

## Instance

Authenticated methods that allow you to get more information about an instance which you have permission to use.

Resource | Description
--- | ---
[<code>GET</code> instance/:iid](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/instance/GET_instance_iid.md) | Returns `iid` instance information
[<code>GET</code> instance/:iid/enabled](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/instance/GET_instance_iid_enabled.md) | Returns whether or not instance `iid` is enabled
[<code>GET</code> instance/:iid/expiration](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/instance/GET_instance_iid_expiration.md) | Returns the expiration of instance `iid`
[<code>POST</code> instance/:iid/validate](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/instance/POST_instance_iid_validate.md) | Validate a DSL expression against the event record on instance `iid`

## Event Record

Authenticated methods that allow you to manage the event record of one your instances.

Resource | Description
--- | ---
[<code>GET</code> instance/:iid/event_record](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/event_record/GET_instance_iid_event_record.md) | Returns the event record on instance `iid`
[<code>POST</code> instance/:iid/event_record](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/event_record/POST_instance_iid_event_record.md) | Updates the event record on instance `iid`

## Lists

Authenticated methods that allow you to manage the lists of one of your instances.

Resource | Description
--- | ---
[<code>GET</code> instance/:iid/lists](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/lists/GET_instance_iid_lists.md) | Returns all lists on instance `iid`
[<code>POST</code> instance/:iid/lists](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/lists/POST_instance_iid_lists.md) | Creates a new list on instance `iid`
[<code>GET</code> instance/:iid/lists/:id](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/lists/GET_instance_iid_lists_id.md) | Returns the list matching `id` on instance `iid`
[<code>POST</code> instance/:iid/lists/:id](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/lists/POST_instance_iid_lists_id.md) | Updates the list matching `id` on instance `iid`
[<code>DELETE</code> instance/:iid/lists/:id](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/lists/DELETE_instance_iid_lists_id.md) | Deletes the list matching `id` on instance `iid`

## Patterns

Authenticated methods that allow you to manage the patterns of one of your instances.

Resource | Description
--- | ---
[<code>GET</code> instance/:iid/patterns](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/patterns/GET_instance_iid_patterns.md) | Returns all patterns on instance `iid`
[<code>POST</code> instance/:iid/patterns](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/patterns/POST_instance_iid_patterns.md) | Creates a new pattern on instance `iid`
[<code>GET</code> instance/:iid/patterns/:id](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/patterns/GET_instance_iid_patterns_id.md) | Returns the pattern matching `id` on instance `iid`
[<code>POST</code> instance/:iid/patterns/:id](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/patterns/POST_instance_iid_patterns_id.md) | Updates the pattern matching `id` on instance `iid`
[<code>DELETE</code> instance/:iid/patterns/:id](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/patterns/DELETE_instance_iid_patterns_id.md) | Deletes the pattern matching `id` on instance `iid`

## Nodes

Authenticated methods that allow you to manage the nodes of one of your instances.

Resource | Description
--- | ---
[<code>GET</code> instance/:iid/nodes](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/nodes/GET_instance_iid_nodes.md) | Returns all nodes on instance `iid`
[<code>POST</code> instance/:iid/nodes](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/nodes/POST_instance_iid_nodes.md) | Creates a new node on instance `iid`
[<code>GET</code> instance/:iid/nodes/:id](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/nodes/GET_instance_iid_nodes_id.md) | Returns the node matching `id` on instance `iid`
[<code>POST</code> instance/:iid/nodes/:id](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/nodes/POST_instance_iid_nodes_id.md) | Updates the node matching `id` on instance `iid`
[<code>DELETE</code> instance/:iid/nodes/:id](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/nodes/DELETE_instance_iid_nodes_id.md) | Deletes the node matching `id` on instance `iid`

## Endpoint Definitions

Authenticated methods that allow you to get the endpoint definitions that are available to your instance.

Resource | Description
---- | ---
[<code>GET</code> instance/:iid/endpoints](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/endpoints/GET_instance_iid_endpoints.md) | Returns all endpoint definitions on instance `iid`
[<code>GET</code> instance/:iid/endpoints/:id](https://github.com/cxengage/cxengage.github.io/blob/master/apidocs/endpoints/GET_instance_iid_endpoints_id.md) | Returns the endpoint matching `id` on instance `iid`



